# dyson 前端开发工程师(dyson-frontend-engineer)

TF Agent Desktop 多会话协作流水线的 **B 席位/前端执行者**(dyson 工程感知):识别开发模式、复用 dyson/shadcn 现有资源、swagger 生成 api-client、交付自证、联调切后端。

## 市场条目(catalog)

| 字段 | 值 |
|---|---|
| id | `dyson-frontend-engineer` |
| type | `preset` |
| name | dyson 前端开发工程师 |
| location | `https://github.com/yanyim/dyson-frontend-engineer.git` |
| path | `dyson-frontend-engineer` |
| version | 1.0.0 |

## 仓库结构

```
dyson-frontend-engineer/        # 仓库根
└── dyson-frontend-engineer/    # 上架物(preset 本体)
    ├── agent.cordis.yml        # dsh 框架消费的 agent 定义
    ├── preset.yml              # dsh preset 元数据(name/description/order)
    ├── desktop.yml             # 旁车文件(TF Agent Desktop 产品层:开场白/建议项)
    └── skills/
        └── dyson-frontend-delivery/   # 私有技能:交付流程+api-client 五步(含信封契约)
```

> **为何是 `<id>/` 子目录而非仓库根**:市场安装纪律要求 preset 仓库内子目录名 === 清单 id
> (`assertShape` 的 basename 对齐校验)。与 `dyson-framework-assistant` 上架形态一致。

## 安装 / 卸载

TF Agent Desktop → 市场 → 找到「dyson 前端开发工程师」→ 安装(内部 `git clone --depth 1` 本仓库,
物化 `dyson-frontend-engineer/` 子目录到 `$DSH_HOME/.agent-presets/dyson-frontend-engineer/`);卸载即删该目录。

## 维护

本仓库是**唯一权威源**(自 tf-ai-destop `presets/` 迁出,#141)。修改 persona/技能后:
commit → push → 在应用市场卸载重装即可生效。

注意:`references/plan-template.md` 等快照类内容如 tf-ai-destop 侧源文档更新,需回灌本仓。

## 出处

- 迁自 tf-ai-destop `presets/dyson-frontend-engineer/`(M9 T2;#135 由 dyson-framework-assistant 改造;#140 补 api-client 五步+信封契约;#141 迁出上架)
- 角色定义:tf-ai-destop `docs/PLAN3.md` §三角色 persona
