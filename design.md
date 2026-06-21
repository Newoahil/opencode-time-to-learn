# time-to-learn Skill 设计文档

## 概述

`time-to-learn` 是一个 OpenCode skill，帮助用户在 Obsidian 的 iLearn 知识库中管理学习进度。通过对未知概念的交互式讲解和对话纠正，记录学习过程中的理解变化，并维护知识点的状态和关系图谱。

## 触发指令

| 指令 | 行为 |
|------|------|
| `/ttl` | 从 `waitTolearn` 文件夹随机选一篇笔记（优先选 `half` 状态） |
| `/ttl Kubernetes` | 指定学习名为 "Kubernetes" 的笔记 |

## 状态模型

| 状态 | 含义 | 位置 | 被选中优先级 |
|------|------|------|-------------|
| `notyet` | 尚未学习 | waitTolearn | 低 |
| `half` | 已学习但不完全掌握 | waitTolearn | 最高 |
| `done` | 完全掌握 | 移动到领域文件夹 | 不再被选中 |

状态通过 Obsidian 原生 frontmatter 属性 `status` 维护，使用 CLI 命令：
```bash
obsidian property:set name="status" value="done" file="Kubernetes"
```

## 核心工作流

```
1. 解析用户输入 → 确定目标笔记
2. 读取笔记内容 → 获取当前 status
3. 我对概念做基本讲解
4. 对话循环：
   - 用户提问或表达自己的理解
   - 我纠正错误、补充细节
   - 我持续跟踪理解程度和对话轮次
5. 对话结束判断（用户零次元操作，全程只参与学习对话）：
   a. 用户明确信号："懂了""明白了""差不多了""总结吧" → 立即进入总结
   b. 自动兜底：2 次用户正确复述/自我纠正 → 判定理解程度较高，自动总结
   c. 自动兜底：连续 2 轮无实质新问题 → 自动总结
   d. 兜底问询：对话超过 6 轮仍无结束信号 → 我问"差不多了吗，需要我总结一下？"
   e. 表述含糊（"还行""大概懂了"）→ 不立即总结，追问一个确认问题后再决定
6. 学习总结（agent 全自动完成）：
   a. 根据对话质量自动判定 status（done/half）
   b. 生成总结追加到笔记末尾
   c. 若 status=done → 寻找或创建领域文件夹 → 移动笔记
   d. 识别 vault 中已有关联知识点 → 双向 wikilink
   e. 向用户报告完成：本次学习总结已记录 ✓ 状态: half/done
```

**核心原则：你只参与学习对话，整理工作全由 agent 自主完成。**

## 学习总结原则

**核心目标：笔记是独立的可回看学习资料。** 用户忘记该知识点后，仅通过阅读笔记就能重新掌握。

- **转述整理，不照搬对话**：将对话内容提炼为结构化知识，而非对话转录
- **自包含性**：笔记脱离对话上下文后仍可独立理解
- **渐进累积**：多次学习同一知识点时，新总结追加在旧总结之后，形成理解演进轨迹

## 学习总结格式

每次对话后在笔记末尾追加：

```markdown
## 学习记录 - 2026-06-20

### 核心概念
<转述性段落，用自己的话讲清楚这个知识点是什么、为什么重要、怎么用。
不是对话记录，而是独立的知识卡片。>

### 关键要点
- <要点 1：本质理解>
- <要点 2：与容易混淆概念的区分>
- <要点 3：使用场景 / 边界>

### 常见误区
| 误区 | 正确理解 |
|------|---------|
| <你最初以为的 A> | <实际是 B> |
| <容易与其他概念混淆的地方> | <区分方式> |

### 实践场景
- <你在工作中遇到的真实场景>
- <我补充的类比或适用场景>

### 关联知识
- [[Docker]] - 容器运行时，Kubernetes 是其编排层
- [[Microservices]] - Kubernetes 解决的核心问题域

### 掌握程度
half → done
```

## 领域文件夹匹配

1. 列出 vault 所有顶层文件夹（排除 `waitTolearn`、`.obsidian`、`.trash` 等系统目录）
2. 基于笔记标题/内容语义判断最匹配的现有文件夹
3. 宽容度 6/10：允许部分语义匹配
4. 无匹配则创建新文件夹

通过 `obsidian move file="Kubernetes" to="Container"` 完成移动。

## 知识关系维护

每次学习后：
1. 扫描 vault 中现有笔记
2. 识别与新概念语义相关的知识点
3. 在双方笔记中追加 wikilink：`[[目标笔记]]`
4. Obsidian 原生图谱自动渲染关系网

## 前置依赖

- Obsidian 必须正在运行
- 目标 vault 名称为 `iLearn`
- vault 中存在 `waitTolearn` 文件夹

## Obsidian CLI 操作清单

| 操作 | CLI 命令 |
|------|----------|
| 列出 waitTolearn 文件 | `obsidian vault="iLearn" files folder="waitTolearn"` |
| 读取笔记 | `obsidian vault="iLearn" read file="Kubernetes"` |
| 追加内容 | `obsidian vault="iLearn" append file="Kubernetes" content="..."` |
| 设置属性 | `obsidian vault="iLearn" property:set name="status" value="done" file="Kubernetes"` |
| 读取属性 | `obsidian vault="iLearn" property:read name="status" file="Kubernetes"` |
| 移动文件 | `obsidian vault="iLearn" move file="Kubernetes" to="Container"` |
| 列出文件夹 | `obsidian vault="iLearn" folders` |
| 搜索相关笔记 | `obsidian vault="iLearn" search query="容器"` |
