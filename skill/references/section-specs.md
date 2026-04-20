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

## 2. Instruction Surface Format (FINAL SPEC)

Every instruction gets this EXACT layout in this EXACT order.
No exceptions. No condensing. No skipping subsections.

---

### INSTRUCTION HEADER (required for every instruction)

```
---
### 2.N — `instruction_name`
> **Context:** `ContextStructName` | **Signers:** N | **Accounts:** N | **Flags:** N
---
```

This one-line summary gives the reader instant orientation before diving into detail.

---

### 2a — Parameter Table

If params exist:

| Param | Type | Overflow |
|---|---|---|
| `amount` | `u64` | ⚠ Yes |
| `is_enabled` | `bool` | ✅ No |

If no params: `> No parameters.`

---

### 2b — Account Table (READABLE FORMAT — this is what was missing)

Use this table format. Every account gets one row. Columns are scannable.

| Account | Type | Mut | Signer | Unchecked | Constraint Summary |
|---|---|:---:|:---:|:---:|---|
| `config` | `Account<Configuration>` | ❌ | ❌ | ❌ | `seeds=[CONFIG_PDA_SEED], bump` |
| `owner` | `Signer` | ✅ | ✅ | ❌ | `mut, address = config.owner` |
| `fee_recipient` | `AccountInfo` | ✅ | ❌ | ⚠️ | `mut, address = config.fee_recipient` |
| `moderator_config` | `Account<ModeratorState>` | ❌ | ❌ | ❌ | `init_if_needed, seeds=[MISSIONX_MODERATOR, moderator.key]` |

RULES for this table:
- `Type` column: strip lifetimes, keep struct name. `Box<Account<'info, Configuration>>` → `Account<Configuration>`
- `Mut` column: ✅ if `mut` in constraint, ❌ otherwise
- `Signer` column: ✅ if `wrapper_type == Signer`, ❌ otherwise  
- `Unchecked` column: ⚠️ if `AccountInfo` or `UncheckedAccount`, ❌ otherwise
- `Constraint Summary` column: extract the KEY constraint only — seeds, address check, has_one, init type. NOT the full raw string.

---

### 2c — Account Fact Cards

For every account in the instruction, produce one fact card using this format:

**`account_name`** [`AccountType`] — `one-line role description (inference)`
- **Validated by:** list every validation present (seeds, address check, has_one,
  associated_token derivation, executable check) or `None` if absent
- **Mutation:** Mutable / Immutable / Init / Close
- **Gap:** what Anchor does NOT check (ownership, type, value range)
- **Manual check:** one specific thing to verify in source

RULES:
- `Validated by: None` is a meaningful finding. Write it explicitly.
- `Gap` is written only when a meaningful security check is missing.
- Role description is inferred from account name + context and must be labeled inference.
- Never reproduce the raw constraint string. Extract meaning into facts.
- Skip fact cards for: `system_program`, `rent`, `token_program`, `associated_token_program`.

Example fact cards:

**`bonding_curve`** [`Account<BondingCurve>`] — Mutable state PDA for reserves (inference)
- **Validated by:**
  - ✅ PDA seeds: `["bonding_curve", mint.key()]`
  - ✅ Cross-check: `bonding_curve.market == market.key()`
  - ✅ Bump present
- **Mutation:** Mutable
- **Gap:** No `has_one` or `close` constraint extracted
- **Manual check:** Verify bump is canonical and not accepted from unchecked input

**`trading_fee_treasury`** [`AccountInfo`] — Fee recipient account for trades (inference)
- **Validated by:**
  - ⚠️ Address match only: `market.trading_fee_treasury == key()`
- **Mutation:** Mutable
- **Gap:** `AccountInfo` wrapper means Anchor does not enforce owner/type checks
- **Manual check:** Confirm `market.trading_fee_treasury` is immutable after initialization

**`buyer`** [`Signer`] — Transaction authority for user trade (inference)
- **Validated by:**
  - ✅ Runtime signer verification
