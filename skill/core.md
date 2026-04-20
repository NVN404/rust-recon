# rust-recon Core Rules (v3.0, Recon-First)

This file is the canonical authority for report behavior.

If any instruction conflicts across files, use this precedence:
1. `skill/core.md`
2. `skill/guardrails.md`
3. `skill/references/section-specs.md`
4. `skill/references/audit-patterns.md`
4. `skill/references/cpi-rules.md`
5. `skill/examples/*` (examples are non-authoritative)

## Purpose

Generate deterministic protocol reconnaissance reports from extracted facts.

This is a recon workflow, not a vulnerability rating workflow.

## Hard Rules

1. Follow workflow steps in order. Do not skip steps.
2. Stop immediately if prerequisites or generated files are missing.
3. Every claim must be traceable to one of these files:
   - `.rust-recon/scope.json`
   - `.rust-recon/facts.json`
   - `.rust-recon/summary.json`
4. If data is missing, write exactly:
   - `Not extracted - verify manually.`
5. Do not invent sections, table schemas, or column names.
6. In Section 2 tables, never use a `Risk` column or a `Severity` column.
7. Keep observations concise and evidence-based.
8. Do not use red/yellow/blue indicator emoji in findings.
9. Do not output Mermaid. Section 4 diagrams must be ASCII only.
10. If quality checks fail, regenerate before final output.
11. Do not use Python, shell templating, or custom scripts to generate recon report text.
12. Use only this skill workflow plus direct reasoning over extracted files.
13. Keep the skill directory immutable during a run; do not add helper scripts or ad-hoc generators.
14. Before report writing, produce a pass/fail pre-execution checklist.
15. Before report writing, print evidence checkpoints: exact command log, exact files read in order, and section schema source.
16. Use a mandatory two-pass flow: Pass 1 extraction validation, then Pass 2 report synthesis.
17. If forbidden placeholder patterns appear in core sections, stop and regenerate.
18. If any required section, subsection, or table header is missing, fail the run.
19. Before final output, provide an explicit no-shortcut declaration.

## Workflow (Strict Order)

0. Validate Anchor workspace and toolchain.
1. Ensure `rust-recon` binary is available.
2. Run `rust-recon scope` and `rust-recon facts`.
3. Verify all three output files exist.
4. Read rules and facts in required order.
5. Execute Pass 1 extraction validation gates.
6. Generate `recon.md` with exact section order (Pass 2 only after Pass 1 passes).
7. Run quality gate and regenerate failed sections.
8. Emit command transcript summary and no-shortcut declaration.

## Pre-Execution Checklist (Required)

Before any report text is drafted, print this checklist with PASS or FAIL:
1. Prerequisites check complete (`Anchor.toml`, `rustc --version`).
2. Tool availability check complete (`rust-recon --version`).
3. Extraction commands ran (`rust-recon scope`, `rust-recon facts`).
4. Required output files exist (`scope.json`, `facts.json`, `summary.json`).
5. Required files were read in the mandated order.

If any item fails, stop and report failure reason. Do not draft the report.

## Evidence Checkpoints (Required)

Before report synthesis, print:
1. `Commands run (ordered):` command plus short purpose.
2. `Files read (ordered):` exact file paths.
3. `Schema source:` `skill/core.md` and `skill/references/section-specs.md`.

If this evidence block is missing, the run is invalid.

## Pass Model (Mandatory)

Pass 1: Extraction Validation
1. Validate checklist and evidence checkpoints.
2. Confirm required JSON fields are present for all mandatory sections.
3. Approve or fail with explicit reason.

Pass 2: Report Synthesis
1. Start only after Pass 1 is approved.
2. Generate report with strict section and table schemas.
3. Run quality gate.

Do not merge both passes into a single unchecked generation step.

## Report Modes

- Default mode: `detailed`
- Optional mode: `condensed` (only when explicitly requested)

## Section Order (Mandatory)

1. Protocol Overview
2. Instruction Surface
3. Account and PDA Catalogue
4. Authority and Trust Model
5. Token and CPI Flows
6. Error Code Registry
7. Recon Signals Summary
8. Manual Verification Checklist
9. Recon Metadata

## Section 2 Non-Negotiables

For every instruction, include 2a through 2f in order.

- 2a table header must be exactly:
  - `Param | Type | Notes`
- 2b table header must be exactly:
  - `Account | Type | Mut | Signer | Unchecked | Constraint Summary`
- 2e table header must be exactly:
  - `Op | Style | Expression | Impact`

## Recon Signals Rules (Section 7)

- Use concise signal rows with evidence and required checks.
- Do not include numeric severity labels in table columns.
- If `flags[]` contains severity, treat it as internal parser metadata only.

## Quality Gate (Must Pass)

1. Section order is exactly 1 through 9.
2. Every instruction has 2a, 2b, 2c, 2d, 2e, 2f.
3. Section 2 tables use exact required headers.
4. No `Risk` column in 2a/2b.
5. No red/yellow/blue indicator emoji in findings sections.
6. Section 4 contains ASCII diagrams with directional flow.
7. Checklist section uses one item per line.
8. Core sections are not filled with repeated placeholder-only text.
9. If placeholder text appears, it must correspond to truly missing extracted data.
10. Command transcript summary is present.
11. Explicit declaration is present: `No shortcut report generator was used.`

## Forbidden Pattern Gate

Stop and regenerate if either condition is true:
1. Repeated `Not extracted - verify manually.` appears across core sections while extracted evidence exists.
2. Large placeholder blocks replace required tables where data is present.

This gate is fail-closed.
