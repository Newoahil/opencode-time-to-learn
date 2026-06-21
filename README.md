# time-to-learn

OpenCode Skill。通过交互式对话学习未知概念，自动管理 Obsidian iLearn 知识库中的学习进度和知识图谱。

## 快速开始

```bash
# /ttl         → 随机选一个待学概念开始学习
# /ttl Docker  → 指定学习 Docker
```

对话过程你只负责提问和理解，**agent 自动完成**总结撰写、状态更新、文件分类、知识关联。

## 功能

- **冷启动**：自动拉起 Obsidian，无需手动打开
- **智能选择**：优先选学了一半的概念（half 状态），巩固薄弱点
- **交互式学习**：AI 讲解 → 你提问 → AI 纠正 → 自动总结
- **全自动整理**：对话结束自动将理解过程总结为结构化的知识卡片
- **状态追踪**：notyet → half → done，掌握程度一目了然
- **知识图谱**：自动识别关联知识点，建立双向 wikilink，Obsidian 原生图谱渲染

## 安装

### 1. 放置 Skill 文件

将 `SKILL.md` 放入 OpenCode 的全局 skills 目录：

```bash
# Windows
mkdir -p %USERPROFILE%\.config\opencode\skills\time-to-learn
copy SKILL.md %USERPROFILE%\.config\opencode\skills\time-to-learn\SKILL.md

# macOS / Linux
mkdir -p ~/.config/opencode/skills/time-to-learn
cp SKILL.md ~/.config/opencode/skills/time-to-learn/SKILL.md
```

### 2. 注册命令

在 `~/.config/opencode/opencode.json` 中添加：

```json
{
  "command": {
    "ttl": {
      "template": "Use the time-to-learn skill. $ARGUMENTS",
      "description": "交互式学习 - 从 iLearn 知识库开始学习"
    }
  }
}
```

### 3. 准备 Obsidian Vault

确保 Obsidian 已安装，且存在 vault `C:\Users\<用户名>\iLearn`，其中包含 `waitTolearn` 文件夹。

### 4. 重启 OpenCode

配置在重启后生效。

## 状态模型

| 状态 | 含义 | 位置 | 选择优先级 |
|------|------|------|-----------|
| `notyet` | 尚未学习 | waitTolearn | 低 |
| `half` | 部分掌握 | waitTolearn | 最高 |
| `done` | 完全掌握 | 领域文件夹 | 不再选中 |

## 工作流

```
阶段 0  冷启动 Obsidian（自动）
阶段 1  选中笔记（优先 half）
阶段 2  读取与讲解
阶段 3  对话学习（全程跟踪理解程度）
阶段 4  撰写学习总结（转述性知识卡片）
阶段 5  更新状态与移动（done → 领域文件夹）
阶段 6  知识关系维护（双向 wikilink）
阶段 7  完成报告
```

## 对话结束判断

| 触发条件 | 行为 |
|----------|------|
| 用户说"懂了""总结吧"等 | 立即进入总结 |
| 连续 2 次正确复述 | 自动判定理解程度高，总结 |
| 连续 2 轮无新问题 | 对话自然枯竭，总结 |
| 对话超过 6 轮 | 主动询问"差不多了吗？" |
| 用户表述含糊（"还行"） | 追问确认后再决定 |

## 前置依赖

- [OpenCode](https://opencode.ai)
- [Obsidian](https://obsidian.md)（agent 会自动启动）
- Obsidian CLI（Obsidian 内置）
- 依赖 skill：`obsidian-cli`

## 学习总结格式

每次学习后 agent 自动在笔记末尾追加结构化的知识卡片：

```markdown
## 学习记录 - 2026-06-20

### 核心概念
<转述段落，独立可回看>

### 关键要点
- <本质理解>
- <易混淆概念区分>
- <使用场景>

### 常见误区
| 误区 | 正确理解 |
|------|---------|

### 实践场景
- <真实案例>

### 关联知识
- [[相关笔记]] - 关联说明

### 掌握程度
half → done
```

## 项目结构

```
time-to-learn/
  SKILL.md        # Skill 定义文件
  README.md       # 本文档（中文）
  README.en.md    # English version
  design.md       # 设计文档
  plan.md         # 实施计划
```

## License

MIT