- **Mutation:** Mutable (lamports)
- **Gap:** No address-binding constraint; any signer can call
- **Manual check:** Confirm no privileged operation relies only on this signer

**`mint`** [`Account<Mint>`] — Token mint reference for curve operations (inference)
- **Validated by:** `None`
- **Mutation:** Immutable
- **Gap:** Unconstrained mint can be substituted unless checked in body/linked account constraints
- **Manual check:** Verify mint is tied to expected market/curve state in instruction logic

---

### 2d — Body Checks

If body_checks exist, use this format — one line per check, emoji prefix:

```
🔒 require!(config.initialized == false)  →  AlreadyInitialized
🔒 require!(payout >= config.missionx_payout_min)  →  MissionxPayoutTooSmall
🔒 require!(clock.unix_timestamp <= state.open_timestamp + state.open_duration)  →  MissionxFailedByExpiration
```

If no body checks:
```
⚠️ No require! checks extracted. Verify access control manually in function body.
```

---

### 2e — Arithmetic Analysis

If arithmetic exists:

| Op | Style | Expression | Risk |
|---|---|---|---|
| `add` | ⚠ unchecked | `reserve0 + effective_sol_spend` | Overflow on reserve accumulation |
| `sub` | ⚠ unchecked | `migration_threshold - reserve0` | Underflow if threshold < reserve |
| `div` | ⚠ unchecked | `checked_mul(...) / BPS` | Div after checked_mul — still unsafe |
| `mul` | ✅ checked | `effective_sol_spend.checked_mul(trade_fee_bps)` | Safe |

RULES:
- Expression column: truncate to 60 chars max, keep the meaningful part
- Risk column: one specific sentence about what breaks, not generic "overflow risk"
- Style: `✅ checked`, `⚠ unchecked`, `⚠ wrapping`, `✅ saturating`

If no arithmetic AND u64 params or token transfers present:
```
⚠️ No arithmetic extracted but u64 params / token transfers present.
   Verify: checked_add/sub/mul used in function body.
```

If no arithmetic and no numeric risk:
```
> No arithmetic operations.
```

---

### 2f — Audit Notes (SPECIFIC, not generic)

This section is the analyst's voice. Each bullet must be instruction-specific.

FORMAT RULES:
- 🔴 = critical, immediate exploit risk
- 🟠 = high, likely exploitable  
- 🟡 = medium, conditional risk
- ✅ = positive finding worth noting
- ⚠️ = needs manual verification

FORBIDDEN phrases (too generic, must never appear):
- "UncheckedAccount — zero Anchor validation"  ← replace with specific exploit
- "Verify manually"  ← replace with what specifically to verify
- "Overflow risk"  ← replace with which field and what breaks
- "Missing access control"  ← replace with what attacker can do without it

REQUIRED format for each flag:
```
🔴 `fee_recipient` is AccountInfo (mut, unchecked) — attacker passes own wallet
   as fee_recipient → ALL trading fees in every buy() redirect to attacker permanently.
   Protocol loses 100% of fee revenue. Verify Market.fee_recipient is validated
   against config.fee_recipient before any system::transfer.

🟡 `init_if_needed` on `moderator_config` — if moderator is re-added after removal,
   ModeratorState.is_enabled resets. Verify function body sets is_enabled = enable_moderator
   explicitly, not relying on zero-init default.

✅ `has_one = admin @ UnauthorizedAdmin` on config — only admin keypair can call this.
   Clean access control, no escalation path identified.

⚠️ `accept_missionx_multi` has no body checks extracted but shares context struct
   with `accept_missionx` (which has expiration check). Verify multi version also
   enforces open_timestamp + open_duration deadline — silent omission is a bug.
```

---

## COMPLETE INSTRUCTION EXAMPLE (copy this format exactly)

---
### 2.12 — `buy`
> **Context:** `BuyAccounts` | **Signers:** 1 (user) | **Accounts:** 9 | **Flags:** 5
---

