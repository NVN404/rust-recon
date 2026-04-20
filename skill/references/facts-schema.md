## v3 facts.json Field Reference

This reference defines how extracted fields map to recon report sections.

## Instruction-Level Fields

| Field | Source | Meaning | Report Usage |
|---|---|---|---|
| `name` | instruction fn name | Instruction identifier | Section 2 header |
| `context` | Anchor context struct | Account context name | Section 2 header |
| `params[]` | function signature | Caller-provided parameters (`name`, `type`, `overflow_risk`) | Section 2a |
| `accounts[]` | context fields | Account metadata and constraints | Section 2b and 2c |
| `body_checks[]` | `require!` and key equality macros | Runtime checks extracted from body | Section 2d |
| `arithmetic[]` | binary operation extraction | Arithmetic style and expression details | Section 2e |
| `cpi_calls[]` | CPI call extraction | Explicit CPI target/method/signer seeds | Section 5 |
| `events_emitted[]` | `emit!` calls | Instruction event names | Section 1/2 notes |
| `uses_remaining_accounts` | `ctx.remaining_accounts` usage | Indicates dynamic account intake | Section 2f / 7 / 8 checklist |
| `error_codes_referenced[]` | parser cross-reference | Referenced custom errors | Section 6 |

## Account Field Details

| Field | Meaning | Notes |
|---|---|---|
| `wrapper_type` | Anchor wrapper (`Account`, `Signer`, `Program`, `AccountInfo`, `UncheckedAccount`) | Used for `Type`, `Signer`, `Unchecked` columns |
| `inner_type` | Generic inner type when present | Used for readable account typing |
| `unchecked` | Derived bool (`AccountInfo` or `UncheckedAccount`) | Drives Section 2b/2c and checklist generation |
| `has_one[]` | `has_one` relationships extracted from account attrs | Summarized in Section 2b constraint column |
| `close_target` | Account receiving lamports on close | Cross-check in Section 2f recon notes |
| `attributes` | Raw `#[account(...)]` attribute text | Source traceability only; do not dump verbatim in final tables |

## Program-Level Fields

| Field | Meaning | Report Usage |
|---|---|---|
| `data_structs[]` | Extracted account/data structs and fields | Section 3a |
| `errors[]` | Extracted custom errors | Section 6 |
| `flags[]` | Parser-generated signal candidates | Section 7 input |

## Important Rendering Rules

1. `flags[].severity` is parser metadata and is optional context only.
2. Do not render a `Severity` or `Risk` column in Section 2.
3. Section 7 uses the schema from `section-specs.md`:
	- `# | Location | Signal | Evidence | Required Check | Source`
4. If a field is missing, write:
	- `Not extracted - verify manually.`

