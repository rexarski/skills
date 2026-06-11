# skills

Personal agent skills, installable via the
[`skills` CLI](https://github.com/vercel-labs/skills). This repo exists so
*every* skill on my machines is lockfile-tracked — the `skills` CLI prunes
anything in `~/.agents/skills/` that isn't in `.skill-lock.json`, so
locally-authored skills must live in a repo it knows about.

## Layout

```
skills/
└── <skill-name>/
    └── SKILL.md      # YAML frontmatter (name, description) + instructions
```

One directory per skill under `skills/`. The `skills` CLI discovers them by
this layout (same convention as e.g. `kepano/obsidian-skills`).

## Install

```fish
skills add rexarski/skills -g -y
```

This clones the repo, copies each skill into `~/.agents/skills/<name>/`,
records it in `~/.agents/.skill-lock.json`, and symlinks it into each
selected agent's skill dir (`~/.claude/skills/`, etc.).

## Pairing with rexarski/dotfiles

[`rexarski/dotfiles`](https://github.com/rexarski/dotfiles) (chezmoi source)
tracks `~/.agents/.skill-lock.json`, which pins every installed skill —
including the ones from this repo — by source and folder hash. New machine:

```fish
chezmoi init --apply rexarski/dotfiles   # writes .skill-lock.json + helpers
~/.agents/update-skills.fish             # reinstalls everything in the lockfile
```

No special-casing: skills from this repo restore exactly like third-party
packs.

## Authoring a new skill

```fish
cd ~/Developer/skills
mkdir skills/<name>
$EDITOR skills/<name>/SKILL.md           # frontmatter: name + description
git add -A && git commit -m "feat(<name>): add skill" && git push
skills add rexarski/skills -g -y         # pick up the new skill locally
chezmoi re-add ~/.agents/.skill-lock.json   # then commit in dotfiles
```

Editing an existing skill is the same flow — commit/push here, then
re-run `skills add` (or `~/.agents/update-skills.fish <name>`) to refresh
the installed copy.
