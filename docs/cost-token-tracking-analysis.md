# 费用追踪与 Token 管理 - 深度分析

## 6.1 功能概述

费用追踪与 Token 管理模块负责精确记录每次 API 调用的 token 用量和费用，支持按模型分类统计（input/output/cache_read/cache_creation/web_search）、会话费用持久化与恢复、预算控制（`--max-budget-usd`）以及费用格式化展示。费用数据存储在项目配置中，支持 `--resume` 时恢复累计费用。系统还集成了 OpenTelemetry 计数器用于可观测性。

## 6.2 核心流程图

```mermaid
flowchart TD
    A[API 响应返回 usage] --> B[addToTotalSessionCost]
    B --> C[calculateUSDCost<br/>按模型计算费用]
    C --> D[addToTotalModelUsage<br/>按模型累计 token]
    D --> E[addToTotalCostState<br/>更新全局费用状态]

    E --> F{检查预算}
    F -->|超过 maxBudgetUsd| G[queryLoop 终止<br/>返回 error_max_budget_usd]
    F -->|未超过| H[继续]

    I[会话结束] --> J[saveCurrentSessionCosts<br/>持久化到项目配置]
    K[--resume] --> L[restoreCostStateForSession<br/>恢复累计费用]

    M[/cost 命令] --> N[formatTotalCost<br/>格式化展示]

    subgraph Token统计["Token 统计维度"]
        T1["input_tokens"]
        T2["output_tokens"]
        T3["cache_read_input_tokens"]
        T4["cache_creation_input_tokens"]
        T5["web_search_requests"]
    end

    style B fill:#e1f5fe
    style F fill:#fff3e0
```

## 6.3 核心调用链

```
claude.ts queryModel()                         # API 响应
  → message_delta event → usage
  → addToTotalSessionCost(cost, usage, model)  # src/cost-tracker.ts:L278
      → calculateUSDCost(model, usage)         # src/utils/modelCost.ts
      → addToTotalModelUsage()                 # 按模型累计
      → addToTotalCostState()                  # src/bootstrap/state.ts
      → getCostCounter()?.add()                # OpenTelemetry 计数器

queryLoop()                                    # src/query.ts
  → getTotalCost() >= maxBudgetUsd?            # 预算检查
  → yield error_max_budget_usd result          # 超预算终止

saveCurrentSessionCosts()                      # 会话结束时
  → saveCurrentProjectConfig()                 # 持久化到 .claude/projects/
```

## 6.4 关键数据结构

```typescript
// 按模型的用量统计
type ModelUsage = {
  inputTokens: number
  outputTokens: number
  cacheReadInputTokens: number
  cacheCreationInputTokens: number
  webSearchRequests: number
  costUSD: number
  contextWindow: number       // 模型上下文窗口大小
  maxOutputTokens: number     // 模型最大输出 token
}

// 持久化的费用状态
type StoredCostState = {
  totalCostUSD: number
  totalAPIDuration: number
  totalToolDuration: number
  totalLinesAdded: number
  totalLinesRemoved: number
  modelUsage: { [modelName: string]: ModelUsage }
}
```

## 6.5 设计决策分析

- 按模型分类统计：不同模型价格不同（Opus vs Sonnet vs Haiku），需要分别计算
- 会话持久化：费用存储在项目配置中（`lastCost`/`lastModelUsage`），`--resume` 时恢复
- 预算控制在 queryLoop 层：每轮工具执行后检查，而非每次 API 调用后，减少检查频率
- OpenTelemetry 集成：通过 `getCostCounter()`/`getTokenCounter()` 暴露给可观测性系统

## 6.7 关键代码位置索引

| 文件 | 关键内容 |
|------|---------|
| `src/cost-tracker.ts` | 费用追踪核心（累计、格式化、持久化） |
| `src/costHook.ts` | 费用阈值 Hook |
| `src/utils/modelCost.ts` | 模型价格表与费用计算 |
| `src/services/tokenEstimation.ts` | Token 估算 |
| `src/utils/tokens.ts` | Token 计数工具 |
| `src/bootstrap/state.ts` | 全局费用状态存储 |
| `src/commands/cost/` | /cost 命令实现 |
