# Trends Skills

Agent skills for [Trends](https://trends.fun). Install them into your AI coding agent to get guided workflows for operating Trends products — with built-in safety rails for write operations and private key handling.

## Available Skills

| Skill | Description |
| --- | --- |
| [trends](skills/trends/) | Trends 操作助手 — 发币、买卖、查仓位、领收益、项目币/IAO 和报错处理 |

## Installation

```bash
npx skills add trends-fun/skills
```

This installs all skills from this repository into your agent. The agent will then automatically activate the relevant skill based on user intent.

## Skill Structure

```plain
skills/
  <skill-name>/
    SKILL.md              # Skill definition and agent behavior rules
    references/           # Supporting docs (recipes, setup guides, error playbooks, etc.)
    evals/
      evals.json          # Evaluation prompts and expected behaviors
```

Each skill is a self-contained directory under `skills/`. The agent discovers skills through the `SKILL.md` entry point in each directory.

## License

[MIT](LICENSE)
