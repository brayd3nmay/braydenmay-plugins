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
