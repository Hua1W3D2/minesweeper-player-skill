# Minesweeper Player Skill

一个用于浏览器扫雷游戏的 Codex skill，采用严格的“先基于可见信息推理”的工作流程。

该 skill 主要面向 `https://www.minesweeper.cn/` 设计，但其中的可见信息求解思路也适用于其他浏览器扫雷游戏。

## 功能

- 检查当前环境是否有可用的浏览器控制 MCP 或工具。
- 在需要时打开 `https://www.minesweeper.cn/`。
- 开始时只基于棋盘上的可见信息进行游戏：
  - 已打开格子的数字
  - 已放置的旗帜
  - 仍被覆盖的格子
  - 剩余雷数计数器
- 应用标准扫雷推理：
  - 直接数字规则
  - 子集规则
  - 精确局部约束求解
  - 全局剩余雷数约束
- 当可见棋盘上不存在可证明的下一步时，停止并报告当前状态。
- 只有在用户允许回退方案时，才会在上述推理全部耗尽后读取隐藏页面状态，完成必须依赖猜测的格子。

## 安装

将 skill 文件夹复制到本地 skills 目录。

Windows PowerShell：

```powershell
New-Item -ItemType Directory -Force "$env:USERPROFILE\.codex\skills" | Out-Null
Copy-Item -Recurse -Force .\minesweeper-player "$env:USERPROFILE\.codex\skills"
```

macOS/Linux：

```bash
mkdir -p ~/.agents/skills
cp -R ./minesweeper-player ~/.agents/skills/minesweeper-player
```

安装后重启 Codex，使 skill 列表刷新。

## 使用

示例提示词：

```text
Use $minesweeper-player to play minesweeper.cn for me.
```

```text
Help me play this Minesweeper page. Use only visible information first, then reveal the answer only if the board becomes a forced guess.
```

```text
Play Minesweeper normally. Do not read hidden mines until all visible deductions are exhausted.
```

## 项目结构

```text
minesweeper-player-skill/
  README.md
  LICENSE
  minesweeper-player/
    SKILL.md
    agents/
      openai.yaml
```

## 说明

该 skill 会刻意区分正常扫雷流程和答案回退流程。如果最后的格子是通过读取隐藏页面状态完成的，代理必须明确说明这一点，不能把这次游戏描述为完全基于规则推理完成。
