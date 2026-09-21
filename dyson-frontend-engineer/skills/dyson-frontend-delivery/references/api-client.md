# api-client 生成与联调手册(dyson 工程)

> 规则提炼自标准 dyson 工程的 `packages/api-client` 实现(orvalHelper/custom-fetch)与实战工程先例;**具体命令与文件名以工程内现状为准**——动手前先探 `packages/api-client` 是否在、有哪些现成 `*.config.ts` 可参照。本文给的是**形态与不变量**,工程内证据优先于本文。

## 生成五步(swagger 地址 → config → 生成命令 → exports → vite 代理)

生成工具=**Orval**;配置工厂=`packages/api-client/orvalHelper.ts`(工程自带,勿改);产物=`generated/<workspace>/`(只读,严禁手改);手写工具层=`src/tool/`(可改)。

以新服务 `cfgreg` 为例:

1. **拿 swagger 地址**:后端完成报告/启动说明里的 api-docs JSON 地址(springdoc `/v3/api-docs`、springfox `/v2/api-docs`)。**生成时会 fetch 它——后端不在线=生成直接失败**;
2. **建 config**:落 `packages/api-client/cfgreg.config.ts`,形制:

   ```ts
   import orvalHelper from "./orvalHelper";
   export default orvalHelper({
       swaggerUrl: 'http://<后端host:port>/v3/api-docs',
       workspace: 'cfgreg',
       apiPrefix: '/CFGREG'
   })
   ```

3. **加生成命令并触发**:`packages/api-client/package.json` scripts 加 `"cfgreg": "orval --config ./cfgreg.config.ts -- --disabled_mixup"`,在该包目录跑 `pnpm cfgreg`。产出 `generated/cfgreg/{services,models}` + MSW mock;postorval 钩子自动重建 `src/mock-index.ts`。`-- --disabled_mixup`=关闭加密层生成(本地简单后端一律加;不加会按 swagger 的 `x-mixup-sdk*` 扩展生成加密注册);
4. **加 exports**:同 package.json 的 exports 加 `"./cfgreg": "./generated/cfgreg/index.ts"`——domain 侧才能 `import { xxx } from '@lib/api-client/cfgreg'`(导出别名与 workspace 目录名对应即可);
5. **vite dev 代理 apiPrefix(最容易漏的一步)**:apiPrefix 是相对路径,不代理=请求打去 dev server 自身=404。在**承载页面的 dev server** 配置里(shell 的 `vite.config.ts`,domain 自启动调试则 domain 的)加:

   ```ts
   server: { proxy: { '/CFGREG': { target: 'http://<后端host:port>', changeOrigin: true } } }
   ```

## 信封(消费视角——读懂生成的 client 在做什么)

orvalHelper 默认 mutator=`customWrappedFetch`(在 `src/tool/custom-fetch.ts`):**每个生成函数返回时都按「信封」解包**。后端契约(详见 C 侧 `.plan/api.md` 信封契约节):

- 每个响应=包装 `{success, message, data}`,`data` 里才是真载荷;
- 业务失败:HTTP 200+`success:false`+`message`,或 HTTP 4xx/5xx+body 含 `message`(后者直接抛 `ApiError`);
- 分页端点(swagger 标了 `x-sdk-page-wrapper`):`data={total, pageNum, pageSize, list:[…]}`,函数收 `pageNum/pageSize` 参数;
- 后端响应不满足信封 → `res.data`=undefined/错形 → **调不通,且怎么改前端都没用**——排障先 curl 后端看响应形状,别急着改前端代码。

## domain 消费姿势(ACL 防腐层,别裸用生成函数)

- 生成函数只在 domain 的 `src/client/api.ts` 里用,三段式命名(`_queryXxx`/`_createXxx` → `useXxxList` hooks → 复合 hook);
- 固定姿势:`const r = await 生成函数(参数); if (!r.success) throw new Error(r.message || '<业务名>失败'); return 清洗(r.data);`——DTO 清洗成领域模型,脏数据不进 UI(工程规范真相源:`.claude/rules/data-layer-react-query.md`,React Query v5 对象参数写法);
- `generated/` 只读;要改接口=让后端改 swagger 再重新生成,不手改产物;
- 静态期(后端未就绪)可用 `@lib/api-client/mock`(MSW handler)自测。

## 时序与自测

生成发生在**切后端联调任务**(后端已起、swagger 可达)时;静态页任务(M1 形态)不生成。联调自测至少走一条主链(建一条数据 → 列表出现该条),联调记录入交付报告。

## 排障速查

| 症状 | 先查 |
|---|---|
| 请求 404 | vite proxy 漏配第 5 步/apiPrefix 拼错 |
| `res.data` undefined | 后端没包信封(让 C 修,不是前端问题) |
| 生成命令失败 | 后端不在线/swagger 地址不可达/地址返回 HTML 非 JSON |
| 生成函数名不可读 | 后端 swagger 的 operationId/tag 乱命名(让 C 改注解重新生成) |
| domain 引用不到 | exports 第 4 步漏加/别名与 workspace 不一致 |
