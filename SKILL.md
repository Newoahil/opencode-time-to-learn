---
name: time-to-learn
description: Use when user types /ttl command, /ttl <topic>, or expresses intent to learn a concept from their Obsidian vault — manages interactive learning dialogue, note status (notyet/half/done), file organization, and knowledge graph via wikilinks
---

# time-to-learn

## 概述

交互式学习 skill。用户只参与学习对话，agent 全自动完成：笔记选择、内容讲解、对话纠正、学习总结撰写、状态判定、文件移动、知识关系 wikilink 维护。

**所有输出必须使用简体中文。** 讲解、对话、总结、报告全部为中文。技术术语可保留英文原文，但解释必须用中文。

## 配置

使用前请根据你的实际环境修改以下变量。Agent 在对话中应使用这些配置：

| 配置项 | 说明 | 示例 |
|--------|------|------|
| `VAULT_PATH` | Obsidian vault 的本地路径 | `C:\Users\<name>\iLearn` 或 `~/Documents/iLearn` |
| `VAULT_NAME` | Obsidian vault 名称（CLI 中 `vault=` 参数） | `iLearn` |
| `LEARN_FOLDER` | 待学习笔记所在文件夹 | `waitTolearn` |

以下文档中以 `VAULT_PATH`、`VAULT_NAME`、`LEARN_FOLDER` 指代这些可配置值。

## 触发

| 指令 | 行为 |
|------|------|
| `/ttl` | 从 `LEARN_FOLDER` 随机选一篇笔记，**优先选 `half` 状态** |
| `/ttl <知识点>` | 指定学习名为 `<知识点>` 的笔记 |

## 状态模型

| 状态 | 含义 | 位置 | 选择优先级 |
|------|------|------|-----------|
| `notyet` | 尚未学习 | `LEARN_FOLDER` | 低 |
| `half` | 部分掌握 | `LEARN_FOLDER` | 最高 |
| `done` | 完全掌握 | 领域文件夹 | 不再选中 |

状态通过 Obsidian 原生 frontmatter 属性 `status` 维护。

## Vault 信息

- Vault 路径：由 `VAULT_PATH` 配置
- 所有 Obsidian CLI 命令必须带 `vault="VAULT_NAME"`
- 依赖 `obsidian-cli` skill

---

## 工作流

### 阶段 0：冷启动 Obsidian（自动）

执行任何 Obsidian CLI 操作前，先确认 Obsidian 是否在运行：

```bash
obsidian vault="VAULT_NAME" files folder="LEARN_FOLDER"
```

若返回 `"The CLI is unable to find Obsidian"`，自动启动：

```bash
# Windows: URI 协议启动
Start-Process "obsidian://open?vault=VAULT_NAME"

# macOS: open "obsidian://open?vault=VAULT_NAME"
# Linux: xdg-open "obsidian://open?vault=VAULT_NAME"

# 轮询等待 Obsidian 就绪（最长 15 秒）
while ($running -ne $true) {
  Start-Sleep -Seconds 2
  $result = obsidian vault="VAULT_NAME" files folder="LEARN_FOLDER" 2>&1
  if ($result -notmatch "unable to find") { break }
}
```

此阶段对用户透明，无需用户手动打开 Obsidian。

### 阶段 1：选中笔记

```
/ttl 无参数：
  obsidian vault="VAULT_NAME" files folder="LEARN_FOLDER"
  → 逐个读取 status 属性 → 优先选 half，half 有多个则随机选一个
  → half 无则从 notyet 中随机选
  → 若 LEARN_FOLDER 为空：告知用户"LEARN_FOLDER 里没有待学习笔记"

/ttl <知识点>：
  obsidian vault="VAULT_NAME" read file="<知识点>"
  → 文件存在 → 继续
  → 文件不存在 → 告知用户并列出 LEARN_FOLDER 中可用笔记
```

### 阶段 2：读取与讲解

```bash
obsidian vault="VAULT_NAME" read file="<笔记名>"
obsidian vault="VAULT_NAME" property:read name="status" file="<笔记名>"
```

读取后进行基本讲解（2-4 句核心要点），不依赖笔记内容（笔记可能最初只有标题），基于自身知识讲解。**讲解全程使用中文。**

### 阶段 3：对话学习

- 用户提问或表达自己的理解
- 纠正错误、补充细节、举例说明
- **持续跟踪**：正确复述次数、实质问题是否枯竭、对话轮次
- **全程使用简体中文对话**

#### 对话结束判断

```
立即结束（满足任一）：
  - 用户明确信号："懂了""明白了""差不多了""总结吧"
  - 用户连续 2 次正确复述/自我纠正 → 理解程度高
  - 连续 2 轮无实质新问题 → 对话自然枯竭

兜底问询：
  - 对话超过 6 轮无结束信号 → 主动问"差不多了吗，需要我总结一下？"
  - 用户表述含糊（"还行""大概懂了"）→ 追问确认问题再决定
```

#### 状态自动判定

| 对话表现 | 判定 |
|----------|------|
| 表达清晰、自我纠正准确、无重大误解 | `done` |
| 仍有模糊/不确定、某个要点未完全澄清 | `half` |

