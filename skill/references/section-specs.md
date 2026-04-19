### 1. Protocol Overview

Sources: `scope.json`, top-level fields of `summary.json`, `events[]` from `facts.json`.

Write:
- One paragraph describing what the protocol does. Infer from instruction names, account names,
  and emitted event names. **Explicitly label this as inference.**
- **Program Inventory table:**

  | Program Name | Program ID | Role |
  |---|---|---|

- Summary counts: instructions, account structs, PDAs, CPI calls, error codes, flags raised.
- Secondary programs in workspace (if any).
- External program dependencies inferred from `cpi_calls[].program` across all instructions:
  Token Program, Token-2022, System Program, ATA Program, etc.
- If `events[]` is non-empty, list all event names — they reveal the developer's intended
  state machine transitions and are the fastest semantic summary of the protocol.

---

### 2. Instruction Surface

For **every** instruction in `facts.json`, produce a subsection `#### <InstructionName>`.

---

#### 2a. Parameter Table

If `params[]` is non-empty:

| Param Name | Type | Overflow Risk |
|---|---|---|

Mark `overflow_risk: true` entries in the Overflow Risk column as ⚠ Yes.
If `params[]` is empty: `> No function-level parameters extracted.`

---

#### 2b. Signature Table

| Field | Value |
|---|---|
| Accounts consumed | all account names for this instruction |
| Signers required | accounts where `wrapper_type == "Signer"` |
| Unchecked accounts | accounts where `unchecked == true` — list with inner type |
| Mutable accounts | accounts with `mut` in `attributes` |
| Init accounts | accounts with `init` or `init_if_needed` in `attributes` |
| Close targets | `<account> → <close_target>` for any non-null `close_target` |
| has_one chains | `<account>.has_one = [...]` for every account with non-empty `has_one` |
| CPI calls made | `<program>::<method>` — from `cpi_calls[]` |
| Events emitted | from `events_emitted[]` |
| Remaining accounts | `uses_remaining_accounts` value |
| Error codes | `error_codes_referenced[]` |

**STRICT RULE for Signers Required:**
> **Signer extraction rule:** If `wrapper_type == "Signer"`, the account IS a signer. Do NOT rely on 
> Constraints or close_target to infer signer status — the wrapper_type field is ground truth.
> If `close_target` is set on a Signer, verify it is the payer/signer themselves (not privilege escalation).

---

#### 2c. Constraint Block

Reproduce every account's raw `attributes` string verbatim:

```
<AccountName>  [<wrapper_type><inner_type>]
  <attributes string>
  has_one: [...]
  close_target: <value or null>
```

---

#### 2d. Body Checks

If `body_checks[]` is non-empty, produce a table:

| Macro | Condition / LHS | RHS (keys_eq only) | Error Code |
|---|---|---|---|

If `body_checks[]` is empty:
> ⚠ No `require!` checks extracted for this instruction. Verify manually that
> access control is enforced in the function body.

---

#### 2e. Arithmetic Analysis

If `arithmetic[]` is non-empty, produce a table:

| Operation | Style | Expression | Overflow Risk |
|---|---|---|---|

Style values: `checked` ✅, `unchecked` ⚠, `saturating` ✅, `wrapping` ⚠.

If `arithmetic[]` is empty and the instruction has `u64` params or token transfers:
> ⚠ No arithmetic extracted but numeric params or token transfer present.
> Verify overflow handling manually.

---

### 3. Account & PDA Catalogue

#### 3b. PDA Catalogue

| PDA Name | Seeds (verbatim) | Bump Storage Field | Derived In |
|---|---|---|---|

After the table, for every PDA where seeds contain a `Pubkey` field or user key:

> ⚠ **Seed Analysis — `<PDA>`:** Seed component `<X>` is caller-supplied (type `Pubkey`).
> Verify the instruction validates `<X>` against an on-chain authoritative source before
> treating this PDA as a security boundary.

---

### 4. Authority & Trust Model

**🔴 CRITICAL: ASCII Diagrams Must Be RICH & DETAILED**

Section 4 is the heart of the security analysis. Use sophisticated ASCII art with:
- Multi-level hierarchies (not just flat lists)
- Role → Account → PDA → Field relationships (full nesting)
- Trust chains with explicit flow labels
- Branching for multiple control paths
- Seed component callouts
- Authority tag callouts [AUTHORITY], [NUMERIC], [TIMESTAMP], etc.

These diagrams go into GitHub and client reports. Make them publication-ready.

#### 4a. Authority Graph (ASCII Art)

**Purpose:** Show who controls what, and how trust flows through the protocol.

