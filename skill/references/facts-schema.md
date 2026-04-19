## v2 `facts.json` Field Reference

The skill now consumes these fields per instruction. Know what each means before using it:

| Field | Source | Meaning |
|---|---|---|
| `params[]` | function signature | Caller-supplied arguments (name + type + overflow_risk flag) |
| `accounts[].wrapper_type` | context struct field type | `Account`, `Signer`, `UncheckedAccount`, `AccountInfo`, `Program`, etc. |
| `accounts[].inner_type` | generic arg of wrapper | The data struct type (e.g. `LiquidityPool`) or `null` |
| `accounts[].unchecked` | derived | `true` if wrapper is `UncheckedAccount` or `AccountInfo` |
| `accounts[].has_one` | `has_one =` in macro | List of field names that must match on the account struct |
| `accounts[].close_target` | `close =` in macro | Account that receives lamports on close, or `null` |
| `accounts[].attributes` | raw macro string | Verbatim `#[account(...)]` string |
| `body_checks[]` | `require!` / `require_keys_eq!` etc. | Runtime invariant checks inside function body |
| `arithmetic[]` | binary ops in body | Each op: `operation`, `style` (checked/unchecked), `expression`, `overflow_risk` |
| `events_emitted[]` | `emit!()` calls | Event struct names emitted by this instruction |
| `uses_remaining_accounts` | `ctx.remaining_accounts` | Boolean — unvalidated account injection surface |
| `cpi_calls[]` | CPI invocations | `program`, `method`, `signer_seeds` |
| `flags[]` | top-level aggregated | Pre-computed issues from the extractor — import directly into Section 7 |

---

