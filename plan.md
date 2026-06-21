# time-to-learn Skill 实施计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 创建 `time-to-learn` skill，使 OpenCode 能通过 `/ttl` 指令与用户进行交互式学习对话，管理 Obsidian iLearn 知识库中的笔记状态和知识关系图谱。

**Architecture:** 单一 SKILL.md 文件定义完整工作流。通过 Obsidian CLI 操作笔记文件（读取、追加、属性设置、移动），配合 wikilink 维护知识点关系。

**Tech Stack:** OpenCode Skill 系统、Obsidian CLI

---

## 文件结构

| 文件 | 用途 |
|------|------|
| Create: `C:\Users\Homan\.opencode\skills\time-to-learn\SKILL.md` | Skill 定义文件 |
| Exists: `D:\works\time-to-learn\design.md` | 设计文档 |
| Exists: `C:\Users\Homan\.opencode\skills\obsidian-skills\skills\obsidian-cli\SKILL.md` | 依赖的 obsidian-cli skill |

---

### Task 1: 准备测试场景 (RED Phase)

**目标：** 编写压力测试场景，在不加载 skill 的情况下建立 baseline（确认无 skill 时 agent 不会正确处理 /ttl 指令）。

- [ ] **Step 1: 创建测试场景文档**

```markdown
# 测试场景文档

保存到: D:\works\time-to-learn\test-scenarios.md

## 场景 1: /ttl 随机选择 + 完整对话流
1. 用户在 waitTolearn 中有 3 篇笔记: "Kubernetes" (notyet), "Docker" (notyet), "gRPC" (half)
2. 用户输入 /ttl
3. 期望: 优先选择 "gRPC" (half 优先)
4. 期望: 读取笔记内容后进行基本讲解
5. 期望: 用户提问后 agent 纠正并对话
6. 期望: 用户说"总结吧" → agent 立即总结、更新状态、双向 wikilink
7. 期望: 用户全程不操作文件，agent 全自动完成整理

## 场景 2: /ttl 指定知识点
1. 用户输入 /ttl Kubernetes
2. 期望: 直接选中 "Kubernetes" 笔记

## 场景 3: status=done 移动笔记
1. 学习完成后 agent 自动判定 status=done
2. 期望: 笔记从 waitTolearn 移到匹配的领域文件夹
3. 期望: vault 无匹配文件夹时自动创建

## 场景 4: 空 waitTolearn
1. waitTolearn 文件夹为空
2. 用户输入 /ttl
3. 期望: 告知用户无待学习笔记

## 场景 5: 知识关系维护
1. 完成 "Kubernetes" 学习后
2. vault 中已存在 "Docker" 笔记
3. 期望: 在 "Kubernetes" 和 "Docker" 间建立双向 wikilink

## 场景 6: 自动兜底结束判断
1. 用户连续正确复述 2 次 → agent 自动判定理解程度高，进入总结
2. 对话超过 6 轮无结束信号 → agent 主动询问"差不多了吗？"
3. 用户说"还行" → agent 追问确认后再决定是否总结

## 场景 7: 状态自动判定
1. 对话中用户表达清晰、无重大误解 → agent 自动设 status=done
2. 对话中用户仍有模糊点 → agent 自动设 status=half
3. 用户全程不手动设置状态
```

- [ ] **Step 2: 运行 Baseline 测试（无 skill）**

用 subagent 模拟 `/ttl` 指令，不加载 time-to-learn skill，记录行为：
- agent 是否能理解 `/ttl` 的含义？
- agent 是否自动知道要操作 Obsidian vault？
- agent 是否知道状态模型和文件移动规则？

记录 baseline 结果到 `D:\works\time-to-learn\baseline-results.md`。

---

### Task 2: 编写 SKILL.md (GREEN Phase)

**目标：** 编写完整的 skill 定义文件。

**Files:**
- Create: `C:\Users\Homan\.opencode\skills\time-to-learn\SKILL.md`

- [ ] **Step 1: 创建 skill 目录**

```bash
New-Item -ItemType Directory -Path "C:\Users\Homan\.opencode\skills\time-to-learn" -Force
```

- [ ] **Step 2: 编写 SKILL.md**

将以下内容写入 `C:\Users\Homan\.opencode\skills\time-to-learn\SKILL.md`：

