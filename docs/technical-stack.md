# 技术选型

**状态**　设计，待实现

## 主语言

采用 **Rust stable、Edition 2024**。吉光的主要工作是等待模型和工具响应，代码生成速度主要取决于模型、请求和网络。选 Rust 是为了学习并掌握本地 harness 的执行机制，同时用类型表达运行状态、任务事件和不同协议的失败语义。Rust 也适合将 CLI、进程控制和本地存储随一个应用分发。具体最低工具链版本在初始化依赖后写入 `rust-version`。 [Rust Edition Guide](https://doc.rust-lang.org/edition-guide/rust-2024/index.html)

Go 的 goroutine、`context.Context` 和标准库适合快速交付同类 CLI 或网络服务。当前优先级是明确建模状态和控制本地进程，选择 Rust，并用较少的 crate 与串行执行控制实现复杂度。

## 首阶段技术栈

| 问题 | 选型 | 使用方式 |
| --- | --- | --- |
| Cargo workspace | Cargo、Clippy、rustfmt | 三个 crate，提交 `Cargo.lock`，依赖从实际用到时引入 |
| 异步和取消 | Tokio、`tokio-util::sync::CancellationToken` | 网络、进程、定时取消、有限容量队列；先串行驱动工具 |
| 模型协议 | `reqwest`、rustls、Serde | API 请求和流式解码在适配器中实现；核心只消费归一化事件 |
| 工具参数 | Serde、`schemars`、`jsonschema` | 从工具输入类型产生 schema，适配器按协议投影；运行时按本地契约校验 |
| 事件和投影 | SQLite、`rusqlite` | 移交给专用数据库线程；启用 WAL，限制事务范围和入队容量 |
| 文件和进程 | `std::fs`、`tokio::process`、Git、ripgrep | 查询仓库、限制输出、控制子进程资源和退出；文件变更由工具实现校验 |
| CLI 与诊断 | `clap`、`tracing`、`tracing-subscriber` | 子命令和结构化诊断；用户看到的实时输出单独消费事件 |
| 测试 | Cargo test、`tempfile`、Fake Provider | 单测覆盖状态和协议，集成测试隔离 Git 仓库，故障注入可重复；真实模型评估显式运行 |

[Tokio 文档](https://docs.rs/tokio/latest/tokio/)、[reqwest 文档](https://docs.rs/reqwest/latest/reqwest/)、[rusqlite 文档](https://docs.rs/rusqlite/latest/rusqlite/)、[Schemars 文档](https://docs.rs/schemars/latest/schemars/)

## 关键取舍

- **直接实现协议适配。** 首个原生适配器走 OpenAI Responses，Fake Provider 用两组不同能力配置验证模型边界。新增 Anthropic Messages 时复用上下文、工具和事件契约。模型专属字段、流式事件和请求形状由适配器处理。 [OpenAI Responses API](https://developers.openai.com/api/reference/resources/responses/methods/create)、[Anthropic Messages API](https://platform.claude.com/docs/en/api/messages)
- **用轻量本地存储保存证据。** SQLite 存结构化事件、幂等键和任务投影，独立文件保存体积较大的日志、补丁和报告。数据库线程按有界队列处理请求，CLI 和模型交互继续在异步运行时上执行。
- **直接实现可检查的 agent loop。** Copilot SDK 的允许应用控制循环和会话持久化值得借鉴。Pi 的小工具集、OpenCode 的会话设计和 DeepSeek Harness 的模块划分作为参考。吉光的完成规则、错误分级和恢复行为由自身实现和测试。 [Copilot SDK agent loop](https://github.com/microsoft/copilot-sdk/blob/main/docs/features/agent-loop.md)、[Copilot SDK 会话持久化](https://github.com/microsoft/copilot-sdk/blob/main/docs/features/session-persistence.md)、[Pi 模型层](https://github.com/earendil-works/pi/blob/main/packages/ai/README.md)、[OpenCode 会话层](https://github.com/anomalyco/opencode/tree/dev/packages/core/src/session)、[DeepSeek Harness 架构](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/architecture.md)
- **按发行边界管理许可证。** 仓库采用 Apache-2.0。第三方依赖的许可证与公告义务按实际分发产物核查。 [项目许可证](../LICENSE)

## 后续选型

若需要常驻后台与工作台，先在已有核心接口上接入 **axum + HTTP/SSE**，再用 **React + Vite** 实现本地界面。SSE 用于状态重放和实时展示，依赖持久事件游标。第二个真实模型协议优先接入 Anthropic Messages。等任务集能量化收益后，再评估摘要、索引检索、模型路由和独立评审。 [axum SSE 文档](https://docs.rs/axum/latest/axum/response/sse/)