#### 2.12a Parameters
| Param | Type | Overflow |
|---|---|---|
| `buy_amount` | `u64` | ⚠ Yes |
| `pay_cap` | `u64` | ✅ No |

#### 2.12b Accounts
| Account | Type | Mut | Signer | Unchecked | Constraint Summary |
|---|---|:---:|:---:|:---:|---|
| `user` | `Signer` | ✅ | ✅ | ❌ | `mut` |
| `config` | `Account<Configuration>` | ❌ | ❌ | ❌ | `seeds=[CONFIG_PDA_SEED], bump` |
| `missionx_state` | `Account<Missionx>` | ✅ | ❌ | ❌ | `mut, seeds=[MISSIONX_STATE, token_mint.key]` |
| `fee_recipient` | `AccountInfo` | ✅ | ❌ | ⚠️ | `mut, address=config.fee_recipient` |
| `token_mint` | `InterfaceAccount<Mint>` | ✅ | ❌ | ❌ | `mut, address=missionx_state.token_mint` |
| `token_program` | `AccountInfo` | ❌ | ❌ | ⚠️ | `executable, address=missionx_state.token_program` |
| `token_vault_pda` | `InterfaceAccount<TokenAccount>` | ✅ | ❌ | ❌ | `mut, seeds=[MISSIONX_TOKEN_VAULT, token_mint.key, missionx_state.key]` |
| `user_ata` | `InterfaceAccount<TokenAccount>` | ✅ | ❌ | ❌ | `mut, authority=user, mint=token_mint` |
| `system_program` | `Program<System>` | ❌ | ❌ | ❌ | — |

#### 2.12c Account Fact Cards

**`user`** [`Signer`] — User-authorized caller for buy path (inference)
- **Validated by:** ✅ Runtime signer verification
- **Mutation:** Mutable
- **Manual check:** Confirm no admin-only settings are reachable from this signer path

**`config`** [`Account<Configuration>`] — Global protocol configuration (inference)
- **Validated by:** ✅ PDA seeds + bump: `[CONFIG_PDA_SEED]`
- **Mutation:** Immutable
- **Manual check:** Verify config fields used in pricing are not stale or spoofable

**`missionx_state`** [`Account<Missionx>`] — Trade state and reserve context (inference)
- **Validated by:** ✅ PDA seeds + bump: `[MISSIONX_STATE, token_mint.key]`
- **Mutation:** Mutable
- **Manual check:** Verify seed source and account key are canonical for this market

**`fee_recipient`** [`AccountInfo`] — Fee sink destination (inference)
- **Validated by:** ⚠️ Address-only match: `config.fee_recipient`
- **Mutation:** Mutable
- **Gap:** `AccountInfo` does not enforce owner/type checks
- **Manual check:** Confirm config update path cannot redirect fees without admin authorization

**`token_mint`** [`InterfaceAccount<Mint>`] — Mint used in transfer and vault checks (inference)
- **Validated by:** ✅ Address match: `missionx_state.token_mint`
- **Mutation:** Mutable
- **Manual check:** Confirm mint authority and decimals assumptions in pricing logic

**`token_program`** [`AccountInfo`] — Token execution program account (inference)
- **Validated by:** ✅ Executable + address match: `missionx_state.token_program`
- **Mutation:** Immutable
- **Gap:** `AccountInfo` wrapper does not enforce concrete program type
- **Manual check:** Confirm state initialization cannot set a malicious token program address

**`token_vault_pda`** [`InterfaceAccount<TokenAccount>`] — Program-controlled token vault (inference)
- **Validated by:**
  - ✅ PDA seeds + bump: `[MISSIONX_TOKEN_VAULT, token_mint.key, missionx_state.key]`
  - ✅ Token constraints: authority = `missionx_state`, mint = `token_mint`, token_program = `token_program`
- **Mutation:** Mutable
- **Manual check:** Verify vault authority never escapes PDA control

