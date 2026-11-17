---
name: zephyr-board-bringup
description: >-
  Collects vendor board docs into a reference folder, identifies the board
  CPU/SoC, matches the closest Zephyr-supported SoC, creates boards/<vendor>/<board>
  support or stops for manual SoC add, builds a minimal sample, and writes
  howto.md. Use when doing Zephyr board bring-up, custom board porting, new
  board support, or when a vendor board is not yet in Zephyr.
disable-model-invocation: true
---

# Zephyr Board Bringup

Guide an agent through bringing up a custom/vendor development board on Zephyr.
Deep checklists and rubrics live in [reference.md](reference.md). Example
sessions are in [examples.md](examples.md). Fill artifacts from
[templates/](templates/).

## Inputs (BoardRequest)

Collect before starting. Ask for anything missing that a later phase needs.

| Field | Required | Notes |
|-------|----------|-------|
| `board_name` | **Yes** | User-facing product name |
| `vendor_hint` | No | Helps disambiguation and `vendor/` directory |
| `doc_urls` | No | Preferred when user already has official links |
| `local_paths` | No | Local docs/SDK/BSP copies |
| `zephyr_workspace` | Before Phase C | West/Zephyr tree; **ask if unknown — never invent** |
| `board_target_hint` | No | Known Zephyr board target string if any |
| `reference_root` | No | Default: `<workspace>/reference/<normalized_board>/` |

**Disambiguation**: If the board name matches multiple products, stop and ask
the user which exact board/revision before any download.

## Global safety rules

- Confirm before overwriting an existing reference folder or board support path.
- Confirm before flash, mass-erase, or other destructive hardware ops (default: **skip flash**).
- **Never invent** pinmux, clocks, memory map, or flash layout. Ask the user or record a TODO.
- Cite evidence from reference docs for CPU identity and hardware claims.
- Never author skills under `~/.cursor/skills-cursor/`.
- Normalize `vendor` / `board` directory names to filesystem-safe forms and confirm with the user (see [reference.md](reference.md)).

## Workflow states

```text
init → collecting_references → references_ready → identifying_cpu
  → matching_zephyr_soc → [stopped_manual_soc END]
  → creating_board_support → building_sample → writing_howto → complete END
```

Any phase may enter `blocked_awaiting_user`. Resume only after the user answers.
Do **not** start a later phase until the prior gate passes (or hard-stop).

---

## Phase A — Collect references

**Goal**: Populate a reference folder and inventory for the board.

1. Resolve `reference_root` (default `reference/<normalized_board>/` under the
   current workspace). See path rules in [reference.md](reference.md).
2. If the folder already exists with content, ask: **refresh / reuse / abort**.
   Never silently overwrite. Set status `reuse` if reusing.
3. Prefer user `doc_urls` / `local_paths`. Copy or download into the reference
   folder.
4. If the user did not supply sources, search for **official** vendor/dev-kit
   documentation and SDK/BSP packages. Present candidates and download **only
   after user confirmation**.
5. If materials are behind login, paywall, or local-only installers, ask the
   user for accessible files/paths. Do not loop forever.
6. On mid-download failure, leave a **partial** inventory and recovery notes.
   Do **not** proceed to Phase B on incomplete critical docs.
7. Write inventory from [templates/inventory.md](templates/inventory.md) into
   the reference folder (e.g. `inventory.md`).

### Phase A gate

- Pass: inventory written; status `complete` or `reuse`.
- Fail: status `partial`/`empty` — list missing items, ask for sources, stay blocked.

---

## Phase B — Identify CPU

**Goal**: Determine CPU/SoC identity from the reference bundle with evidence.

1. Read datasheets, manuals, SDK readmes, and board docs in the reference folder.
2. Record `soc_name`, optional architecture, and **≥1 evidence citation**
   (document + short excerpt).
3. Set confidence `high` | `medium` | `low`. If not `high`, or documents
   conflict, set `needs_user_confirm` and **block until the user confirms**
   or supplies the SoC identity.
4. Keep results for the match report (Phase C).

### Phase B gate

- Pass: confirmed `CpuIdentity` with evidence.
- Fail: ambiguous with no user confirmation — do not match yet.

---

## Phase C — Match Zephyr SoC

**Goal**: Find the closest already-supported SoC in the user's Zephyr tree, or stop.

1. Require a valid `zephyr_workspace`. If missing or invalid, ask for the path;
   do not invent one.
2. Apply the similarity rubric in [reference.md](reference.md):
   exact SoC → series/family → architecture-only only with explicit user accept.
3. Search Zephyr `soc/`, board `board.yml` `socs:` lists, and existing boards
   that use candidate SoCs (as pattern sources).
4. Write the report from [templates/match-report.md](templates/match-report.md)
   (suggested path: `reference/<board>/match-report.md`).

### Outcomes

| Outcome | Action |
|---------|--------|
| `exact` or `similar` | Continue to Phase D. For `similar`, list residual risks; recommend user ACK. |
| `none` | **Hard stop.** Notify user to add CPU/SoC support manually. Include detected CPU details and suggested next steps. **Do not create board support.** END. |

### Phase C gate

- Pass: outcome `exact` or `similar` (and ACK if similar when user requires it).
- Hard stop: outcome `none` → `stopped_manual_soc`.

---

## Phase D — Create board support

**Goal**: Create HWMv2 board package `boards/<vendor>/<board>/`.

1. Confirm normalized `vendor` and `board` directory names with the user.
2. Target path: `<zephyr_workspace>/boards/<vendor>/<board>/` (or the user's
   `BOARD_ROOT` equivalent). If the path exists, require overwrite confirmation
   or abort.
3. Copy structural patterns from example boards that use the matched SoC.
4. Create mandatory files (see [reference.md](reference.md) checklist):
   - `board.yml`
   - board DTS (`<board>*.dts`, pinctrl as needed)
   - `Kconfig.<board>`
   - Plus defconfig / `board.cmake` / yaml as required by the pattern board
5. Map **only evidenced** hardware (or user-supplied values). Missing
   pins/clocks/flash/console UART → ask the user or record TODOs. **Never invent.**
6. List all TODOs / known gaps before leaving the phase.

### Phase D gate

- Pass: `board.yml` present, ≥1 DTS, `Kconfig.<board>` present, todos listed.
- Fail: incomplete structure or fabricated values — fix or ask before continuing.

---

## Phase E — Sample build and howto

**Goal**: Prove the board target builds (when possible) and leave runnable docs.

1. Prefer Zephyr `samples/hello_world`. Use blinky-style only if LED nodes are
   evidenced in the board DTS/references.
2. Resolve full `board_target` string (name[/qualifiers]).
3. Attempt `west build -b <board_target> <sample>` in the Zephyr workspace.
4. Record `build_status`: `success` | `failed` | `skipped` and a short summary.
5. **Flash**: default **skip**. Attempt flash only after explicit user
   confirmation. Never mass-erase without confirm.
6. Always write howto from [templates/howto.md](templates/howto.md)
   (suggested path: beside the sample workdir or `reference/<board>/howto.md`).
   On build failure, still write howto and document gaps / manual fixes.

### Phase E gate

- Pass: `build_status` set and `howto.md` written (flash optional).

---

## Done

Session is complete when either:

- **Success path**: Phases A–E finished (flash may be skipped), or
- **Hard stop**: Phase C outcome `none` with manual SoC instructions delivered.

Summarize artifacts created and any open TODOs for the user.
