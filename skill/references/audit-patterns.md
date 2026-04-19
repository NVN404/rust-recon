#### 2f. Audit Notes (mechanical — STRICT contradiction checking)

**🔴 MANDATORY CROSS-REFERENCE RULES:**

For EVERY account with `close_target` non-null, add this verification:
1. Check the instruction's `attributes` for `close = <target>` macro
2. Cross-check the instruction's name/behavior: Does it actually CLOSE the account?
3. **If close_target exists BUT the instruction does NOT close the account → FLAG AS CONTRADICTION**

Example:
```
✅ CORRECT:
  unstake_locked: close_target = user_stake → Instruction name is "unstake" ✓ Behavior is close ✓

❌ CONTRADICTION:
  instant_unlock: close_target = user_stake → But instruction does NOT close account ✗ 
  FLAG: "close_target declared but account NOT closed. Remove close = target or add closure logic."
```

---

**Check each condition and write a bullet only if true:**

- `init_if_needed` in any account `attributes` → **Re-initialization risk.** Account `<name>` can be re-entered. Verify all fields are reset correctly on re-init. Cite the field from Section 3a that is most at-risk.
- `close_target` is non-null **AND instruction name/behavior matches** → **Lamport drain target is `<close_target>`.** Verify this is a protocol-controlled account, not caller-supplied.
- `close_target` is non-null **BUT instruction does NOT close** → 🔴 **CONTRADICTION: close_target declared but no account closure in instruction. Verify in source or remove macro.** This is a parser or logic error.
- `unchecked == true` on any account → **`<name>` is `<wrapper_type>` (zero Anchor validation).** Every check is manual — confirm in body.
- `uses_remaining_accounts == true` → **Unvalidated account injection surface.** Any account can be passed here.
- `body_checks[]` is empty AND (`mut` accounts exist OR CPI calls exist) → **No extracted access control.** Either validation is in the macro layer only or is absent.
- Any param with `overflow_risk: true` AND any `arithmetic[].style == "unchecked"` on a matching expression → **Unchecked arithmetic on overflow-risk param `<name>`.** Confirm `checked_*` or `saturating_*` used in full source.
- `cpi_calls[]` contains a non-token program → **CPI to non-standard program `<program>`.** Verify address is validated at call site.
- `has_one` chain on a mutable account with no signer → **Trust chain relies on `has_one = <field>`.** Confirm the field is set during `init` by an authorised signer and is immutable after.
- `init` with `payer =` that is not the instruction's signer → **Payer manipulation risk.**

---

#### 3a. Account Structs (Field-Level Analysis — STRICT)

**🔴 MANDATORY: Every struct must have a complete field table. NO exceptions.**

For each `#[account]` struct in `facts.json`, produce:

**Struct Name: `<StructName>`**

| Field | Type | Tag | Notes |
|---|---|---|---|

**STRICT RULES for field extraction:**
> If a field's inner type, default value, or constraint cannot be extracted, write:
> `> Not extracted — verify manually in source.`
> 
> **FORBIDDEN:** Vague placeholders like "(verify in source)", "(verify curve formula...)", "(...)", "(storage details)", etc.
> 
> **CORRECT:** `| pricing_formula | bytes | [STORED] | > Not extracted — verify manually in source. |`
> **WRONG:** `| (pricing state) | — | — | (verify curve formula...) |`

**STRICT RULES for tagging (apply ALL that match):**
- Field named `bump` → `[STORED BUMP]`
- Field name contains `admin`, `owner`, `authority`, `signer`, `key` → `[AUTHORITY]`
- Field type is `u64`/`u128`/`i64` and name contains `amount`, `balance`, `total`, `reserve`, `reward`, `fee`, `shares`, `stake`, `deposit` → `[NUMERIC ⚠ overflow]`
- Field type is `i64`/`u64` and name contains `timestamp`, `slot`, `epoch` → `[TIMESTAMP ⚠ manipulation]`
- Field type is `bool` and name contains `paused`, `frozen`, `active`, `enabled` → `[PAUSE FLAG]`
- Field is a `Pubkey` that can be user-controlled (not immutable seed component) → `[PUBKEY ⚠ validation]`
- Field name contains `debt`, `accrued`, `pending`, `claimable` → `[ACCOUNTING ⚠ reset]` (watch for re-stake/re-entry bypasses)

**After every field table:**
1. Write: **"Used by instructions:"** [list 2-3 key instructions]
2. Write: **"Re-init safety:"** [PASS/FAIL] — For each `[NUMERIC]` and `[ACCOUNTING]` field, state whether it is **reset on re-init** or if `init_if_needed` can bypass. MUST verify in instruction body or mark as FAIL.
3. Write: **"Authority chain:"** — Trace who can mutate this struct and via which instructions.

**Example (GOOD):**
```
Struct Name: UserStake

| Field | Type | Tag | Notes |
|---|---|---|---|
| authority | Pubkey | [AUTHORITY] | Set during create, immutable after |
| amount_staked | u64 | [NUMERIC ⚠ overflow] | Checked arithmetic on deposit |
| rewards_debt | u64 | [ACCOUNTING ⚠ reset] | Reset on unstake; risk: init_if_needed could bypass |
| locked_until | u64 | [TIMESTAMP ⚠ manipulation] | Clock-based, not user-supplied |

Used by instructions: stake_liquid, claim_rewards, unstake_liquid

Re-init safety: FAIL — init_if_needed on this account allows re-entry without resetting rewards_debt. Exploit: Call stake → claim → re-stake via same account → rewards counted twice.

Authority chain: user (signer) creates → user can mutate via stake/unstake → operator CANNOT mutate.
```

---

