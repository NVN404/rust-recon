# rust-recon Core Rules (v3.0, Recon-First)

This file is the canonical authority for report behavior.

If any instruction conflicts across files, use this precedence:
1. `skill/core.md`
2. `skill/references/section-specs.md`
3. `skill/references/audit-patterns.md`
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

## Workflow (Strict Order)

0. Validate Anchor workspace and toolchain.
1. Ensure `rust-recon` binary is available.
2. Run `rust-recon scope` and `rust-recon facts`.
3. Verify all three output files exist.
4. Read rules and facts in required order.
5. Generate `recon.md` with exact section order.
6. Run quality gate and regenerate failed sections.

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
