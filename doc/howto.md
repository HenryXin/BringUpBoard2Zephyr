# Howto: Install the zephyr-board-bringup skill on a new machine

This guide installs the **Cursor Agent skill** package so you can run Zephyr board bring-up workflows on another machine. It does **not** install Zephyr itself — you still need a west/Zephyr workspace when you run the skill through Phase C and beyond.

## What you are installing

Copy this directory tree intact:

```text
zephyr-board-bringup/
├── SKILL.md
├── reference.md
├── examples.md
└── templates/
    ├── inventory.md
    ├── match-report.md
    └── howto.md
```

Source path in this repo:

```text
.cursor/skills/zephyr-board-bringup/
```

Do **not** put the skill under `~/.cursor/skills-cursor/` — that path is reserved for Cursor’s built-in skills.

## Prerequisites

- [Cursor](https://cursor.com/) with Agent available
- Access to this skill package (clone/copy this repo, or copy only the skill folder)
- Optional for end-to-end bring-up later: a Zephyr west workspace + toolchain

## Choose an install location

| Mode | Path | When to use |
|------|------|-------------|
| **Project** (recommended) | `<your-project>/.cursor/skills/zephyr-board-bringup/` | Skill shared with the repo / team; matches this skill’s default design |
| **Personal** | `~/.cursor/skills/zephyr-board-bringup/` | Available in every project on that machine |

Pick one. Project install is the default for this skill.

## Install (project)

From the machine that already has the skill package:

```sh
# Adjust SOURCE and DEST for your paths
SOURCE=/path/to/zephyr_board_bringup/.cursor/skills/zephyr-board-bringup
DEST=/path/to/your-project/.cursor/skills/zephyr-board-bringup

mkdir -p "$(dirname "$DEST")"
cp -a "$SOURCE" "$DEST"
```

Or copy the whole repo and open it (or a project that already contains `.cursor/skills/zephyr-board-bringup/`) in Cursor.

## Install (personal)

```sh
SOURCE=/path/to/zephyr_board_bringup/.cursor/skills/zephyr-board-bringup
DEST="$HOME/.cursor/skills/zephyr-board-bringup"

mkdir -p "$HOME/.cursor/skills"
cp -a "$SOURCE" "$DEST"
```

## Verify the install

1. Confirm files exist under the chosen destination:
   - `SKILL.md`
   - `reference.md`
   - `examples.md`
   - `templates/inventory.md`
   - `templates/match-report.md`
   - `templates/howto.md`
2. Open `SKILL.md` and check frontmatter:
   - `name: zephyr-board-bringup`
   - non-empty `description`
   - `disable-model-invocation: true`
3. Restart Cursor or open a new Agent chat in a workspace that can see that skills path if the skill does not appear yet.

## Invoke the skill

The skill is **explicit-invoke only** (`disable-model-invocation: true`). In Agent chat, name it, for example:

- “Use the **zephyr-board-bringup** skill to bring up board \<name\>…”
- or ask the agent to follow `.cursor/skills/zephyr-board-bringup/SKILL.md`

Have ready when you start:

| Input | Required |
|-------|----------|
| Board name | Yes |
| Vendor hint / doc URLs / local docs | Helpful |
| Path to your Zephyr west workspace | Before SoC match / board creation |

## After install: running bring-up

Once installed, the skill walks phases A–E (collect references → identify CPU → match Zephyr SoC → create board support → sample build + board `howto.md`). Details live in the skill’s `SKILL.md`.

You still need:

1. A valid Zephyr workspace path for matching and board creation
2. Official board docs or local files for reference collection
3. Explicit confirmation before flash or overwriting existing paths

For package structure checks without hardware, see `specs/001-board-bringup-skill/quickstart.md`.

## Update or remove

**Update** — replace the installed folder with a fresh copy of `.cursor/skills/zephyr-board-bringup/` from this repo (same `cp -a` as above).

**Remove** — delete the installed `zephyr-board-bringup/` directory from the project or personal skills path.

## Troubleshooting

| Symptom | What to check |
|---------|----------------|
| Agent never loads the skill | Confirm path is `.cursor/skills/` or `~/.cursor/skills/`, not `skills-cursor`; invoke by name |
| Missing templates / broken phases | Re-copy the full tree; all six files above must be present |
| Skill runs but cannot match/create boards | Provide a real `zephyr_workspace` path; install Zephyr/west separately |
| Wrong project sees (or doesn’t see) the skill | Project skills are per-repo; use personal install for machine-wide availability |
