# WorkBuddy 平台规范速查

**两份规范尚未到手**——《Expert 开发规范 v2.3》与《Connector 对接规范 v3.0》，以共创营渠道版本为准。下面的条目整理自现有材料，拿到规范后须逐条核对。

## 模块选型（白皮书第 13、16–22 页；Expert 规范第 17–26 页）

| 模块 | 结论 |
|---|---|
| Expert | 首版必须 |
| 内置 MCP | 首版必须，Expert 内置依赖 |
| Skill | 首版必须，随 Expert 分发 |
| 独立 Connector | 首版不单独上架 |
| Playbook | 各准备 3–5 个 |

## Expert 包结构与约束（Expert 规范第 2–3、8–11 页）

必须包含：`.codebuddy-plugin/plugin.json` / `avatars/expert.png`（512×512 PNG/JPG ≤500KB，包内文件不可外链）/ `agents/vvic-product-finder.md` / `skills/vvic-product-search/SKILL.md` / `.mcp.json` / `README.md`。`agents/`、`skills/`、`bin/` 须在插件**根目录**，不得放入 `.codebuddy-plugin/`。禁止 `hooks/`、`commands/`、`.lsp.json`。

## plugin.json 字段约束（Expert 规范第 8–11、27–31 页）

| 字段 | 约束 |
|---|---|
| `name` | 全局唯一小写 kebab-case；上架后不可改 |
| `agentName` / Agent 文件名 / frontmatter `name` | 三者完全一致，否则 Agent 无法加载 |
| `categoryId` | `07-SalesCommerce` |
| `defaultInitPrompt` | 与 `quickPrompts[0]` 字符级一致 |
| `tags` | 恰好 3 个（规范「3–5 个」，本项目取更严格的 3 个） |
| `auth.type` | `none`（首版匿名） |

密钥禁止硬编码（搜款网凭证由 MCP 网关持有）；frontmatter 禁止声明 `tools` 字段（Expert 规范第 29–31 页）。

## 传输与性能基线（Connector 规范第 3–5 页）

延迟基线针对服务端耗时还是店主感受到的端到端时间，**待向 WorkBuddy 平台确认**。这两个口径差得很远——找款链路里绝大部分时间不在服务端，按哪个口径答复决定了要不要做优化。

| 指标 | 规范要求 | 状态 |
|---|---|---|
| 传输协议 | Streamable HTTP / SSE；生产必须 HTTPS | — |
| QPS | ≥ 50 | 未压测 |
| P50 延迟 | ≤ 500ms | 未压测，口径待确认 |
| P99 延迟 | ≤ 3000ms | 未压测 |
| 超时率 | < 1% | 未压测 |
| 错误率 | ≤ 0.5% | 未压测 |
| 并发长连接 | ≥ 200 条 | 未压测 |
| 单次请求 | 建议 30 秒内 | — |

## 其他约束与禁止

**Skill**（Connector 规范第 11–13 页）：必填 `name`/`description`/`version`/`author`；独立 Skill 须加 `category`（内置是否必填**待确认**）；不得含密钥或内部 URL。

**提交与运营**：ZIP → 自动校验 → 人工审核 → 上架；压测报告须含测试环境、QPS、P50/P95/P99、错误率、资源占用、持续时间；同步提交 3–5 个 Playbook；内置 Skill 运营要求**待确认**。

**禁止**（Connector 规范第 25–26 页）：虚假声明能力、恶意采集用户数据、夹带营销。

**OAuth（首版不适用）**（Connector 规范第 34–43 页）：OAuth 2.1 + PKCE；access token 约 1 小时，refresh token ≥30 天；未认证请求返回 401 + `resource_metadata`。
