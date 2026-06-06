# agent-skills

Personal collection of agent skills following the [Agent Skills specification](https://agentskills.io/specification).

Skills live under `skills/<name>/SKILL.md` and are managed with the GitHub CLI `gh skill` command (preview).

## Install

```bash
gh skill install kindaidai/agent-skills <skill> --agent claude-code --scope user
```

## Update

```bash
gh skill update --all
```

## Skills

| Skill | Description |
|-------|-------------|
| baseline-ui | Enforces an opinionated UI baseline to prevent AI-generated interface slop. |
| browser-use | Browser automation via the `browser-use` CLI. |
| frontend-design | Create distinctive, production-grade frontend interfaces. |
| web-design-guidelines | Review UI code for Web Interface Guidelines compliance. |
| tmux-relay | Relay messages to other Claude Code instances via tmux panes. |
