# braydenmay-plugins

A personal, local [Claude Code plugin marketplace](https://code.claude.com/docs/en/plugin-marketplaces).

Each skill is its own plugin, so they can be installed independently.

## Layout

```
.
├── .claude-plugin/
│   └── marketplace.json           # registers each plugin
├── plugins/
│   └── <plugin-name>/
│       ├── .claude-plugin/
│       │   └── plugin.json        # plugin manifest
│       └── skills/
│           └── <skill-name>/
│               └── SKILL.md
└── README.md
```

Add a new skill by creating a new plugin folder under `plugins/`, adding it to `marketplace.json`, then running `/plugin reload`.

## Install

From inside Claude Code:

```
/plugin marketplace add /Users/braydenmay/repos/braydenmay-plugins
/plugin install readme@braydenmay-plugins
```

Install only the plugins you want; each is independent.

## Plugins

| Plugin | Description |
| --- | --- |
| [readme](plugins/readme/skills/writing-readme/SKILL.md) | Use when writing or generating a README from scratch for a software project. |
| [kryptonite](plugins/kryptonite/skills/using-kryptonite/SKILL.md) | Skills for Claude Code agent teams: brainstorming, planning with 8 parallel validators, TDD, debugging, and long-lived agent-team coordination via `coordinating-agent-teams`. |
| [refine-jakes-resume](plugins/refine-jakes-resume/skills/refine-jakes-resume/SKILL.md) | Use when working on a CS-internship resume based on the Jake's Resume LaTeX template. |
| [council](plugins/council/README.md) | Convene a council of AI coding agents — each writes its candid perspective on a project or spec into `COUNCIL.md`. Works across Claude Code, Cowork, Codex, and Antigravity. |