```
---
name: time-to-learn
description: Use when user types /ttl command or wants to learn a concept through interactive dialogue — manages Obsidian iLearn vault, maintains note status (notyet/half/done), moves notes between folders, and builds knowledge relationship graph via wikilinks
---

# time-to-learn

## 概述

交互式学习 skill。通过对话讲解概念、纠正理解，管理 Obsidian iLearn 知识库中的学习进度和知识图谱。

## 触发指令

- `/ttl` — 从 `waitTolearn` 随机选一篇笔记（优先 `half` 状态）
- `/ttl <知识点>` — 指定学习某篇笔记

## 状态模型

| 状态 | 含义 | 位置 | 选择优先级 |
|------|------|------|-----------|
| `notyet` | 尚未学习 | waitTolearn | 低 |
| `half` | 部分掌握 | waitTolearn | 最高 |
| `done` | 完全掌握 | 领域文件夹 | 不再选中 |

状态通过 Obsidian 原生 frontmatter 属性 `status` 维护。

## 核心工作流

### 1. 选中笔记

```
解析 /ttl [知识点] → 确定目标笔记
  - 无参数: 列出 waitTolearn 下所有文件，优先选 status=half，否则随机
  - 有参数: 在 waitTolearn 中按名称查找
  - 若 waitTolearn 为空: 告知用户无待学习笔记
```

Vault: `C:\Users\Homan\iLearn`，使用 `vault="iLearn"` 参数。

### 2. 读取与讲解

```bash
# 读取笔记内容
obsidian vault="iLearn" read file="<笔记名>"

# 读取当前状态
obsidian vault="iLearn" property:read name="status" file="<笔记名>"
```

然后对概念做基本讲解（2-4 句核心要点）。

### 3. 对话学习

- 用户提问或表达自己的理解
- 我纠正错误、补充细节、举例说明
- **持续跟踪：** 记录用户的理解过程、误区、举过的例子、复述正确次数
- **核心原则：用户只参与对话，不操作文件。整理全由 agent 自主完成。**

#### 对话结束判断

```
自然结束（满足任一即自动进入总结，不询问用户）：
  - 用户明确信号："懂了""明白了""差不多了""总结吧"
  - 用户连续 2 次正确复述/自我纠正 → 理解程度高，自动总结
  - 连续 2 轮无实质新问题 → 对话自然枯竭，自动总结

兜底问询：
  - 对话超过 6 轮仍无结束信号 → 主动问"差不多了吗，需要我总结一下？"
  - 用户表述含糊（"还行""大概懂了"）→ 追问一个确认问题，再决定

状态自动判定（不询问用户）：
  - 用户表达清晰、自我纠正准确 → status=done
  - 用户仍有模糊/不确定 → status=half
```

### 4. 学习总结（Agent 全自动）

**核心原则：笔记是独立的学习资料，不是对话记录。** 用户忘记该知识点后，仅通过阅读笔记就能重新掌握。

- **转述整理**：提炼对话为结构化知识卡片，不照搬对话原文
- **自包含**：笔记脱离对话上下文仍可独立理解
- **渐进累积**：多次学习追加在旧记录之后，形成理解演进轨迹

在笔记末尾追加 `## 学习记录 - YYYY-MM-DD` 章节：

```markdown
## 学习记录 - YYYY-MM-DD

### 核心概念
<转述性段落，用自己的话讲清楚这个知识点是什么、为什么重要、怎么用>

### 关键要点
- <要点 1：本质理解>
- <要点 2：与容易混淆概念的区分>
- <要点 3：使用场景 / 边界>

### 常见误区
| 误区 | 正确理解 |
|------|---------|
| <用户最初的理解偏差> | <纠正后的正确理解> |

### 实践场景
- <用户在对话中提到的真实场景>
- <我补充的类比或适用场景>

### 关联知识
- [[相关笔记]] - 关联说明

### 掌握程度
half → done
```

使用 `obsidian append` 追加内容（若笔记原只有标题，直接写入；有内容则追加）。

### 5. 更新状态与移动（Agent 自主判定与执行）

不询问用户，根据对话质量自动判定状态：

```bash
# 自动判定状态并更新属性
obsidian vault="iLearn" property:set name="status" value="done" file="<笔记名>"

# 若 done → 匹配领域文件夹并移动
obsidian vault="iLearn" folders
# 基于笔记标题语义匹配现有文件夹（6/10 宽容度）
# 有匹配 → obsidian vault="iLearn" move file="<笔记名>" to="<匹配文件夹>"
# 无匹配 → 创建新文件夹，obsidian vault="iLearn" create path="<新文件夹>/temp" 然后移动
```

**half 状态不移出 waitTolearn**，仅 done 状态触发移动。

### 6. 知识关系维护

```bash
# 扫描 vault 中其他笔记
obsidian vault="iLearn" files

# 搜索语义相关的笔记
obsidian vault="iLearn" search query="<相关关键词>"

# 双向建立 wikilink: 在双方笔记追加 [[对方笔记]]
obsidian vault="iLearn" append file="<笔记A>" content="\n- [[笔记B]]"
obsidian vault="iLearn" append file="<笔记B>" content="\n- [[笔记A]]"
```