**`user_ata`** [`InterfaceAccount<TokenAccount>`] — User token receiving account (inference)
- **Validated by:** ✅ ATA derivation: authority = `user`, mint = `token_mint`, token_program = `token_program`
- **Mutation:** Mutable
- **Manual check:** Confirm ATA is used consistently for transfer destination

Skipped in 2c by rule: `system_program` (standard program account)

#### 2.12d Body Checks
```
🔒 require!(effective_sol_spend <= pay_cap)  →  SlippageLimit
```

#### 2.12e Arithmetic
| Op | Style | Expression | Risk |
|---|---|---|---|
| `add` | ⚠ unchecked | `reserve0 + effective_sol_spend` | reserve0 can overflow if buys accumulate near u64::MAX |
| `sub` | ⚠ unchecked | `migration_threshold - reserve0` | underflow if reserve0 > migration_threshold (post-migration state) |
| `div` | ⚠ unchecked | `checked_mul(trade_fee_bps) / BPS` | integer division after checked mul — result still truncates |
| `mul` | ✅ checked | `effective_sol_spend.checked_mul(trade_fee_bps)` | Safe |

#### 2.12f Audit Notes
- 🔴 `fee_recipient` is AccountInfo (unchecked, mut) — though `address = config.fee_recipient`
  validates the key, AccountInfo wrapping means Anchor does NOT check ownership or type.
  If `config.fee_recipient` can be updated via `set_options`, attacker who compromises
  owner key redirects ALL buy fees by changing config first.

- 🔴 `token_program` is AccountInfo (unchecked) — `address = missionx_state.token_program`
  validates address but not program type. If missionx_state.token_program can be set to
  a malicious program address at create_missionx time, ALL token CPIs in buy() execute
  against attacker-controlled code.

- 🟠 Unchecked arithmetic on `reserve0 + effective_sol_spend` — bonding curve reserve
  has no overflow guard. At extreme buy volumes, reserve0 wraps to 0, breaking all
  subsequent price calculations. Add `checked_add` or `saturating_add`.

- 🟠 Unchecked `migration_threshold - reserve0` — if called after migration_threshold
  is lowered via set_options while reserve0 is high, subtraction underflows.
  Verify this expression only executes when reserve0 < migration_threshold.

- ✅ Slippage guard `pay_cap` is enforced before any state mutation. User cannot
  be front-run beyond their declared tolerance.

- ✅ `token_vault_pda` uses full PDA derivation with token::authority = missionx_state —
  vault authority is program-controlled, not user-supplied.

---

## ENFORCEMENT RULE FOR SKILL

SELF-CHECK before writing any instruction section:
1. Does the account table have ✅/❌/⚠️ columns? If not → redo
2. Does 2f have specific exploit paths, not generic phrases? If not → redo  
3. Is every unchecked account flagged with a concrete attack scenario? If not → redo
4. Are arithmetic risks named (which field, what breaks) not just "overflow risk"? If not → redo

---

### 3. Account & PDA Catalogue

#### 3a Account Structs
Per struct from `data_structs[]`:
Table: `| Field | Type | Tag | Notes |`
Include Re-init safety analysis and Authority chain.

#### 3b PDA Catalogue
Table: `| PDA Name | Seeds (verbatim) | Bump Storage Field | Derived In |`
Followed by **PDA Security Analysis**.

---

### 4. Authority & Trust Model

**🔴 CRITICAL: ASCII Diagrams Must Be RICH & DETAILED**
This section must use sophisticated ASCII art with boxes (┌─┐│└┘), lines (─│├┤┬┴┼), and arrows (▼, ►, ◄).

HARD ENFORCEMENT (non-optional):
- Do NOT output a single flat rectangle with text rows (this is invalid).
- Every diagram in 4a/4b/4c must contain at least:
  1) 3 or more boxed nodes
  2) 4 or more directional arrows
  3) 2 or more hierarchy levels (top actor/state -> child instruction/account)
