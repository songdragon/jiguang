# 目录结构

**状态**　设计，待实现

当前仓库维护首页、许可证和设计文档。以下 Cargo 文件和开发目录随对应功能首次实现时创建。

## 首阶段目标结构

```text
jiguang/
├── .gitignore
├── LICENSE
├── README.md
├── Cargo.toml
├── Cargo.lock
├── docs/
│   ├── vision.md
│   ├── repository-layout.md
│   ├── technical-stack.md
│   └── architecture.md
├── crates/
│   ├── jiguang-core/
│   │   ├── Cargo.toml
│   │   └── src/
│   │       ├── lib.rs
│   │       ├── task.rs
│   │       ├── run.rs
│   │       ├── event.rs
│   │       ├── ports.rs
│   │       ├── model.rs
│   │       ├── context.rs
│   │       └── completion.rs
│   ├── jiguang-platform/
│   │   ├── Cargo.toml
│   │   ├── migrations/
│   │   ├── src/
│   │   │   ├── lib.rs
│   │   │   ├── providers/
│   │   │   ├── storage/
│   │   │   ├── workspace/
│   │   │   └── tools/
│   │   └── tests/
│   │       └── fixtures/
│   └── jiguang-cli/
│       ├── Cargo.toml
│       ├── src/
│       │   ├── main.rs
│       │   ├── commands.rs
│       │   └── render.rs
│       └── tests/
└── evals/
    ├── README.md
    └── tasks/
```

## 依赖方向

```text
jiguang-cli ──► jiguang-platform ──► jiguang-core
      └───────────────────────────► jiguang-core
```

| crate | 职责 | 边界 |
| --- | --- | --- |
| `jiguang-core` | 领域类型、任务状态机、事件、核心接口、循环推进、上下文和完成判定 | 通过接口获取工具、模型、时间、工作区和持久化能力 |
| `jiguang-platform` | 原生模型协议、Fake Provider、SQLite、产物存储、Git 工作区、文件及命令工具 | 把外部失败映射为核心可识别的结果；向核心报告副作用和证据 |
| `jiguang-cli` | `jiguang` 二进制、参数与配置、依赖装配、进度渲染 | 命令委托核心推进任务，通过结构化结果决定输出和退出状态 |

数据库迁移跟随平台存储实现，集成夹具归其测试使用。CLI 端到端测试使用临时 Git 仓库和脚本化模型。评估任务独立于生产代码，记录输入、初始版本、运行预算和评估方法。保留集和外部验证器留在 agent 无法读取的评估环境。任务事件、产物、鉴权配置归本地应用数据目录，工作区中只有用户授权的变更。

工作台开始实现后可新增 `crates/jiguang-server/` 和 `apps/web/`。服务端复用核心及平台，Web 前端经 API 消费任务和事件。

[技术架构](architecture.md)
