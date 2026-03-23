# MiroFish 文档（中文归档）

本文件夹包含项目 [README-EN.md](../../README-EN.md) 之外的详细指南。此文件为原 `docs/README.md` 内容的中文归档版本。

## 指南

| 文档 | 说明 |
|------|------|
| [WORKFLOW.md](../WORKFLOW.md) | 从种子上传到报告及可选智能体访谈的端到端流程 |
| [CONFIGURATION.md](../CONFIGURATION.md) | 环境变量、Flask 设置与调参项 |
| [API.md](../API.md) | REST API 参考，含 `curl` 与 JSON 示例 |

## 约定

**基础 URL**

除非另有说明，示例假定后端在本地运行：

- `BASE=http://localhost:5001`

若在 `.env` 中设置了 `FLASK_PORT`，请将 `5001` 替换为该端口值。

**占位 ID**

示例使用统一占位符；请用 API 返回的真实值替换：

| 占位符 | 含义 |
|--------|------|
| `proj_xxx` | 项目 ID（来自本体生成） |
| `task_xxx` | 异步任务 ID（图构建、仿真准备、报告生成） |
| `graph_xxx` | Zep 图 ID |
| `sim_xxx` | 仿真 ID |
| `report_xxx` | 报告 ID |

**响应结构**

多数接口返回 JSON，顶层含 `success` 布尔字段，以及 `data` 或 `error`。详见 [API.md](../API.md) 中各路由。

**异步操作**

图构建、仿真准备与报告生成在后台执行，并返回 `task_id`。请轮询相应任务或状态接口直至完成（见 [WORKFLOW.md](../WORKFLOW.md)）。
