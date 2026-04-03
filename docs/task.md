# 核心模块深度分析任务

基于项目全貌分析，以下是需要逐一深度分析的核心模块列表。

## 任务列表

- [x] 1. 查询引擎与 AI 对话循环 - 管理用户消息提交、Claude API 调用、工具调用循环的核心引擎
- [x] 2. 工具系统 - 定义、注册、权限控制和执行 30+ 种内置工具
- [x] 3. 权限与安全系统 - 多层权限模式、工具调用分类器、文件系统沙箱
- [x] 4. MCP 集成 - Model Context Protocol 服务器连接管理、工具代理、资源读取
- [x] 5. 命令与技能系统 - 斜杠命令注册、技能加载、插件系统
- [x] 6. REPL 交互界面 - 基于 Ink 的终端 TUI 渲染与交互
- [x] 7. 状态管理 - 全局 AppState 存储与 React 状态绑定
- [x] 8. 上下文与提示词构建 - 系统提示词组装、Git 状态注入、CLAUDE.md 加载
- [x] 9. 多 Agent 协作（Swarm） - Agent 派生、团队管理、Teammate 通信
- [x] 10. 远程会话与 Bridge - 远程控制桥接、SSH 远程、WebSocket 传输
- [x] 11. 后台任务系统 - Shell/Agent/远程任务的生命周期管理
- [x] 12. 记忆与会话持久化 - 会话历史存储、记忆提取、跨会话记忆

- [x] 13. 上下文压缩系统 - 多层压缩策略（snip/microcompact/autocompact/reactive compact）管理长对话上下文
- [x] 14. 插件系统 - 插件加载、安装、marketplace 集成与生命周期管理
- [x] 15. 认证与 OAuth 系统 - OAuth 2.0 PKCE 流程、API Key 管理、企业认证
- [x] 16. 设置与配置系统 - 多层设置合并（user/project/local/policy/managed）与远程同步
- [x] 17. 遥测与分析系统 - 事件日志、GrowthBook feature flags、采样与 sink 架构
- [x] 18. 推测执行（Speculation） - 预测用户意图并提前执行查询，提升响应速度

- [x] 19. 非交互式模式（Print/SDK） - `-p` 模式和 SDK stream-json 协议的完整执行路径
- [x] 20. Hooks 生命周期系统 - SessionStart/PreToolUse/PostToolUse/Stop 等可扩展钩子
- [x] 21. LSP 集成 - 语言服务器协议集成，提供代码诊断与被动反馈
- [x] 22. 费用追踪与 Token 管理 - API 调用费用追踪、token 用量统计与预算控制
