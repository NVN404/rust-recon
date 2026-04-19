# rust-recon Claude Orchestrator

This is the entry point for generating deep security recon reports using rust-recon.

## Step 0 — Prerequisites Check
Before anything else, confirm:
1. You are inside a Solana Anchor project directory.
2. `Anchor.toml` exists at the workspace root.
3. Rust toolchain is installed (`rustc --version` should succeed).
If `Anchor.toml` is not found, stop and tell the user: "This does not appear to be an Anchor workspace."

## Step 1 — Install rust-recon & Skill Context
Since Claude Skills cannot easily bundle Rust binaries, we clone the tool repository locally (if not already present), install the binary, and map our skill context to this process.

Check if `~/.rust-recon_tool` exists.
```bash
export TARGET_DIR=~/.rust-recon_tool
if [ ! -d "$TARGET_DIR" ]; then
    git clone https://github.com/NVN404/rust-recon.git $TARGET_DIR
    cd $TARGET_DIR
    cargo install --path cli
else
    cd $TARGET_DIR && git pull
fi
rust-recon --version
```

## Step 2 — Generate Recon Data
Return to the project directory and run:
```bash
rust-recon scope
rust-recon facts
```

## Step 3 — Load Context
Read these files from the current skill directory IN ORDER before writing a single line of output:
1. `skill/core.md`          — Hard rules, section order (1-9)
2. `skill/references/facts-schema.md`    — What fields mean
3. `skill/references/section-specs.md`  — What each section contains
4. `skill/references/audit-patterns.md` — Mechanical checks (2f, 3a)
5. `skill/references/cpi-rules.md`      — CPI handling and Token-2022
6. `skill/examples/*`      — Example reports for reference
7. `.rust-recon/scope.json` (from current project)
8. `.rust-recon/facts.json` (Ground truth, from current project)
9. `.rust-recon/summary.json` (from current project)

> **Note:** If unsure about output quality or formatting, refer to `skill/examples/` for reference.

## Step 4 — Detect Report Format from User Input
Parse the user's input to determine report style:
- If user input contains `condensed`, use **Condensed Format** (250-350 lines, ASCII only, exact counts)
- If user input contains `detailed`, use **Detailed Format** (1000+ lines, full analysis)
- Default: **Standard Format** (550-700 lines, balanced detail)

**Important:** The format choice affects:
- Section 2 (Instruction Surface): 1 summary table (condensed) vs. detailed per-instruction breakdown (detailed)
- Diagrams: ASCII art only (no Mermaid in condensed mode)
- Tone: Executive summary vs. comprehensive technical analysis
