# Execution Guardrails

These guardrails are STRICT constraints for any AI agent or LLM generating the `recon.md` report from `facts.json` output by the rust-recon tool.

## HARD CONSTRAINTS (Non-negotiable)

### 1. Single-Pass JSON Read
- Read `facts.json` EXACTLY ONCE.
- Extract all required data (instructions, programs, errors, CPIs, arithmetic, accounts) in ONE memory pass.
- No re-reading `facts.json` for any reason.
- No re-reading any reference files (core.md, facts-schema.md, etc.) after the first read.

### 2. No Temporary Files
- Do NOT create any intermediate files whatsoever.
- **Banned file patterns:**
  - `*_dump` (e.g., `ix_dump`, `program_dump`)
  - `*_brief` (e.g., `ix_brief`)
  - `*_extract`
  - `content.txt`
  - Any `.txt`, `.json`, or `.md` file except the final `recon.md`
- If you need to shape data, do it in-memory. Do not write scratch files to disk.

### 3. No Multiple Executions
- Complete the entire report generation in ONE response.
- No "let me run this command" followed by "let me read that output."
- No retry loops. No fallback queries.
- Integrate all logic into a single coherent workflow.

### 4. Direct Report Output
- Read `facts.json` → Extract data → Write `recon.md`.
- No intermediate steps visible to the user.
- Report must be fully detailed (9 sections, all instructions, 2a-2f subsections, Mermaid diagrams, checklists).

---

## Quality Requirements
- Zero loss of detail from the source `facts.json`.
- All instructions with full subsections (2a-2f).
- Readable, clean formatting without jumbling or truncation.
- All Mermaid diagrams intact and properly formatted according to schema.
- Architecture checklists complete and correct.
- Audit patterns and CPI rules applied correctly.

## Efficiency Targets
- **Token Target:** Minimize context window usage by skipping multiple extraction rounds.
- **Time Target:** Near-instant execution by performing generation natively in-memory rather than relying on iterative multi-step file reads.
- **Fallback:** If extraction fails, STOP. Do not retry, do not dump debug files. Ask the human operator for direction.