Build from `has_one` chains + `Signer` wrapper types using ASCII box/line art.

Example structure (adapt to your program):

```
       ┌────────────────────┐
       │   owner (Signer)   │
       └────────┬───────────┘
                │
       ┌────────▼──────────────┐
       │  config (init)        │
       │  [AUTHORITY]          │
       └────────┬──────────────┘
                │
      ┌─────────┼─────────┐
      │         │         │
   ┌──▼──┐  ┌──▼──┐  ┌──▼───┐
   │ tx1 │  │ tx2 │  │ tx3  │
   └─────┘  └─────┘  └──────┘
```

Rules:
- Each role/account is a box.
- Arrows (─→ or ├─→) show "controls" or "mutates" relationships.
- Label edges with instruction names.
- If a role controls multiple targets, use branching.

If < 2 signers: use prose instead.

#### 4b. Instruction Flow Diagram (ASCII Art)

Show instruction dependencies and state transitions using boxes and arrows.

Example structure:

```
    ┌─────────────┐
    │   init()    │
    │  (config)   │
    └──────┬──────┘
           │
    ┌──────▼──────────┐
    │ create_mission()│
    │   (mission PDA) │
    └──────┬──────────┘
           │
    ┌──────▼────────────┐
    │ accept_mission()  │
    │ (participant PDA) │
    └──────┬────────────┘
           │
    ┌──────▼──────────┐
    │ complete_ix()   │
    │   (settlement)  │
    └─────────────────┘
```

Show:
- Prerequisites (e.g., "init before create").
- State machine transitions.
- Grouped stages (setup, execution, settlement).

#### 4c. Account Dependency Diagram (ASCII Art)

Show which account structs depend on which via PDA seeds or `has_one` links.

Example structure (use boxes and nesting):

```
┌─────────────────────────────────────────────────────────┐
│                    config (root)                        │
│            [AUTHORITY, owner, executor, ...]           │
└──────────────────┬──────────────────────────────────────┘
                   │
        ┌──────────┼──────────┐
        │          │          │
    ┌───▼──┐   ┌───▼──┐   ┌──▼────┐
    │mission│   │ pool │   │ vault │
    │ (PDA) │   │(PDA) │   │(PDA)  │
    └───┬──┘   └───┬──┘   └──┬───┘
        │          │         │
    ┌───▼──────────▼─────────▼──┐
    │  participant / user_stake │
    │  (per-user account PDAs)  │
    └──────────────────────────┘
```

Rules:
- Root account (authority) at top.
- Each PDA seed component listed in relevant box.
- Nesting shows derivation hierarchy.
- Use `[TAGS]` from section 3a inline in boxes.

#### 4d. Trust Assumption Table

| Role | Controls | Assumed Honest | Privilege Escalation Path |
|---|---|---|---|

Trace `has_one` chains for escalation paths. Write `None identified` only after
checking every chain explicitly.

Flag if found:
- Single role controls pause flag AND treasury/vault → Centralisation risk.
- No time-lock, multi-sig, or DAO pattern in any account fields → Single point of failure.
- Admin is payer on multiple `init` instructions → Can grief protocol.

---

### 5. Token & CPI Flows

#### 5a. Token Account Table

| Account Name | Mint | Authority | Direction | Instructions |
|---|---|---|---|---|

Direction: `source` if this account is debited in a CPI transfer/burn,
`dest` if credited in transfer/mint_to, `both` if appears in both roles.
Infer from `cpi_calls[].method` and account position.

### 6. Error Code Registry

| Code | Name | Message | Instructions That Reference It |
|---|---|---|---|

Populate "Instructions That Reference It" from `error_codes_referenced[]` across all instructions.

After the table:
- Any error code with zero instructions referencing it → **Dead error code.** Either unused or
  referenced only in body code not yet extracted — verify manually.
- Any instruction with `u64` params or numeric arithmetic and no arithmetic-related error code
  (`Overflow`, `Underflow`, `InvalidAmount`) → **Missing overflow error coverage.**
- Any instruction with token transfers and no slippage/amount error code → **Missing slippage guard.**

---

### 7. Attack Surface Summary

**Import the pre-computed `flags[]` array from `facts.json` directly.**
Translate each flag entry into one table row. Do not re-derive — the extractor already did this.

Then add any flags raised in Sections 2–6 that are not already in `flags[]`
(e.g. trust model observations, dead error codes, missing arithmetic errors).

| # | Location | Finding | Severity | Source |
|---|---|---|---|---|

