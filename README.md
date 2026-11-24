# 吉光 · Jiguang

吉光是一个独立的本地 coding agent，计划使用 Rust 实现。它直接连接模型 API，在 Git 工作区内读取、修改和验证代码，并保存任务进度以及完成证据。

项目目前处于设计阶段。首阶段交付 CLI 和核心 harness，优先解决任务做到一半就退出、恢复时丢失进度、结果缺少验证的问题。

## 设计文档

| 文档 | 内容 |
| --- | --- |
| [项目愿景](docs/vision.md) | 产品定位、首阶段范围、验收指标 |
| [目录结构](docs/repository-layout.md) | Cargo workspace、crate 职责、模块和测试位置 |
| [技术选型](docs/technical-stack.md) | Rust 技术栈、依赖取舍和引入顺序 |
| [技术架构](docs/architecture.md) | 核心接口、模型与上下文、事件模型、完成判定和开发计划 |

[Apache-2.0](LICENSE)
