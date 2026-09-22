---
name: bk-cli-aidev
description: 当需要通过 `bk-cli aidev` 查询或管理蓝鲸 AIDev 空间、Skill、MCP、Prompt、智能体、知识库、知识文档、凭证、资源引用，或上传/导入相关制品时使用。
---

# bk-cli aidev — 蓝鲸 AIDev 资源能力

用于通过 `bk-cli aidev` 调用蓝鲸 AIDev 私有 OpenAPI。

CRITICAL — 开始前 MUST 先读取 `../bk-cli-shared/SKILL.md`。共享 skill 负责认证、context、tenant、stage、dry-run、verbose、header/body 和通用请求规则；本 skill 只补充 `aidev` 命令自己的语义与输入约定。

## 网关名

- 社区版网关名为 `bk-aidev`。
- 上云环境构建时如注入了匹配的 `BK_TE_DOMAIN`，CLI 会在请求执行层把 `bk-aidev` 兼容映射为 `bkaidev`。
- 调用者通常只需要使用 `bk-cli aidev ...`，不要手动改命令名或路径。

## 当前覆盖范围

`bk-cli aidev` 主要是 YAML action 集合，并包含一个 Go-implemented 文件上传 action，覆盖附件 OpenAPI 中的 AIDev 资源接口。常用资源包括：

- 空间：`retrieve_private_spaces`
- Skill：`list_private_skills`、`create_private_skills_upsert`、`retrieve_private_skills`、`delete_private_skills`、`retrieve_private_skills_download`、`retrieve_private_skills_versions`
- MCP：`list_private_mcps`、`create_private_mcps`、`retrieve_private_mcps`、`retrieve_private_mcps_register_json`、`retrieve_private_mcps_by_code`、`retrieve_private_resource_mcps`
- Prompt：`list_private_prompts`、`upsert_private_prompts`、`retrieve_private_prompts`
- 智能体：`list_private_agents`、`create_private_agents`、`retrieve_private_agents`、`delete_private_agents`、`publish_private_agents`、`update_private_agents`、`append_private_agent_admins`
- 知识库与知识文档：`create_private_knowledgebase`、`list_private_knowledgebase`、`query_private_knowledgebase`、`create_private_knowledges`、`list_private_knowledges` 等
- 凭证：`list_credentiale_types`、`list_credentials`、`create_credentials`、`update_credentials`、`delete_credentials` 等

完整命令列表以 `bk-cli aidev --help` 为准，单个命令参数以 `bk-cli aidev <action> --help` 为准。

## 输入约定和易错点

- 所有 AIDev action 都要求应用认证 + 用户认证 + 接口资源权限。调用前需要确保当前 context 的凭据完整，并且应用已申请对应 API 权限。
- CLI flag 名保持上游 OpenAPI 参数名不变，例如 `space_id`、`skill_id`、`mcp_type`、`agent_code`、`page_size`。
- 带复杂 request body 的 action 统一通过共享 `--body '<json>'` 传入；执行时要求请求体的 action 会在本地校验 `--body` 非空。
- 查看请求体结构时使用 `bk-cli aidev <action> -h --body-schema`。
- `create_private_upload` 是文件上传命令，使用 `--file`、`--space_id` 和可选 `--module` 发送 `multipart/form-data`，不使用 `--body`。

## 常用示例

```bash
bk-cli aidev retrieve_private_spaces --keyword demo
bk-cli aidev create_private_upload --file ./artifact.zip --space_id demo-space --module skill
bk-cli aidev list_private_skills --space_id demo-space --page 1 --page_size 20
bk-cli aidev retrieve_private_skills --skill_id 1 --space_id demo-space
bk-cli aidev create_private_skills_upsert --body '{"space_id":"demo-space","skill_code":"demo","skill_name":"Demo","description":"demo","url":"bkrepo://demo.zip"}'
bk-cli aidev list_private_mcps --space_id demo-space --mcp_type apigw
bk-cli aidev retrieve_private_mcps --mcp_type apigw --mcp_id 1 --space_id demo-space
bk-cli aidev list_private_agents --space_id demo-space --page 1 --page_size 20
bk-cli aidev retrieve_private_agents --agent_id 1 --space_id demo-space
bk-cli aidev list_credentials --space_id demo-space
bk-cli aidev query_private_knowledgebase --body '{"space_id":"demo-space","query":"hello"}'
```

## 排障建议

- 先用 `--dry-run` 确认请求 URL、stage、query 和 body 是否符合预期。
- 如果报权限错误，同时检查当前登录身份和应用是否已经申请该 AIDev API 权限。
- 如果缺少请求体，先运行 `bk-cli aidev <action> -h --body-schema` 查看字段，再补 `--body`。
