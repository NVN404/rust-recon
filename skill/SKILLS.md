# rust-recon Skill Runbook (v3.0, Strict Recon)

This skill is a deterministic runbook for generating protocol reconnaissance reports from
`rust-recon` outputs.

This workflow is recon-first, not vulnerability-rating-first.

## Canonical Priority (Mandatory)

If instructions conflict, resolve by priority:
1. `skill/core.md`
2. `skill/references/section-specs.md`
3. `skill/references/audit-patterns.md`
4. `skill/references/cpi-rules.md`
5. `skill/examples/*` (examples are non-authoritative)

## Hard Rules

1. Follow execution steps in order and do not skip checks.
2. Stop if prerequisites fail or required files are missing.
3. Every claim must map to a field from:
   - `.rust-recon/scope.json`
   - `.rust-recon/facts.json`
   - `.rust-recon/summary.json`
4. If data is missing, write exactly:
   - `Not extracted - verify manually.`
5. Do not invent sections, headings, or table schemas.
6. Do not use `Risk` or `Severity` columns in Section 2 tables.
7. Do not use red/yellow/blue indicator emoji in findings.
8. Diagrams in Section 4 must be ASCII only.
9. Do not generate report text using Python, shell templating, or custom scripts.
10. Keep the skill directory immutable during a run (no helper generators).
11. Use a strict two-pass flow: extraction validation first, report synthesis second.
12. Before report writing, emit checklist and evidence checkpoints.
13. If required schema is missing, fail the run and do not output a partial report.
14. Before final output, include command transcript summary and no-shortcut declaration.

## Step 0 - Prerequisites

Confirm all of the following before running extraction:
1. In Anchor workspace root
2. `Anchor.toml` exists
3. `rustc --version` succeeds

If `Anchor.toml` is missing, stop and return:
"This does not appear to be an Anchor workspace. Navigate to the project root and try again."

## Step 1 - Install rust-recon (if needed)

Check:
```bash
rust-recon --version
```

If missing:
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

Stop on install failure.

## Step 2 - Generate Data

From project root:
```bash
rust-recon scope
rust-recon facts
```

Required outputs:
- `.rust-recon/scope.json`
- `.rust-recon/facts.json`
- `.rust-recon/summary.json`

If any are missing, stop and report exact file(s).

## Step 3 - Read Context in Exact Order

Read in this sequence:
1. `skill/core.md`
2. `skill/references/facts-schema.md`
3. `skill/references/section-specs.md`
4. `skill/references/audit-patterns.md`
5. `skill/references/cpi-rules.md`
6. `.rust-recon/scope.json`
7. `.rust-recon/facts.json`
8. `.rust-recon/summary.json`

Do not use examples to infer schema.

## Step 4 - Select Report Mode

- Default: `detailed`
- Optional: `condensed` (only when user explicitly requests)

No third mode is allowed.

## Step 5 - Pass 1 Extraction Validation (Fail Closed)

Before drafting any report content, print PASS/FAIL for:
1. Prerequisites complete
2. Tool availability complete
3. `scope` and `facts` commands ran
4. Required JSON files exist
5. Required files read in exact order

Also print evidence checkpoints:
1. Ordered command list with purpose
2. Ordered file-read list
3. Active schema source (`skill/core.md` and `skill/references/section-specs.md`)

If any gate fails, stop and report the failure reason.

## Step 6 - Pass 2 Report Synthesis

Write `recon.md` in project root with the exact 1-9 section order defined in `skill/core.md`.

Section 2 is mandatory per instruction with 2a through 2f in order.

Generate `recon.md` only after Step 5 passes.

## Step 7 - Quality Gate (Fail Closed)

Before final output, all checks must pass:
1. Sections 1-9 exist in exact order.
2. Every instruction includes 2a, 2b, 2c, 2d, 2e, 2f.
3. Section 2a header is exactly: `Param | Type | Notes`
4. Section 2b header is exactly:
   `Account | Type | Mut | Signer | Unchecked | Constraint Summary`
5. Section 2e header is exactly: `Op | Style | Expression | Impact`
6. No forbidden columns in Section 2 (`Risk`, `Severity`).
7. Section 4 diagrams are ASCII with directional flow.
8. Section 8 checklist has one item per line.
9. Placeholder text is not used as bulk filler where evidence exists.
10. Any `Not extracted - verify manually.` line is evidence-backed.

If any check fails, regenerate only failed sections and re-run gate.

## Step 8 - Output Attestation (Required)

Before returning results, include:
1. Command transcript summary (every command run, in order, with purpose).
2. Explicit declaration: `No shortcut report generator was used.`
3. Reviewer note confirming skill compliance, not just completeness.

