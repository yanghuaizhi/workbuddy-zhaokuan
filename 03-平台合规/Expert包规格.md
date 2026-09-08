# Expert 包规格

照着建文件的操作手册。骨架完整保留，`<占位符>` 标注需要填写的位置。包的装配归怀志。

## 目录树

```
vvic-product-finder/           ← ZIP 打包时以此目录为根
├── .codebuddy-plugin/
│   └── plugin.json            ← 元数据与配置
├── avatars/
│   └── expert.png             ← 512×512 PNG/JPG，≤500KB，包内文件不可用外链
├── agents/
│   └── vvic-product-finder.md ← 文件名须与 agentName 一致
├── skills/
│   └── vvic-product-search/
│       └── SKILL.md
├── .mcp.json
└── README.md
```

`agents/`、`skills/`、`bin/` 必须在插件**根目录**，不能放入 `.codebuddy-plugin/`。禁止包含 `hooks/`、`commands/`、`.lsp.json`——导致审核失败。

打包：整个 `vvic-product-finder/` 压缩为 ZIP，ZIP 根目录直接包含各文件，不多套目录层。

## plugin.json 骨架

字段约束：`name` 全局唯一小写 kebab-case（上架后不可改）；`agentName`、Agent 文件名（去掉 `.md`）、Agent frontmatter `name` 三者必须完全一致；`categoryId` 固定 `07-SalesCommerce`；`tags` 恰好 3 个；`defaultInitPrompt` 与 `quickPrompts[0]` 字符级完全一致；`auth.type` 为 `none`（首版匿名）。**密钥禁止硬编码**，搜款网凭证由 MCP 网关服务端持有。

```json
{
  "name": "vvic-product-finder",
  "agentName": "vvic-product-finder",
  "displayName": "<对外显示的名称，如「搜款网找款助手」>",
  "description": "<一句话描述，面向店主，说清楚能做什么>",
  "version": "1.0.0",
  "categoryId": "07-SalesCommerce",
  "tags": [
    "<标签1，恰好3个>",
    "<标签2>",
    "<标签3>"
  ],
  "avatars": {
    "expert": "avatars/expert.png"
  },
  "defaultInitPrompt": "<与 quickPrompts 第一条完全一致的文本>",
  "quickPrompts": [
    "<第一条引导语，与 defaultInitPrompt 完全一致>",
    "<第二条引导语>",
    "<第三条引导语>"
  ],
  "dependencies": {
    "mcpServers": {
      "zhaokuan-find-goods": {
        "type": "streamableHttp",
        "url": "<生产环境 MCP 端点地址，部署后填入>",
        "x-workbuddy": {
          "displayName": "搜款网找款服务",
          "description": "调用搜款网图搜与文字找款能力",
          "auth": {
            "type": "none"
          }
        }
      }
    }
  }
}
```

## Agent 定义文件骨架

路径 `agents/vvic-product-finder.md`。**frontmatter 禁止声明 `tools` 字段**——工具由系统统一分配。

```markdown
---
name: vvic-product-finder
description: <描述这个 Agent 做什么，面向平台审核者，说清楚能力边界>
---

<Agent 的系统提示（System Prompt）写在这里>

<说明这个 Agent 的主要能力、使用方式、以及在什么情况下应该用哪个工具>

<示例：
- 店主发图片时，先申请上传入口换取图片地址，再调 vvic_find_goods_by_image
- 店主用文字描述时，调 vvic_find_goods_by_text
- 收到候选表格后，原样转发服务端返回的表格，不重排、不重写、不另外生成报告
>
```

## SKILL.md 骨架

路径 `skills/vvic-product-search/SKILL.md`

```markdown
---
name: vvic-product-search
description: <一句话说明 Skill 能做什么>
version: 1.0.0
author: <团队名称或个人>
category: <填写适用分类，独立 Skill 必填，内置 Skill 待确认是否必填>
---

## 能力说明

<这个 Skill 能做什么、不能做什么>

## 工具列表

### create_image_upload

- 用途：申请限时上传入口，店主的本地图片先传上去换取可搜地址
- 说明：图片不能直接作为工具参数传入，必须先走这一步

### vvic_find_goods_by_image

- 用途：以图找相似款
- 入参：
  - `image_url`：上一步换来的图片地址
  - `city`：市场代码，`gz`（广州沙河，默认）或 `pn`
- 出参：候选列表及渲染好的表格

### vvic_find_goods_by_text

- 用途：按关键词找款，也用于二次收窄
- 入参：
  - `keyword`：搜索关键词
  - `city`：市场代码
  - 筛选条件：代发、价格区间等
- 出参：候选列表及渲染好的表格
- 注意：这条通道返回的价格字段是空的

## 使用示例

<给出 2–3 个完整的对话示例，包括店主输入和助手响应>

## 常见错误处理

| 错误状态 | 含义 | 建议动作 |
|---|---|---|
| `invalid_image` | 图片格式或尺寸不符合要求 | 告知店主具体是哪里不合适，引导重传 |
| `empty` | 没有找到相似款 | 建议换一张更清楚的图 |
| `upstream_error` | 上游服务异常 | 按错误消息判断是否可重试 |
```

## .mcp.json 骨架

路径 `.mcp.json`（插件根目录）。`x-workbuddy` 仅供平台展示读取，**不会写入用户的最终 MCP 配置**。

```json
{
  "mcpServers": {
    "zhaokuan-find-goods": {
      "type": "streamableHttp",
      "url": "<生产环境 MCP 端点 URL>",
      "x-workbuddy": {
        "displayName": "搜款网找款服务",
        "description": "为服装网店店主提供以图找款和文字找款能力，货源覆盖广州沙河服装批发市场",
        "auth": {
          "type": "none"
        }
      }
    }
  }
}
```