- Show trust flow, not just lists. A reader must see source -> control edge -> target.
- If the current diagram can be converted to a table without losing information, it is too flat and must be redrawn.

#### 4a Authority Graph (ASCII Art)
Show who controls what, and how trust flows through the protocol mappings from actors down to specific instructions and derived accounts. Example:
```

Minimum content requirements for 4a:
- One trust boundary header block.
- Separate actor lane and protocol state lane.
- Explicit edges from each signer role to controlled instructions.
- Explicit edges from controlled instructions/roles to state accounts or PDAs.
- Call out all unchecked-account trust edges in a dedicated block.
┌──────────────────────────────────────────────────────────────┐
│                 PROTOCOL TRUST BOUNDARIES                    │
└──────────────────────────────────────────────────────────────┘
  [ Actor: OWNER ]
   │
   ├──➤ init()
   └──➤ set_options()
```

#### 4b Instruction Flow Diagram (ASCII Art)
Show instruction dependencies and state transitions (Setup Phase, Trading/Execution Phase, Completion) using ASCII boxes and arrows.

Minimum content requirements for 4b:
- Three phase boxes: setup, execution/trading, completion.
- One directional path between phases.
- At least one loop or bidirectional edge where protocol behavior is cyclical.
- Do not compress all phases into one vertical list.

#### 4c Account Dependency Diagram (ASCII Art)
Show which account structs depend on which via PDA seeds or `has_one` links in an ASCII tree/hierarchy.

Minimum content requirements for 4c:
- Show seed source nodes (for example mint/version inputs) feeding into PDA derivation nodes.
- Show at least one constraint edge (for example has_one / market equality relation).
- Show downstream token-account linkage for movement paths.

#### 4d Trust Assumption Table
| Role | Controls | Assumed Honest | Privilege Escalation Path | Risk |
|---|---|---|---|---|
List all authorities and outline single-points of failure.

---

### 5. Token & CPI Flows

#### 5a Token Account Flow Table
| Account Name | Mint | Authority | Balance Role | Instructions | Flow |
|---|---|---|---|---|---|

#### 5b CPI Call Map (ASCII Art)
Include a graphical mapping of Explicit CPI calls and Implicit Token Operations (via vault mutations).

#### 5c Token-2022 Compatibility
Assess compatibility and potential vectors (e.g. fee-on-transfer).

---

### 6. Error Code Registry
Table: `| Code | Name | Message | Instructions That Reference It |`
Followed by expectations and audit findings regarding missing error coverage.

---

### 7. Attack Surface Summary
Import the pre-computed `flags[]` array from `facts.json` directly. Translate each into a row.
| # | Location | Finding | Severity | Source | Exploit Path |
|---|---|---|---|---|---|

---

### 8. Manual Verification Checklist
Generate this list dynamically from **gaps in the extracted data**.
Format using categorizations:
```
CRITICAL SECURITY VERIFICATIONS:
═════════════════════════════════
[ ] <Instruction>: Missing body checks — verify bounds and logic
[ ] <Account>: Zero-constraint UncheckedAccount — verify payload

ADDITIONAL AUDIT ITEMS:
═══════════════════════
[ ] <Struct>::<Field>: Tagged [TIMESTAMP ⚠ manipulation] — verify dependencies
```

READABILITY ENFORCEMENT (non-optional):
- One checklist item per line. Never join multiple [ ] items into a paragraph.
- Insert one blank line between category headers and their checklist items.
- Keep each checklist line concise; target <= 140 characters per line.
- Group checklist items in this exact order:
  1) Missing body checks per instruction
  2) Unchecked account wrappers
  3) Arithmetic risk checks
  4) Struct-field risk tags
- Duplicate signals must be deduplicated (same instruction + same risk appears once).

---

### 9. Recon Metadata
Provide exact deterministic metrics for instructions analyzed, context structs modeled, CPIs traced, risks flagged, report lines, etc.
