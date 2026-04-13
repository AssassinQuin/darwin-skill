![Auto Skill Optimizer](assets/banner.svg)

# Auto Skill Optimizer

**像训练模型一样优化你的 Claude Code Skills。**

受 [Andrej Karpathy 的 autoresearch](https://github.com/karpathy/autoresearch) 启发，将自主实验循环从模型训练搬到 Skill 优化领域。一个只能向前转的棘轮。

> [**查看可视化展示页 →**](https://alchaincyf.github.io/auto-skill-optimizer/)

---

## 核心循环

```mermaid
graph LR
    A["🔍 评估"] --> B["🔧 改进"]
    B --> C["🧪 实测验证"]
    C --> D["👤 人类确认"]
    D --> E{分数提升?}
    E -->|Yes| F["✅ 保留 commit"]
    E -->|No| G["↩️ git revert"]
    F --> A
    G --> A

    style A fill:#D4532B,color:#fff,stroke:none
    style B fill:#333,color:#fff,stroke:none
    style C fill:#333,color:#fff,stroke:none
    style D fill:#333,color:#fff,stroke:none
    style E fill:#fafafa,color:#111,stroke:#D4532B,stroke-width:2px
    style F fill:#2B8A3E,color:#fff,stroke:none
    style G fill:#C92A2A,color:#fff,stroke:none
```

---

## 为什么做这个

Claude Code 的 Skill 生态在快速扩张。当你有 10 个 Skills 时可以手动维护；当你有 60+ 个 Skills 时，你需要一个系统。

传统的 Skill 审查是**纯结构性的**：检查格式对不对、步骤有没有编号、路径能不能访问。但一个格式完美的 Skill，跑出来的效果可能很差。

Auto Skill Optimizer 同时评估**结构质量**和**实际效果**，然后只保留真正有改进的修改。

---

## 从 autoresearch 到 Skill Optimizer

```mermaid
graph TB
    subgraph autoresearch["🔬 Karpathy autoresearch"]
        direction TB
        AR1["program.md<br/>目标与约束"]
        AR2["train.py<br/>被优化的代码"]
        AR3["val_bpb<br/>评估指标"]
        AR4["git ratchet<br/>只进不退"]
    end

    subgraph optimizer["⚡ Auto Skill Optimizer"]
        direction TB
        SO1["SKILL.md<br/>评估标准与约束"]
        SO2["每个 SKILL.md<br/>被优化的资产"]
        SO3["8 维加权总分<br/>结构 60 + 效果 40"]
        SO4["keep / revert<br/>只保留改进"]
    end

    AR1 -.->|映射| SO1
    AR2 -.->|映射| SO2
    AR3 -.->|映射| SO3
    AR4 -.->|映射| SO4

    style autoresearch fill:#fafafa,stroke:#999,stroke-width:1px
    style optimizer fill:#fdf2ec,stroke:#D4532B,stroke-width:2px
    style AR1 fill:#f5f5f5,stroke:#999,color:#333
    style AR2 fill:#f5f5f5,stroke:#999,color:#333
    style AR3 fill:#f5f5f5,stroke:#999,color:#333
    style AR4 fill:#f5f5f5,stroke:#999,color:#333
    style SO1 fill:#D4532B,color:#fff,stroke:none
    style SO2 fill:#D4532B,color:#fff,stroke:none
    style SO3 fill:#D4532B,color:#fff,stroke:none
    style SO4 fill:#D4532B,color:#fff,stroke:none
```

关键区别：autoresearch 全自主运行（loss 可以自动比较），Skill 优化增加了**人在回路**。因为 Skill 的「好坏」不像 loss 那样可以纯数值判断。

---

## 五条核心原则

```
01  单一可编辑资产    每次只改一个 SKILL.md，变量可控，改进可归因
02  双重评估         结构评分（静态分析）+ 效果验证（跑测试看输出）
03  棘轮机制         只保留改进，自动回滚退步，分数只升不降
04  独立评分         评分用子 agent，避免「自己改自己评」的偏差
05  人在回路         每个 Skill 优化完后暂停，用户确认再继续
```

---

## 8 维度评估体系

总分 100。结构维度靠静态分析（60分），效果维度必须实测（40分）。

```mermaid
graph LR
    subgraph structure["结构维度 — 60 分"]
        S1["Frontmatter 质量<br/><b>8</b>"]
        S2["工作流清晰度<br/><b>15</b>"]
        S3["边界条件覆盖<br/><b>10</b>"]
        S4["检查点设计<br/><b>7</b>"]
        S5["指令具体性<br/><b>15</b>"]
        S6["资源整合度<br/><b>5</b>"]
    end

    subgraph effect["效果维度 — 40 分"]
        E1["整体架构<br/><b>15</b>"]
        E2["🎯 实测表现<br/><b>25</b>"]
    end

    structure --> TOTAL["总分<br/><b>100</b>"]
    effect --> TOTAL

    style structure fill:#f5f5f5,stroke:#999,stroke-width:1px
    style effect fill:#fdf2ec,stroke:#D4532B,stroke-width:2px
    style TOTAL fill:#D4532B,color:#fff,stroke:none
    style E2 fill:#D4532B,color:#fff,stroke:none
    style S1 fill:#fff,stroke:#ddd
    style S2 fill:#fff,stroke:#ddd
    style S3 fill:#fff,stroke:#ddd
    style S4 fill:#fff,stroke:#ddd
    style S5 fill:#fff,stroke:#ddd
    style S6 fill:#fff,stroke:#ddd
    style E1 fill:#fff,stroke:#ddd
```

> 实测表现权重最高（25分）。Skill 写得再漂亮，跑出来效果不好就是零。

---

## 优化循环：5 个阶段

```mermaid
graph TB
    P0["<b>Phase 0</b><br/>初始化<br/>确定范围 · 创建 git 分支 · 加载历史"]
    P05["<b>Phase 0.5</b><br/>测试设计<br/>为每个 Skill 设计 2-3 个测试 prompt"]
    P1["<b>Phase 1</b><br/>基线评估<br/>8 维度打分 · 建立优化前基准线"]
    P2["<b>Phase 2</b><br/>优化循环<br/>诊断 → 改进 → 重评 → keep/revert"]
    P3["<b>Phase 3</b><br/>汇总报告<br/>Before/After 分数表 · 关键改进摘要"]

    P0 --> P05
    P05 -->|"👤 人类确认测试 prompt"| P1
    P1 -->|"👤 人类确认基线分数"| P2
    P2 -->|"👤 每个 Skill 完成后确认"| P3

    style P0 fill:#f5f5f5,stroke:#999,color:#333
    style P05 fill:#f5f5f5,stroke:#999,color:#333
    style P1 fill:#333,color:#fff,stroke:none
    style P2 fill:#D4532B,color:#fff,stroke:none
    style P3 fill:#333,color:#fff,stroke:none
```

**Phase 2 的核心逻辑**：

```mermaid
graph TB
    A["找出得分最低的维度"] --> B["生成 1 个具体改进方案"]
    B --> C["编辑 SKILL.md · git commit"]
    C --> D["子 agent 独立重新评分"]
    D --> E{"新分 > 旧分?"}
    E -->|"Yes ✅"| F["保留改进"]
    E -->|"No ❌"| G["git revert"]
    F --> H{"达到 3 轮?"}
    G --> I["跳到下一个 Skill"]
    H -->|No| A
    H -->|Yes| I
    I --> J["👤 展示 diff + 分数变化<br/>等待人类确认"]

    style A fill:#333,color:#fff,stroke:none
    style E fill:#fafafa,stroke:#D4532B,stroke-width:2px,color:#111
    style F fill:#2B8A3E,color:#fff,stroke:none
    style G fill:#C92A2A,color:#fff,stroke:none
    style J fill:#D4532B,color:#fff,stroke:none
```

---

## 棘轮机制

分数只能上升。每一轮要么改进 Skill，要么干净地回滚。不会随时间积累局部退化。

```mermaid
graph LR
    R0["72<br/><small>基线</small>"]
    R1["78<br/><small>✅ 保留</small>"]
    R2["<s>75</s><br/><small>❌ 回滚</small>"]
    R3["84<br/><small>✅ 保留</small>"]
    R4["87<br/><small>✅ 保留</small>"]

    R0 --> R1 --> R2 --> R3 --> R4

    style R0 fill:#666,color:#fff,stroke:none
    style R1 fill:#2B8A3E,color:#fff,stroke:none
    style R2 fill:#fff,color:#C92A2A,stroke:#C92A2A,stroke-width:2px,stroke-dasharray:5 5
    style R3 fill:#2B8A3E,color:#fff,stroke:none
    style R4 fill:#2B8A3E,color:#fff,stroke:none
```

轮次 2 的 75 分低于当前最优的 78 分，被自动回滚。有效基线始终锁定在 78，后续改进从 78 继续。

---

## 为什么需要双重评估

```mermaid
graph TB
    subgraph old["❌ 纯结构审查"]
        O1["检查 frontmatter 格式"]
        O2["验证步骤编号"]
        O3["确认路径有效"]
        O4["⚠️ 无法判断实际输出质量"]
        O5["⚠️ 无法检测过度约束"]
    end

    subgraph new["✅ 双重评估"]
        N1["结构评分 + 实测验证"]
        N2["跑真实 prompt 对比输出"]
        N3["子 agent 独立评分"]
        N4["每轮只改一个维度"]
        N5["分数不涨就回滚"]
    end

    style old fill:#f9f9f9,stroke:#ccc,stroke-width:1px
    style new fill:#fdf2ec,stroke:#D4532B,stroke-width:2px
    style O4 fill:#fff5f5,stroke:#C92A2A,color:#C92A2A
    style O5 fill:#fff5f5,stroke:#C92A2A,color:#C92A2A
    style N1 fill:#D4532B,color:#fff,stroke:none
    style N2 fill:#D4532B,color:#fff,stroke:none
    style N3 fill:#D4532B,color:#fff,stroke:none
```

---

## 快速开始

### 安装

```bash
# 将 SKILL.md 放入 Claude Code Skills 目录
mkdir -p ~/.claude/skills/huashu-auto-skill-optimizer
cp SKILL.md ~/.claude/skills/huashu-auto-skill-optimizer/SKILL.md
```

### 使用

```
# 评估所有 Skills（只评估不改）
> 评估所有 skills

# 优化指定 Skill
> 优化 huashu-slides 这个 skill

# 全量优化（推荐首次使用）
> 优化所有 skills

# 查看历史
> 看看 skill 优化历史
```

### 输出示例

```
┌──────────────────────────┬────────┬────────┬────────┐
│ Skill                    │ Before │ After  │ Δ      │
├──────────────────────────┼────────┼────────┼────────┤
│ huashu-proofreading      │ 78     │ 87     │ +9     │
│ huashu-slides            │ 72     │ 83     │ +11    │
│ huashu-publish           │ 81     │ 88     │ +7     │
├──────────────────────────┼────────┼────────┼────────┤
│ 平均                     │ 77     │ 86     │ +9     │
└──────────────────────────┴────────┴────────┴────────┘
```

---

## 设计灵感

这个项目的设计直接受 **Andrej Karpathy 的 [autoresearch](https://github.com/karpathy/autoresearch)** 启发。

autoresearch 证明了一个优雅的想法：你可以把「写论文」这件事变成一个自主实验循环。定义目标（`program.md`），让 agent 不断生成和测试变更（`train.py`），用可量化的指标（`val_bpb`）决定保留还是回滚。

Auto Skill Optimizer 把同样的思路搬到了 Claude Code Skill 优化。区别在于：

1. **评估更复杂**：需要 8 个维度的加权评分，单一数值说不清楚
2. **需要实测**：结构评分只是一半，另一半必须跑真实 prompt 看效果
3. **人在回路**：Skill 的「好」是主观的，需要人来做最终判断

核心机制完全相同：**只保留可测量的改进，其余全部回滚。**

---

## 约束规则

1. 不改变 Skill 的核心功能和用途
2. 不引入新依赖
3. 每轮只改一个维度，避免多变更无法归因
4. 优化后 SKILL.md 不超过原始大小的 150%
5. 所有改动在 git 分支上，用 git revert 回滚
6. 效果维度必须用子 agent 评分，不能自己改完自己评

---

## 文件结构

```
auto-skill-optimizer/
├── README.md              # 你正在看的文件
├── SKILL.md               # 核心：评估标准 + 优化流程 + 约束规则
├── showcase.html          # Pentagram 风格的可视化展示页（可本地打开）
└── assets/
    └── aso-hero.png       # README 配图
```

---

## 致谢

- [Andrej Karpathy](https://github.com/karpathy) 的 [autoresearch](https://github.com/karpathy/autoresearch) 提供了核心设计灵感
- [Claude Code](https://claude.ai/code) 的 Skill 生态提供了优化场景
- [花叔](https://x.com/AlchainHust) 的 60+ Skills 实践提供了真实测试环境

---

**License**: MIT
