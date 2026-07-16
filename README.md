# common-skill

Common Claude Code skills, packaged as a distributable plugin.

## Included skills

- **skill-check** — Diagnoses a skill folder against Agent Skills best practices in two phases: a deterministic structural check (`scripts/check_skill.py`) plus a qualitative review across six dimensions.

## Install

This repository is both a Claude Code **plugin** and its own **marketplace**.

```
/plugin marketplace add hirokita117/common-skill
/plugin install skill-check@common-skill
```

Once installed, invoke the skill with:

```
/skill-check:skill-check [path-to-skill-folder]
```

## Layout

```
common-skill/
├── .claude-plugin/
│   ├── plugin.json        # plugin manifest
│   └── marketplace.json   # marketplace definition (source: this repo)
└── skills/
    └── skill-check/
        ├── SKILL.md
        └── scripts/
            ├── check_skill.py
            └── list_skills.py
```

The `check_skill.py` script requires PyYAML (`pip install pyyaml`).
