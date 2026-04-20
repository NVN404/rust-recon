# rust-recon Orchestrator (Strict, Recon-First)

This command is the only execution path for generating `recon.md`.

## Source Priority (Mandatory)

If guidance conflicts, resolve in this order:
1. `skill/core.md`
2. `skill/references/section-specs.md`
3. `skill/references/audit-patterns.md`
4. `skill/references/cpi-rules.md`
5. `skill/examples/*` (non-authoritative)

Do not invent formatting outside these rules.

## Execution Guardrails (Non-Negotiable)

1. Do not use Python, shell templating, or custom scripts to generate recon report text.
2. Do not create helper generators in the skill directory during a run.
3. Use a strict two-pass flow:
    - Pass 1: extraction validation
    - Pass 2: report synthesis
4. Before Pass 2, print checklist PASS/FAIL and evidence checkpoints.
5. If any required section, subsection, or table header is missing, fail the run.
6. If placeholder-only text appears where extracted data exists, stop and regenerate.
7. Before final output, include command transcript summary and explicit no-shortcut declaration.

## Step 0 - Prerequisites Check

Before anything else, confirm:
1. You are in a Solana Anchor project directory.
2. `Anchor.toml` exists at workspace root.
3. Rust toolchain is available (`rustc --version`).

If `Anchor.toml` is missing, stop and return:
"This does not appear to be an Anchor workspace. Navigate to the project root and try again."

## Step 1 - Ensure rust-recon Binary

Check binary first:

```bash
rust-recon --version
```

If missing, install:

```bash
export TARGET_DIR=~/.rust-recon_tool
if [ ! -d "$TARGET_DIR" ]; then
    git clone https://github.com/NVN404/rust-recon.git "$TARGET_DIR"
else
    cd "$TARGET_DIR" && git pull
fi
cd "$TARGET_DIR"
cargo install --path cli
rust-recon --version
```

If install fails, report the exact error and stop.

## Step 2 - Generate Recon Data

From the Anchor project root:

```bash
rust-recon scope
rust-recon facts
```

Required outputs:
- `.rust-recon/scope.json`
- `.rust-recon/facts.json`
- `.rust-recon/summary.json`

If any file is missing, stop and report exactly which command/file failed.

## Step 3 - Read Context in Exact Order

Read these files in this exact sequence before drafting any report text:
1. `skill/core.md`
2. `skill/references/facts-schema.md`
3. `skill/references/section-specs.md`
4. `skill/references/audit-patterns.md`
5. `skill/references/cpi-rules.md`
6. `.rust-recon/scope.json`
7. `.rust-recon/facts.json`
8. `.rust-recon/summary.json`

Important:
- Do not use examples to define schema.
- Examples may be consulted only for diagram readability when needed.

## Step 4 - Choose Format

- If user explicitly says `condensed`, use condensed mode.
- Otherwise default to `detailed`.

No third "standard" mode is allowed.

## Step 5 - Pass 1 Extraction Validation (Required)

Before drafting report content, print PASS/FAIL for:
1. Prerequisites complete
2. Tool availability complete
3. Extraction commands executed
4. Required output files exist
5. Required files read in exact order

Then print evidence checkpoints:
1. Ordered command list with short purpose
2. Ordered list of files read
3. Active schema source (`skill/core.md`, `skill/references/section-specs.md`)

If any check fails, stop and report exact failure reason.

## Step 6 - Generate Report (Pass 2 Only)

Write `recon.md` at project root.

Mandatory constraints:
- Follow section order 1-9 from `skill/core.md`.
- For each instruction, include subsections 2a-2f in order.
- Section 2a table header must be exactly: `Param | Type | Notes`
- Section 2b table header must be exactly: `Account | Type | Mut | Signer | Unchecked | Constraint Summary`
- Section 2e table header must be exactly: `Op | Style | Expression | Impact`
- No `Risk` or `Severity` column in Section 2 tables.
- No red/yellow/blue indicator emoji in findings sections.
- Diagrams must be ASCII only.

## Step 7 - Quality Gate (Fail Closed)

Before returning output, validate:
1. All 9 sections exist in the required order.
2. Every instruction has all subsections 2a-2f.
3. Table headers exactly match required schema.
4. No forbidden columns (`Risk`, `Severity`) in Section 2 tables.
5. Section 4 diagrams show directional flow (not flat lists).
6. Section 8 checklist has one item per line.
7. Placeholder text is not used as a bulk substitute for extracted content.
8. Any `Not extracted - verify manually.` line is evidence-backed.

If any check fails, regenerate the affected section(s) and re-run this gate.

## Step 8 - Output Attestation (Required)

Before final output, provide:
1. Command transcript summary: every command run, in order, with purpose.
2. Explicit declaration: `No shortcut report generator was used.`
3. Reviewer statement: skill-compliant schema check passed.
