# common-skill

Common Claude Code skills, packaged as distributable plugins.

This repository is a Claude Code **marketplace** that ships two plugins of its own and references four external ones.

## Plugins

### common-skill

- **skill-check** — Diagnoses a skill folder against Agent Skills best practices in two phases: a deterministic structural check (`scripts/check_skill.py`) plus a qualitative review across six dimensions.
- **eli13** — Explains a topic like you're 13 years old, using a simple HTML artifact with big pictures and few words, with the final output translated into Japanese.

### ja-refine

- **ja-refine** — Rewrites LLM-generated Japanese into Japanese that is easy to understand, without changing its meaning. It holds no writing rules itself; it orchestrates four external skills in a fixed order and settles conflicts between their rules.

It has two modes, chosen from your prompt:

| Mode | Use for | Stages |
|---|---|---|
| `doc` (default) | Specialised documents and official internal documents: specs, reports, minutes, proposals, READMEs | japanese-tech-writing → natural-japanese → yomiyasu |
| `blog` | Blog posts and anything written for an unspecified audience | japanese-tech-writing → cognitive-rhythm-writing → natural-japanese → yomiyasu |

When the mode cannot be determined, it runs in `doc` mode. Given a file, it overwrites that file with the result; given pasted text, it replies with the result.

## External skills used by ja-refine

`ja-refine` depends on the following skills. **None of them is copied into this repository.** The marketplace manifest only points at each upstream source, pinned to a commit, and Claude Code fetches them from there at install time.

| Plugin | Upstream | Author | License |
|---|---|---|---|
| yomiyasu | https://github.com/nanaism/yomiyasu (v3.1.0) | nanaism | MIT |
| natural-japanese | https://github.com/coji/natural-japanese (v2.0.0) | coji | MIT |
| japanese-tech-writing | https://gist.github.com/k16shikano/fd287c3133457c4fd8f5601d34aa817d | k16shikano | Unlicense |
| cognitive-rhythm-writing | https://gist.github.com/k16shikano/eb2929f13ed19c97188393d297be8432 | k16shikano | Unlicense |

The two gists hold a single `SKILL.md` each and are not plugins on their own. The marketplace entry registers each gist's git URL as a plugin source so that Claude Code recognises it as a skill.

To move to a newer upstream version, update the `sha` (and `ref`) of the entry in `.claude-plugin/marketplace.json`.

natural-japanese runs its lint scripts with [`uv`](https://docs.astral.sh/uv/), and yomiyasu runs its checks with Python 3. Without them, each skill falls back to a manual check.

## Install

```
/plugin marketplace add hirokita117/common-skill
/plugin install common-skill@common-skill
/plugin install ja-refine@common-skill
```

Installing `ja-refine` also installs the four external plugins above as dependencies.

Once installed, invoke the skills with:

```
/common-skill:skill-check [path-to-skill-folder]
/common-skill:eli13 [topic]
/ja-refine:ja-refine [doc|blog] [file-or-request]
```

## Layout

```
common-skill/
├── .claude-plugin/
│   ├── plugin.json        # manifest of the common-skill plugin
│   └── marketplace.json   # marketplace definition (2 local + 4 external plugins)
├── skills/                # skills of the common-skill plugin
│   ├── skill-check/
│   │   ├── SKILL.md
│   │   └── scripts/
│   │       ├── check_skill.py
│   │       └── list_skills.py
│   └── eli13/
│       └── SKILL.md
└── plugins/
    └── ja-refine/         # the ja-refine plugin
        ├── .claude-plugin/
        │   └── plugin.json   # declares the 4 dependencies
        └── skills/
            └── ja-refine/
                └── SKILL.md
```

The `check_skill.py` script requires PyYAML (`pip install pyyaml`).
