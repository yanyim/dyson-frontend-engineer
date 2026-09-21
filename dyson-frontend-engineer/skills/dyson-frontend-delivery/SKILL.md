---
name: dyson-frontend-delivery
description: dyson 前端开发工程师的干活规范。接到前端任务信(做静态页/改页面/切真实后端联调)、被请求「方案会审」(本地探索审查方案)、要写交付报告、要起草 api 接口初版、或被请求审查接口文档时使用。收到流水线协作邮件先按 tf-flow 技能行事(唤醒四步循环)。
---

# dyson 前端交付(dyson-frontend-delivery)

## 开工第一步:识别开发模式(每次任务都做)

读工作区根 `package.json`:`dyson-bootstrap` 字段在=dyson 工程(存量兜底:pnpm-workspace 三 glob+dyson 签名包命中≥2);不在=通用前端模式,**在交付报告与回信中言明**。dyson 工程按本技能 dyson 各节干活。

## 方案会审(接到「方案会审」信)

在**自己工作区本地探索**,从实现视角审查方案,回信指出疑惑点/风险/遗漏——重点:工程能提供什么(组件/表单/主题/工程技能)、缺什么前置(如后端 swagger 地址未定)、方案哪步不可行。**走 api-client 联通方式时必核两项**:① 工程 `packages/api-client` 在不在、生成五步走不走得通(references/api-client.md);② 方案里后端是否承诺**信封契约**(响应包装/分页扩展/swagger 地址与包装 schema)——没承诺=会审疑惑点,别默认可行。**探索出的工程事实(可复用面/前置依赖)必须回信**,不许默认方案可行。

## dyson 工程:复用与技能

- **复用优先**:shadcn/ui、表单组件、主题等工程已有组件一律复用;先探特性包结构(client 数据层/components 视图层/routes 容器层/tool 工具层)再动手;
- **新建子项目一律 `/dyson-create`(硬规,#140)**:建 shell/domain/block/lib 任何一类都必须走工程技能 `/dyson-create`,**严禁手搓目录**——手搓必错点:依赖方向(`@shell→@domain→@block→@lib` 禁反向)/tsconfig 相对深度(项目根 2 层与 src 内 3 层差一级)/index.css 主题注入/端口分配/domain 双入口+路由嫁接工厂。交付报告须言明子项目经 `/dyson-create` 创建;手搓=违规,A 退回;
- **工程技能优先**:工程本地技能(`/dyson-doctor` 规范检查等)能干的活交给技能,不手搓;
- 遵工程 `AGENTS.md` 规范;改动后 `tsc --noEmit` 与构建跑通再交付。

## dyson 工程:api-client 生成五步(调接口的唯一正道,不裸写 fetch;#140 具体化)

生成工具=Orval,dyson 工程自带 `packages/api-client`(orvalHelper 配置工厂+custom-fetch 信封解包层+`generated/` 只读产物)。**详细手册=本技能 `references/api-client.md`,动手前先读**。五步(以新服务 `cfgreg` 为例):

1. **拿 swagger 地址**:后端 api-docs JSON 地址(联调/切后端任务硬前置;**生成时后端必须在线**;缺了在回执与会审中指出);
2. **建 `cfgreg.config.ts`**:落 `packages/api-client/`,形制 `orvalHelper({swaggerUrl, workspace:'cfgreg', apiPrefix:'/CFGREG'})`;
3. **加生成命令并触发**:该包 package.json scripts 加 `"cfgreg": "orval --config ./cfgreg.config.ts -- --disabled_mixup"` → 包目录 `pnpm cfgreg` → 产出 `generated/cfgreg/`;
4. **加 exports**:同 package.json exports 加 `"./cfgreg": "./generated/cfgreg/index.ts"`——漏这步 domain 引用不到(`import from '@lib/api-client/cfgreg'`);
5. **vite dev 代理 apiPrefix**(最易漏的一步):承载页面的 `vite.config.ts` 加 `server.proxy['/CFGREG']→后端`——漏=请求打去 dev server 自身=404,联调必卡。

**信封常识**(详见 references):生成的 client 默认强制解包 `{success, message, data}` 信封;`res.data` undefined=后端没包信封(让 C 修,不是前端问题);消费只在 domain `src/client/api.ts` 适配器里(`if(!res.success)throw`+DTO 清洗),不裸用生成函数、不手改 `generated/`。

## 接到前端任务(派发形态)

1. 读信内指针的 PLAN 节(绝对路径,跨区只读)——**页面清单+逐条验收=完成定义**;
2. `.plan/wireframe.html`(A 区)=排版/配色**意图参考,非交互契约**:实现自由、不复刻,验收以 PLAN 为准;
3. 产出落自己工作区(如 `web/`);dyson 工程=在工程结构内开发(脚手架优先 /dyson-create);
4. **`.plan/api.md` 接口初版**(任务含此时):从页面意愿倒推,端点/请求/响应/错误形状写全;硬约束入文:**CORS `Access-Control-Allow-Origin: *`**;若本项目走 api-client,注明「后端须提供 swagger 地址」**并写全「信封契约」节**(包装格式 `{success,message,data}`/分页形状+`x-sdk-page-wrapper` 扩展/裸响应 `x-sdk-no-wrapper`/swagger 须声明包装 schema——细则见本技能 references/api-client.md)——C 的实现与后续联调全靠这节对齐,漏写=联调返工。

## 交付报告(三要素,缺一必被退回)

1. **验收对照**:逐条「条目 → 结果 → 证据指针」;
2. **自测记录**:实跑的验证(命令+关键输出)——静态期=本地校验(必填拦截/提交态/空态);dyson 工程加 `tsc --noEmit`+构建结果;
3. **自主决策清单**:自行拍板的事项与理由(复用取舍、组件方案、模式识别结论等)。

## 审查接口文档(被请求审查时)

对照页面意愿逐条核:字段够不够用、错误形状能否友好呈现、**信封契约节在不在且与 references 一致**(走 api-client 时:包装格式/分页扩展/swagger 包装 schema);回复**二选一**:「无异议」或「有异议+逐条列点」。

## 切后端联调(切换形态)

- 前置:后端完成报告的**启动说明+swagger 地址**;dyson 工程先走 api-client 五步(第 5 步 vite 代理别漏),再在页面接入;
- 自测走通至少一条主链(如:建一条数据 → 列表出现该条);联调记录(操作+观察结果)入交付报告;
- 校验错误呈现(非法数据被 4xx 拒绝时用户看得到)一并验。
