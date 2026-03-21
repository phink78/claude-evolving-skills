# Claude Evolving Skills

[English](README.md)

让 Claude Code 自己改进自己的配置。不用手动追 GitHub 爆款 repo，这些 skill 会自动扫描、评估、辩论、采纳改进。

> **AI agent 请看**: [AGENTS.md](AGENTS.md)（英文），专门为 agent 消费设计。
>
> 灵感来自 Karpathy 的 [Software 3.0](https://www.latent.space/p/s3) 演讲。另见 [agents.md spec](https://agents.md/)。

## 这是什么

一组可复用的 Claude Code skill，放进 `~/.claude/skills/` 就能用。装上之后你的 agent 会开始自动改进配置：

- **扫描** GitHub、HN、Reddit 和官方文档，找新工具和技术
- **评估** 发现的内容，用评分标准 + 多模型辩论（Claude + Gemini + Codex）
- **反思** 过去的对话，找失败模式和效率问题
- **采纳** 安全的改动自动生效，有风险的标记给你审核
- **追踪** 每次改动记录在树形历史中，随时可以回溯

方法论参考了近期 self-evolving agent 研究：[AFlow](https://arxiv.org/abs/2410.10762)（ICLR 2025）、[AgentEvolver](https://arxiv.org/abs/2511.10395)、[Live-SWE-agent](https://arxiv.org/abs/2511.13646)、[EvoAgentX](https://arxiv.org/abs/2507.03616)。

## Skills

| Skill | Agent 数 | 功能 |
|-------|---------|------|
| [agentic-radar](skills/agentic-radar/SKILL.md) | 5 | 扫描 GitHub trending、HN、Reddit，发现 agentic 工具，更新本地工具库，提议 CLAUDE.md 改进。 |
| [vendor-docs-radar](skills/vendor-docs-radar/SKILL.md) | 3 | 监控 Anthropic、Google、OpenAI 官方博客/文档的新功能和 breaking changes。 |
| [reflect-and-learn](skills/reflect-and-learn/SKILL.md) | 6 | 核心自我改进循环。回顾历史对话，双通道评分，多模型辩论，记忆整理，经验剪枝，工具共进化检测，进化树追踪。 |
| [gemini-agent](skills/gemini-agent/SKILL.md) | — | 把任务委托给 Gemini CLI。被其他 skill 用于多模型协商。 |
| [codex-agent](skills/codex-agent/SKILL.md) | — | 把任务委托给 Codex CLI。被其他 skill 用于多模型协商。 |

## 快速开始

### 方案 A：让你的 agent 自己消化这个 repo（推荐）

最好的用法是让 AI agent 读这个 repo，然后自己构建适合你工作流的版本。把 Claude Code 指向这个 repo：

> 读 https://github.com/PalmDr/claude-evolving-skills ——理解这些 skill 的模式，然后给我搭建一套适合我工作流的自进化配置。

你的 agent 会读 SKILL.md 文件，理解其中的模式（并行 agent、多模型辩论、进化追踪），然后创建适合你的 skill。这比直接复制别人的配置更符合这个项目的精神。

### 方案 B：直接克隆安装

```bash
# 1. 复制 skills 到你的 Claude Code 配置目录
git clone https://github.com/PalmDr/claude-evolving-skills.git
cp -r claude-evolving-skills/skills/* ~/.claude/skills/

# 2.（可选）复制自动化脚本
mkdir -p ~/.claude/scripts
cp claude-evolving-skills/scripts/* ~/.claude/scripts/
chmod +x ~/.claude/scripts/*.sh

# 3. 初始化 ~/.claude 的 git（如果还不是 git repo）
cd ~/.claude && git init

# 4. 初始化进化追踪
mkdir -p ~/.claude/evolution-history/reflections
touch ~/.claude/evolution-history/evolution-tree.jsonl
touch ~/.claude/evolution-history/scoreboard.jsonl
touch ~/.claude/CHANGELOG.md

# 5.（可选）初始化 agentic-radar 的工具库
mkdir -p ~/.claude/tools-registry/reviews
touch ~/.claude/tools-registry/REGISTRY.md
touch ~/.claude/tools-registry/adoption-log.md

# 6.（可选）初始化报告目录
mkdir -p ~/.claude/agentic-radar-reports
mkdir -p ~/.claude/vendor-docs-reports

# 7. 添加到你的 CLAUDE.md（或者让 skill 自己来）
```

Skill 通过匹配触发短语或斜杠命令激活。

## 前置依赖

| 工具 | 安装方式 | 用途 |
|------|---------|------|
| [Claude Code](https://docs.anthropic.com/en/docs/claude-code) | `npm i -g @anthropic-ai/claude-code` | 所有 skill |
| [Gemini CLI](https://github.com/google-gemini/gemini-cli) | `npm i -g @google/gemini-cli` | gemini-agent、reflect-and-learn |
| [Codex CLI](https://github.com/openai/codex) | `npm i -g @openai/codex` | codex-agent、reflect-and-learn |
| git | 大多数系统自带 | 进化追踪，提交配置变更 |
| [jq](https://jqlang.github.io/jq/) | `brew install jq` / `apt install jq` | reflect-and-learn（解析会话 JSONL） |

API key：`GEMINI_API_KEY`（或 OAuth）和 `OPENAI_API_KEY`。

Gemini 和 Codex 是可选的——reflect-and-learn 没有它们也能跑，只是少了多模型辩论。

**成本：** 每周跑一次多模型辩论大约 $5-10/月的 API 调用费。扫描类 skill 主要用免费的 web search。

## 自动化调度

所有 skill 都可以通过 launchd（macOS）或 cron（Linux）按周调度：

| 日 | Skill | 用途 |
|----|-------|------|
| 周一 | agentic-radar | GitHub + 社区扫描 |
| 周三 | reflect-and-learn | 对话分析 + 自我改进 |
| 周五 | vendor-docs-radar | 官方文档更新 |

### macOS (launchd)

```bash
# 示例 plist——根据你的偏好调整日期和时间
cat > ~/Library/LaunchAgents/com.claude.reflect-and-learn.plist << 'EOF'
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.claude.reflect-and-learn</string>
    <key>ProgramArguments</key>
    <array>
        <string>/bin/bash</string>
        <string>-l</string>
        <string>-c</string>
        <string>~/.claude/scripts/reflect-and-learn.sh</string>
    </array>
    <key>StartCalendarInterval</key>
    <dict>
        <key>Weekday</key><integer>3</integer>
        <key>Hour</key><integer>3</integer>
        <key>Minute</key><integer>0</integer>
    </dict>
    <key>RunAtLoad</key>
    <false/>
</dict>
</plist>
EOF
launchctl load ~/Library/LaunchAgents/com.claude.reflect-and-learn.plist
```

### Linux (cron)

```cron
0 3 * * 1 ~/.claude/scripts/agentic-radar.sh
0 3 * * 3 ~/.claude/scripts/reflect-and-learn.sh
0 3 * * 5 ~/.claude/scripts/vendor-docs-radar.sh
```

## 工作原理

```
┌───────────────┐  ┌─────────────────┐  ┌──────────────────┐
│ agentic-radar │  │ vendor-docs-    │  │ reflect-and-     │
│ (Mon)         │  │ radar (Fri)     │  │ learn (Wed)      │
└───────┬───────┘  └────────┬────────┘  └────────┬─────────┘
        │                   │                    │
        └─────────┬─────────┘                    │
                  ▼                              ▼
        ┌─────────────────┐           ┌──────────────────┐
        │ Proposals       │           │ Session Analysis │
        │ (new tools,     │           │ (failures,       │
        │  config diffs)  │           │  efficiency,     │
        └────────┬────────┘           │  satisfaction)   │
                 │                    └────────┬─────────┘
                 └───────────┬────────────────┘
                             ▼
                  ┌──────────────────────┐
                  │ Multi-Voice Debate   │
                  │ Claude + Gemini +    │
                  │ Codex score & argue  │
                  └──────────┬───────────┘
                             ▼
                  ┌──────────────────────┐
                  │ Adoption             │
                  │ P0/P1 → auto-apply   │
                  │ HIGH-IMPACT → flag   │
                  │ DEFER → log only     │
                  └──────────┬───────────┘
                             ▼
                  ┌──────────────────────┐
                  │ Evolution Tree       │
                  │ (git commit +        │
                  │  tree.jsonl +        │
                  │  scoreboard.jsonl)   │
                  └──────────────────────┘
```

## License

MIT
