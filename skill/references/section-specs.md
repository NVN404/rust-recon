# Section Specs (v3.0, Recon-First)

This file defines the exact report contract.

If this file conflicts with another reference, follow `skill/core.md`.

## Global Rules

1. Use only extracted facts from `.rust-recon/*.json`.
2. No custom section names beyond the 1-9 contract.
3. If data is missing, write exactly:
   - `Not extracted - verify manually.`
4. Use neutral recon language.
5. Do not use red/yellow/blue indicator emoji.
6. In Section 2 tables, never use `Risk` or `Severity` columns.
7. Use ASCII-only diagrams in Section 4.

## Report Modes

### Detailed (default)
- Full Section 2a-2f per instruction.
- Complete Section 3 through Section 9.

### Condensed (only on explicit user request)
- Section 2 as a single summary table plus short notes.
- Sections 3-9 still required.

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

---

## 1. Protocol Overview

Sources:
- `.rust-recon/scope.json`
- `.rust-recon/facts.json`
- `.rust-recon/summary.json`

Required content:
1. One inferred purpose paragraph labeled as inference.
2. Program inventory table:

| Program Name | Program ID | Role |
|---|---|---|

3. Exact counts summary:
   - instructions
   - context structs
   - data structs
   - CPI calls
   - error codes
   - parser flags
4. External dependency list inferred from account wrappers and CPI targets.

---

## 2. Instruction Surface

Detailed mode: every instruction must include 2a through 2f in order.

### Instruction Header (required)

```text
---
### 2.N - instruction_name
Context: ContextStructName | Signers: N | Accounts: N | Flags: N
---
```

### 2a - Parameter Table

If parameters exist, use exactly this header:

| Param | Type | Notes |
|---|---|---|

Notes guidance:
- Numeric scalar (`u64`, `i64`, `u128`): `numeric input`
- Boolean: `boolean switch`
- Pubkey: `address input`
- Struct/enum: `structured payload`
- Bytes/vector/string: `variable-size input`

If none:
- `No parameters.`

### 2b - Account Table

Use exactly this header:

| Account | Type | Mut | Signer | Unchecked | Constraint Summary |
|---|---|:---:|:---:|:---:|---|

Rules:
1. `Type`: strip lifetimes and keep readable wrapper.
2. `Mut`: `Yes` if mutable, otherwise `No`.
3. `Signer`: `Yes` if signer wrapper or signer flag, otherwise `No`.
4. `Unchecked`: `Yes` for `AccountInfo` or `UncheckedAccount`, otherwise `No`.
5. `Constraint Summary`: short, key-only summary (seeds, has_one, address, init).

### 2c - Account Fact Cards

Emit one card per non-skipped account.
Skip cards for standard program/sysvar accounts:
- `system_program`
- `rent`
- `token_program`
- `associated_token_program`

Card template:

```text
account_name [AccountType] - one-line role description (inference)
- Validated by: concise list, or None
- Mutation: Mutable | Immutable | Init | Close
- Gap: only if meaningful
- Manual check: one concrete source check
```

### 2d - Body Checks

If checks exist, one line per check:

```text
require!(condition) -> ErrorName
require_keys_eq!(lhs, rhs) -> ErrorName
```

If none:
- `No extracted body checks. verify manually.`

### 2e - Arithmetic Analysis

If arithmetic exists, use exactly this header:

| Op | Style | Expression | Impact |
|---|---|---|---|

Rules:
1. `Style`: `checked`, `unchecked`, `wrapping`, or `saturating`.
2. `Expression`: concise expression, keep meaningful operands.
3. `Impact`: concrete state impact if operation fails or wraps.

If none and numeric params exist:
- `No extracted arithmetic. verify checked arithmetic in source.`

If none and no numeric path:
- `No arithmetic operations.`

### 2f - Recon Notes (Condensed)

Use at most 4 bullets per instruction.
Use one of these prefixes only:
- `[fact]`
- `[check]`
- `[gap]`

Rules:
1. Notes must be instruction-specific.
2. Avoid generic text.
3. Do not use severity grading icons.

---

## 3. Account and PDA Catalogue

### 3a - Account Structs

For each `data_structs[]` entry:

| Field | Type | Tag | Notes |
|---|---|---|---|

Tag set:
- `[STORED_BUMP]`
- `[AUTHORITY]`
- `[NUMERIC]`
- `[TIMESTAMP]`
- `[PAUSE_FLAG]`
- `[PUBKEY]`
- `[ACCOUNTING]`

After each struct include:
1. `Used by instructions:`
2. `Re-init safety:`
3. `Authority chain:`

### 3b - PDA Catalogue

| PDA Name | Seeds (verbatim) | Bump Storage Field | Derived In |
|---|---|---|---|

Then add concise PDA integrity notes.

---

## 4. Authority and Trust Model

ASCII diagrams only. No Mermaid.

### 4a - Authority Graph
Requirements:
1. At least 3 boxed nodes.
2. At least 4 directional edges.
3. At least 2 hierarchy levels.

### 4b - Instruction Flow
Requirements:
1. Setup, execution, completion phases.
2. Directional phase transitions.
3. At least one loop or repeat path where applicable.

### 4c - Account Dependency Diagram
Requirements:
1. Seed source nodes.
2. PDA derivation nodes.
3. Constraint links (`has_one`, address equality, authority links).

### 4d - Trust Assumption Table

| Role | Controls | Assumed Honest | Escalation Path | Observation |
|---|---|---|---|---|

---

## 5. Token and CPI Flows

### 5a - Token Account Flow Table

| Account Name | Mint | Authority | Balance Role | Instructions | Flow |
|---|---|---|---|---|---|

### 5b - CPI Call Map

List explicit CPI calls and likely implicit token operations when extraction is incomplete.

If CPIs appear incomplete, add a concise limitation note with required manual checks.

### 5c - Token-2022 Compatibility

If Token-2022 indicators appear, note extension-sensitive checks:
- transfer method compatibility
- fee-on-transfer accounting implications

---

## 6. Error Code Registry

| Code | Name | Message | Referenced By |
|---|---|---|---|

Then add a short coverage comment:
- missing coverage areas
- inconsistent error mapping

---

## 7. Recon Signals Summary

Derive rows from `flags[]`, `body_checks[]`, `accounts[]`, and `arithmetic[]`.

Use this exact table:

| # | Location | Signal | Evidence | Required Check | Source |
|---|---|---|---|---|---|

Rules:
1. Do not include a severity column.
2. Keep rows concise and factual.
3. Merge duplicate signals by location + signal.
4. If no signals, state `No recon signals extracted.`

---

## 8. Manual Verification Checklist

One checklist item per line.
Use grouped sections in this exact order:
1. Missing body checks
2. Unchecked account wrappers
3. Arithmetic validation
4. Struct field and state assumptions
5. CPI target and signer seed checks

Checklist format:

```text
[ ] instruction_name: specific manual verification task
```

---

## 9. Recon Metadata

Provide deterministic counts only:
- instructions analyzed
- context structs modeled
- account structs catalogued
- PDA entries
- CPI calls traced
- error codes indexed
- checklist items generated

Do not estimate. No approximations.