**不询问用户状态**，agent 根据对话质量自主判定。

### 阶段 4：撰写学习总结

**核心原则：笔记是独立的可回看学习资料。** 用户忘记后仅靠笔记就能重新掌握。转述整理，不是对话转录。**风格必须口语化**，用"我以为X→其实Y"、"之前理解错的地方"这种自己写笔记的表达方式，避免学术术语。

```bash
obsidian vault="VAULT_NAME" append file="<笔记名>" content="
## 学到了什么 - YYYY-MM-DD

### 一句话说清楚
<用最简单的话解释这个知识点，像在跟朋友聊天>

### 我是怎么理解的
1. 一开始我以为...
2. 后来发现...
3. 现在我懂了...

### 之前理解错的地方

我以为 <旧理解>
→ 其实 <正确理解>

### 在哪能用上
- <你在对话里提到的真实场景>
- <我补充的例子>

### 相关的
- [[笔记A]] - <一两句话说明关联>
- [[笔记B]] - <一两句话说明关联>

### 掌握程度
half
"
```

### 阶段 5：更新状态与移动

```bash
# 根据阶段 3 的自动判定更新状态
obsidian vault="VAULT_NAME" property:set name="status" value="done" file="<笔记名>"
```

**仅 `done` 状态触发移动：**

```bash
# 列出 vault 顶层文件夹，排除 LEARN_FOLDER 和系统目录
obsidian vault="VAULT_NAME" folders

# 基于笔记标题/内容语义匹配现有文件夹（宽容度 6/10）
# 有匹配 → obsidian vault="VAULT_NAME" move file="<笔记名>" to="<匹配文件夹>"
# 无匹配 → 创建新文件夹并移动
```

`half` 状态**保留在 LEARN_FOLDER**，下次 `/ttl` 优先选中。

### 阶段 6：知识关系维护

```bash
# 扫描 vault 中其他笔记
obsidian vault="VAULT_NAME" files

# 搜索语义相关的笔记
obsidian vault="VAULT_NAME" search query="<关键词>"

# 双向建立 wikilink
obsidian vault="VAULT_NAME" append file="<笔记A>" content="\n- [[笔记B]]"
obsidian vault="VAULT_NAME" append file="<笔记B>" content="\n- [[笔记A]]"
```

Obsidian 原生图谱自动渲染关系网。

### 阶段 7：完成报告

```
本次学习总结已记录 ✓  状态: done
📁 已移至: <领域文件夹>/
🔗 关联: [[相关笔记A]] | [[相关笔记B]]
```

---

## CLI 快速参考

所有命令必须带 `vault="VAULT_NAME"`。

| 操作 | 命令 |
|------|------|
| 列出文件夹内容 | `obsidian vault="VAULT_NAME" files folder="LEARN_FOLDER"` |
| 读取笔记 | `obsidian vault="VAULT_NAME" read file="name"` |
| 追加内容 | `obsidian vault="VAULT_NAME" append file="name" content="..."` |
| 设置属性 | `obsidian vault="VAULT_NAME" property:set name="status" value="done" file="name"` |
| 读取属性 | `obsidian vault="VAULT_NAME" property:read name="status" file="name"` |
| 移动文件 | `obsidian vault="VAULT_NAME" move file="name" to="folder"` |
| 列出文件夹 | `obsidian vault="VAULT_NAME" folders` |
| 搜索笔记 | `obsidian vault="VAULT_NAME" search query="term"` |
| 列出所有文件 | `obsidian vault="VAULT_NAME" files` |

---

## 前置条件

- Obsidian 已安装（agent 会自动启动，无需手动打开）
- Vault 存在且路径由 `VAULT_PATH` 指定
- vault 中存在由 `LEARN_FOLDER` 指定的文件夹

## Edge Cases

| 情况 | 处理 |
|------|------|
| LEARN_FOLDER 为空 | 告知用户无待学习笔记 |
| 指定笔记不存在 | 告知并列出 LEARN_FOLDER 中可用笔记 |
| Obsidian 未运行 | 自动通过 `obsidian://open?vault=VAULT_NAME` URI 启动，轮询等待就绪 |
| Obsidian 自动启动失败 | 告知用户手动打开 Obsidian |
| 笔记已 done（在领域文件夹） | 告知已完成学习，位于 `folder/` |
| 用户中途中断对话 | 不追加总结，不更新状态 |

## Red Flags

- 每次 `/ttl` 对话结束后**必须**追加学习总结（即使对话很短）
- 每次学习后**必须**检查并建立知识关系 wikilink
- 状态更新**必须**由 agent 自主判定（不询问用户），在总结之后执行
- done 状态**必须**触发移动到领域文件夹
- **用户只参与学习对话，不得让用户操作文件或手动设置状态**
- 所有 Obsidian CLI 命令**必须**带 `vault="VAULT_NAME"`
- 总结**必须**是转述性知识卡片，不是对话记录
- **所有输出、讲解、对话、总结必须使用简体中文**（技术术语可保留英文，但解释用中文）
