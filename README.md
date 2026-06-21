# time-to-learn

OpenCode Skill。通过交互式对话学习未知概念，自动管理 Obsidian 知识库中的学习进度和知识图谱。

## 快速开始

```bash
# /ttl         → 随机选一个待学概念开始学习
# /ttl Docker  → 指定学习 Docker
```

对话过程你只负责提问和理解，**agent 自动完成**总结撰写、状态更新、文件分类、知识关联。

## Agent 一键安装

将以下内容粘贴到 OpenCode 中，agent 将自动完成安装：

```
请帮我安装 time-to-learn skill：
1. 从 https://github.com/Newoahil/opencode-time-to-learn 下载 SKILL.md
2. 放到 ~/.config/opencode/skills/time-to-learn/SKILL.md
3. 在 ~/.config/opencode/opencode.json 中注册 /ttl 命令：
   "command": { "ttl": { "template": "Use the time-to-learn skill. $ARGUMENTS", "description": "交互式学习" } }
4. 修改 SKILL.md 顶部的配置变量（VAULT_PATH / VAULT_NAME / LEARN_FOLDER）为你的实际路径
5. 在 Obsidian vault 中创建 LEARN_FOLDER 对应的文件夹
```

安装完成后重启 OpenCode，输入 `/ttl` 开始学习。

## 功能

- **冷启动**：自动拉起 Obsidian，无需手动打开
- **智能选择**：优先选学了一半的概念（half 状态），巩固薄弱点
- **交互式学习**：AI 讲解 → 你提问 → AI 纠正 → 自动总结
- **全自动整理**：对话结束自动将理解过程总结为结构化的知识卡片
- **状态追踪**：notyet → half → done，掌握程度一目了然
- **知识图谱**：自动识别关联知识点，建立双向 wikilink，Obsidian 原生图谱渲染

## 配置

安装后需修改 `SKILL.md` 开头的配置变量：

| 变量 | 说明 | 示例 |
|------|------|------|
| `VAULT_PATH` | Vault 本地路径 | `~/Documents/Notes` |
| `VAULT_NAME` | Vault 名称（CLI vault= 参数） | `Notes` |
| `LEARN_FOLDER` | 待学文件夹名 | `waitTolearn` |

## 安装（手动）

### 1. 放置 Skill 文件

```bash
# Windows
mkdir %USERPROFILE%\.config\opencode\skills\time-to-learn
copy SKILL.md %USERPROFILE%\.config\opencode\skills\time-to-learn\

# macOS / Linux
mkdir -p ~/.config/opencode/skills/time-to-learn
cp SKILL.md ~/.config/opencode/skills/time-to-learn/
```

### 2. 注册命令

在 `~/.config/opencode/opencode.json` 中添加：

```json
{
  "command": {
    "ttl": {
      "template": "Use the time-to-learn skill. $ARGUMENTS",
      "description": "交互式学习"
    }
  }
}
```

### 3. 配置变量

编辑 `SKILL.md` 顶部的 VAULT_PATH / VAULT_NAME / LEARN_FOLDER 为你的实际值。

### 4. 准备 Vault

在 Obsidian vault 中创建 `LEARN_FOLDER` 对应的文件夹（如 `waitTolearn`），放入你想学习的概念笔记（每个概念一个文件，标题即概念名）。

### 5. 重启 OpenCode

配置在重启后生效。

## 状态模型

| 状态 | 含义 | 位置 | 选择优先级 |
|------|------|------|-----------|
| `notyet` | 尚未学习 | LEARN_FOLDER | 低 |
| `half` | 部分掌握 | LEARN_FOLDER | 最高 |
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
## 学习记录 - YYYY-MM-DD

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

## License

MIT
