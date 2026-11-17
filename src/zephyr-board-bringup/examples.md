# Examples: Zephyr Board Bringup

## Happy path

**User**: “Bring up my Acme ProtoBoard X1 on Zephyr. Docs:
https://acme.example/protoboard-x1/docs (confirmed). Zephyr is at
`~/zephyrproject`.”

**Agent**:

1. **Phase A** — Creates `reference/protoboard-x1/`, downloads confirmed docs,
   writes `inventory.md` with status `complete`.
2. **Phase B** — From the datasheet: SoC `ACM32F405`, Cortex-M4, confidence
   `high`, cites datasheet excerpt.
3. **Phase C** — Finds Zephyr SoC `acm32f405` (exact) used by
   `boards/acme/acm32f405_dk`. Writes `match-report.md` outcome `exact`.
4. **Phase D** — Confirms vendor `acme`, board `protoboard_x1`. Creates
   `~/zephyrproject/zephyr/boards/acme/protoboard_x1/` with `board.yml`, DTS,
   `Kconfig.protoboard_x1`. Console UART from schematic; LED pins TODO until
   user confirms.
5. **Phase E** — `west build -b protoboard_x1 samples/hello_world` → success.
   Flash skipped (no confirm). Writes `howto.md` with build/console steps.

**Artifacts**: reference inventory, match report, board package, howto.

## No-match hard stop

**User**: “Port ExoticChip Eval Kit. Zephyr at `~/zephyrproject`.” Local PDF
provided.

**Agent**:

1. **Phase A** — Copies PDF into `reference/exoticchip-eval-kit/`, inventory
   `complete`.
2. **Phase B** — SoC `EXC-RISC9000` from PDF, confidence `high`.
3. **Phase C** — No exact/series match in Zephyr; architecture-only RISC-V
   parts exist but user does **not** accept high residual risk → outcome
   `none`. Writes match report with manual next steps (add SoC under `soc/`,
   then re-run). **Does not** create `boards/...`. **Does not** build sample.

**Artifacts**: reference inventory + match report only; stop message for
manual SoC support.
