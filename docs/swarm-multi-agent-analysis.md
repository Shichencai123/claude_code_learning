# 多 Agent 协作（Swarm） - 深度分析

## 6.1 功能概述

多 Agent 协作系统（Swarm）允许主 Agent 通过 `AgentTool` 派生子 Agent（Teammate），实现并行任务执行。支持两种运行模式：进程内模式（in-process，共享内存）和 tmux 模式（独立终端进程）。团队通过 `TeamCreateTool`/`TeamDeleteTool` 管理，Teammate 之间通过 `SendMessageTool` 通信。每个 Teammate 有独立的会话上下文、权限继承和工具集。

## 6.2 核心流程图

```mermaid
flowchart TD
    A[主 Agent] --> B[AgentTool.call]
    B --> C{运行模式?}

    C -->|in-process| D[spawnInProcess]
    D --> E[创建子 QueryEngine]
    E --> F[独立消息历史<br/>共享 AppState]

    C -->|tmux| G[tmux 新窗口]
    G --> H[启动新 claude 进程]
    H --> I[独立进程<br/>通过文件通信]

    B --> J[TaskState 追踪]
    J --> K[任务状态更新<br/>pending/running/completed]

    subgraph 团队管理["团队管理"]
        T1[TeamCreateTool] --> T2[创建团队<br/>设置 Teammate 模式]
        T3[TeamDeleteTool] --> T4[清理团队资源]
        T5[SendMessageTool] --> T6[Teammate 间消息传递]
    end

    style B fill:#e1f5fe
    style D fill:#e8f5e9
    style G fill:#fff3e0
```

## 6.3 核心调用链

```
AgentTool.call({ name, prompt })               # src/tools/AgentTool/AgentTool.ts
  → createSubagentContext()                    # 创建子 agent 上下文
  → runForkedAgent() / spawnInProcess()        # 执行子 agent
      → query() 循环                          # 独立的查询循环
  → 收集结果 → 返回给主 agent

TeamCreateTool.call({ name })                  # src/tools/TeamCreateTool/
  → setDynamicTeamContext()                    # 设置团队上下文
  → 配置 Teammate 模式（in-process/tmux）
```

## 6.7 关键代码位置索引

| 文件 | 关键内容 |
|------|---------|
| `src/tools/AgentTool/` | Agent 派生工具、Agent 定义加载 |
| `src/utils/swarm/` | Swarm 核心：进程内运行、tmux 运行、权限同步 |
| `src/utils/swarm/spawnInProcess.ts` | 进程内 Teammate 启动 |
| `src/utils/swarm/teammateInit.ts` | Teammate 初始化 |
| `src/utils/swarm/leaderPermissionBridge.ts` | Leader-Teammate 权限桥接 |
| `src/tools/TeamCreateTool/` | 团队创建工具 |
| `src/tools/SendMessageTool/` | Teammate 间消息工具 |
| `src/tasks/InProcessTeammateTask/` | 进程内 Teammate 任务 |
