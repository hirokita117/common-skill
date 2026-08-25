# common-skill

Common Claude Code skills, packaged as a distributable plugin.

## Included skills

- **skill-check** — Diagnoses a skill folder against Agent Skills best practices in two phases: a deterministic structural check (`scripts/check_skill.py`) plus a qualitative review across six dimensions.
- **eli13** — Explains a topic like you're 13 years old, using a simple HTML artifact with big pictures and few words, with the final output translated into Japanese.

## Install

This repository is both a Claude Code **plugin** and its own **marketplace**.

```
/plugin marketplace add hirokita117/common-skill
/plugin install common-skill@common-skill
```

Once installed, invoke the skills with:

```
/common-skill:skill-check [path-to-skill-folder]
/common-skill:eli13 [topic]
```

## Layout

```
common-skill/
├── .claude-plugin/
│   ├── plugin.json        # plugin manifest
│   └── marketplace.json   # marketplace definition (source: this repo)
└── skills/
    ├── skill-check/
    │   ├── SKILL.md
    │   └── scripts/
    │       ├── check_skill.py
    │       └── list_skills.py
    └── eli13/
        └── SKILL.md
```

The `check_skill.py` script requires PyYAML (`pip install pyyaml`).