## Obsidian CLI 操作参考

依赖 `obsidian-cli` skill。所有操作需使用 `vault="iLearn"` 参数。

| 操作 | 命令 |
|------|------|
| 列出文件夹内容 | `obsidian vault="iLearn" files folder="waitTolearn"` |
| 读取笔记 | `obsidian vault="iLearn" read file="name"` |
| 追加内容 | `obsidian vault="iLearn" append file="name" content="..."` |
| 设置属性 | `obsidian vault="iLearn" property:set name="status" value="done" file="name"` |
| 读取属性 | `obsidian vault="iLearn" property:read name="status" file="name"` |
| 移动文件 | `obsidian vault="iLearn" move file="name" to="folder"` |
| 列出文件夹 | `obsidian vault="iLearn" folders` |
| 搜索笔记 | `obsidian vault="iLearn" search query="term"` |
| 列出所有文件 | `obsidian vault="iLearn" files` |
| 创建文件 | `obsidian vault="iLearn" create path="folder/name.md" content="..."` |

## 前置条件

- Obsidian 必须正在运行
- `C:\Users\Homan\iLearn` vault 必须存在
- vault 中存在 `waitTolearn` 文件夹

## Edge Cases

- **waitTolearn 为空**: 告知用户，不执行操作
- **指定笔记不存在**: 告知用户笔记名和可用列表
- **Obsidian 未运行**: 提示用户先打开 Obsidian
- **笔记已 done**: 告知用户该笔记已完成学习，位于领域文件夹
```

- [ ] **Step 3: 验证 skill 文件格式**

确认 YAML frontmatter 格式正确，name 符合命名规范，description 以 "Use when" 开头。

---

### Task 3: 验证 Skill (GREEN Phase 确认)

**目标：** 用测试场景验证 skill 是否完整覆盖所有需求。

- [ ] **Step 1: 对照测试场景逐项检查**

对照 Task 1 的 5 个测试场景，检查 SKILL.md 是否覆盖：
- [ ] 场景 1: `/ttl` 随机选择 + half 优先
- [ ] 场景 2: `/ttl 知识点` 指定学习
- [ ] 场景 3: done 移动笔记 + 创建文件夹
- [ ] 场景 4: 空 waitTolearn 处理
- [ ] 场景 5: 知识关系 wikilink 维护

- [ ] **Step 2: 交叉引用检查**

确认 SKILL.md 中引用的所有 Obsidian CLI 命令在 `obsidian help` 输出中存在且参数正确。

---

### Task 4: 重构与加固 (REFACTOR Phase)

**目标：** 修补漏洞，确保 skill 对各种边缘情况有明确指导。

- [ ] **Step 1: 识别潜在的 rationalization 漏洞**

| 潜在漏洞 | 加固措施 |
|----------|----------|
| "对话太短，不需要完整总结" | SKILL.md 没有明确要求每次对话都总结 |
| "知识点间没有明显关联，跳过 wikilink" | SKILL.md 没有明确关系维护是必须的 |
| "用户说理解了但不确定，设为 half" | 状态询问逻辑需要明确 |

检查 SKILL.md 是否已覆盖以上情况，必要时补充。

- [ ] **Step 2: 添加 Red Flags 清单**

在 SKILL.md 末尾追加：
```markdown
## Red Flags — 必须执行，不得跳过

- 每次 /ttl 对话结束后**必须**追加学习总结
- 每次学习后**必须**检查并建立知识关系 wikilink
- 状态更新**必须**在追加总结之后执行，由 agent 自动判定（不询问用户）
- done 状态**必须**触发移动操作
- 用户只参与学习对话，**不得**让用户操作文件或手动设置状态
- 使用 vault="iLearn" 参数，**不得**省略
```

- [ ] **Step 3: 最终格式审查**

- name 字段仅含字母数字和连字符
- description 以 "Use when" 开头，第三人称
- 无 TBD/TODO 占位符
- 所有命令示例可直接执行

---

### Task 5: 部署

- [ ] **Step 1: 确认文件位置**

```bash
Test-Path "C:\Users\Homan\.opencode\skills\time-to-learn\SKILL.md"
```

- [ ] **Step 2: 提交项目文档（可选）**

```bash
git -C "D:\works\time-to-learn" init
git -C "D:\works\time-to-learn" add design.md plan.md test-scenarios.md baseline-results.md
git -C "D:\works\time-to-learn" commit -m "feat: time-to-learn skill design and plan"
```
```

(no changes needed - plan complete)
