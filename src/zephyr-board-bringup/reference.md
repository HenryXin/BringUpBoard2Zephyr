# Reference: Zephyr Board Bringup

Supporting detail for [SKILL.md](SKILL.md). Load when executing matching,
board-file creation, or path normalization.

## Evidence and no-fabrication rules

- Every CPU/SoC claim MUST cite ≥1 reference item (path + short excerpt).
- Board hardware values (pinmux, clocks, memory map, flash layout, console
  UART, LEDs, buttons) MUST come from reference docs **or** explicit user
  input. Otherwise record a TODO and ask — never guess.
- Prefer official vendor documentation and Zephyr’s Board Porting Guide over
  forum snippets or unverified mirrors.
- When copying from a similar Zephyr board, treat that board as a **structural
  pattern** only; replace board-specific nodes with evidenced values or TODOs.

## Path normalization

- Default reference root: `<current_workspace>/reference/<normalized_board>/`.
- `normalized_board`: lowercase; replace spaces/underscores runs with `-`;
  keep `[a-z0-9-]`; collapse duplicate `-`; trim leading/trailing `-`.
- `vendor`: prefer Zephyr `dts/bindings/vendor-prefixes.txt` prefix when the
  vendor is listed; otherwise ask the user. Use `others` only if the vendor is
  unknown and the user agrees.
- Confirm final `vendor` / `board` directory names with the user before create.
- Never write skill files under `~/.cursor/skills-cursor/`.

## SoC similarity rubric

Search the user’s Zephyr tree (`soc/`, `boards/**/board.yml` socs lists,
existing board directories) in this order:

1. **Exact** — Same SoC name/id as detected (best).
2. **Similar** — Same series/family and compatible core; document residual
   risks (package, pinmux, memory, peripherals). Recommend user ACK before
   Phase D.
3. **Architecture-only** — Same ISA/core class only. Treat as **not sufficient**
   unless the user **explicitly accepts** high residual risk; otherwise outcome
   is `none`.
4. **None** — No acceptable match → hard stop; instruct manual SoC/port add.
   Do not create `boards/<vendor>/<board>/`.

Record matched example boards that use the chosen SoC — use them as copy
templates for HWMv2 files.

## HWMv2 board file checklist

Target layout (Zephyr Hardware Model v2):

```text
boards/<vendor>/<board>/
├── board.yml                 # REQUIRED — name, vendor, socs, variants
├── <board>[_qualifiers].dts  # REQUIRED — board hardware description
├── <board>-pinctrl.dtsi      # as needed
├── Kconfig.<board>           # REQUIRED — SoC/board Kconfig select
├── Kconfig.defconfig         # as needed
├── <board>_defconfig         # as needed
├── board.cmake               # runner / flash hooks as needed
└── <board>.yaml              # Twister metadata as needed
```

Mandatory before leaving Phase D:

| File | Required |
|------|----------|
| `board.yml` | Yes |
| ≥1 board `.dts` | Yes |
| `Kconfig.<board>` | Yes |

Copy structure from an existing board that uses the matched SoC, then replace
board-specific content using evidence. See Zephyr *Board Porting Guide*
(`docs.zephyrproject.org` → Hardware → Board Porting Guide) for `board.yml`
schema and board target naming (`board[@revision][/soc[/cpu][/variant]]`).

## Phase artifact locations (suggested)

| Artifact | Suggested path |
|----------|----------------|
| Inventory | `reference/<board>/inventory.md` |
| Match report | `reference/<board>/match-report.md` |
| Howto | `reference/<board>/howto.md` or beside the sample build dir |
| Board package | `<zephyr_workspace>/boards/<vendor>/<board>/` |