Severity scale:
- 🔴 **Critical** — missing signer on state mutation, arbitrary CPI, re-init on privileged account, unchecked account on vault.
- 🟠 **High** — unchecked arithmetic on reserve/balance field, user-controlled PDA seed unvalidated, oracle with no cap.
- 🟡 **Medium** — `init_if_needed` on non-privileged account, `close` to potentially caller-controlled target, single admin centralisation.
- 🔵 **Low/Info** — dead error codes, missing Token-2022 `transfer_checked`, no time-lock pattern, missing events.

Always append:
> *Severity ratings are triage hints from static structure only. Confirm exploitability
> with manual review and a working PoC before reporting.*

---

### 8. Manual Verification Checklist

Generate this list from **gaps in the extracted data** — not from a generic template.
For each gap, name the specific instruction and field.

Format:
```
[ ] <Instruction>: Verify <X> — rust-recon could not extract <Y> from <location>
```

Auto-generate the following checks when the condition is true:

| Condition | Checklist Item |
|---|---|
| `unchecked == true` on any account | `[ ] <Ix>/<Account>: Confirm manual validation in function body` |
| `body_checks[]` empty + CPI present | `[ ] <Ix>: Confirm access control logic exists in body — no require! extracted` |
| `uses_remaining_accounts == true` | `[ ] <Ix>: Enumerate expected remaining_accounts and their validation` |
| `arithmetic[]` empty + `u64` param | `[ ] <Ix>: Audit arithmetic on <param> — no operations extracted` |
| Any `timestamp` or `slot` field | `[ ] <Struct>: Verify clock manipulation resistance for <field>` |
| Token accounts + no `transfer_checked` | `[ ] <Ix>: Confirm Token-2022 compatibility if mint is upgradeable` |
| `events_emitted[]` empty on state-mutating ix | `[ ] <Ix>: No events emitted — consider whether indexers can observe this state change` |

---

### 9. Recon Metadata

```
Tool             : rust-recon v2
Generated        : <timestamp>
Source files     : .rust-recon/scope.json, facts.json, summary.json
Instructions     : <N>
Account structs  : <N>
PDAs             : <N>
CPI calls        : <N>
Error codes      : <N>
Pre-computed flags (extractor) : <N>
Flags total (report)           : <N>
```

> **EXACT COUNTS RULE:** All metadata counts must be exact integers. Do NOT use approximations like `11+`, `35+`, `8+`, or `~`. Count directly from `facts.json` arrays or enumerate manually. Every number must be precise.

---

## Generation Rules (Final)

1. **Never hallucinate.** No field, account, instruction, or constraint that is not in the JSON.
2. **Verbatim seeds and attributes.** Reproduce PDA seed arrays and `#[account(...)]` strings exactly.
3. **No generic filler.** Every sentence must name a specific account, instruction, field, or expression from the data.
4. **`flags[]` is imported, not re-derived.** Paste extractor flags into Section 7 first, then add report-level observations.
5. **`body_checks[]` absence is a finding.** An instruction with no extracted `require!` calls and mutable state is a gap — flag it.
6. **`arithmetic[]` absence + numeric params = manual check item.** Always.
7. **🔴 ASCII art diagrams ONLY in Section 4** — **NO Mermaid, NO markdown flowcharts**. Use boxes (┌─┐│└┘), lines (─│├┤┬┴┼), arrows (→↓). Make diagrams **RICH and DETAILED** with:
   - Section 4a: **Authority Graph** showing all roles, trust chains, control flows (not just simple boxes)
   - Section 4b: **Instruction Flow Diagram** with prerequisites, state transitions, grouped stages
   - Section 4c: **Account Dependency Diagram** with hierarchy, nesting, PDA seed components, [TAGS]
8. **Tables over lists** for anything with ≥ 3 entries.
9. **🔴 EXACT COUNTS RULE (Critical):** All numeric metadata in Section 9 and throughout the report must be **exact integers**. 
   - ✅ DO use: "17 instructions", "21 error codes", "70 arithmetic operations"
   - ❌ DON'T use: "11+", "35+", "8+", "~", or any approximations
   - Count directly from `facts.json` arrays or enumerate manually
   - Every number must be precise and traceable to source data
10. **DETAILED format is production-ready** — use as default. Condensed format only when explicitly requested for 50+ instruction codebases.

---

## Full Invocation Summary

```bash
# 1. Install (skip if already installed)
git clone https://github.com/0xkingx/rust-recon.git
cd rust-recon
cargo install --path cli
cd <back to your anchor project>

# 2. Generate data
rust-recon scope
rust-recon facts

# 3. Tell Claude:
# "Run the rust-recon skill against this project."
# Claude reads .rust-recon/*.json and writes recon.md
```
```