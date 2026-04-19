#### 5b. CPI Call Map (ASCII Art)

Show which instructions invoke CPIs and their targets.

Example structure:

```
Instruction        │ Target Program      │ Method      │ Signer
─────────────────┼────────────────────┼─────────────┼────────
create_liquidity │ System Program     │ create_acct │ admin
stake            │ Token Program      │ transfer_to │ user
swap             │ Token Program      │ transfer    │ user
migrate          │ Metadata Program   │ create_mdata│ executor
```

After each block:
- Non-token/non-system program → flag arbitrary CPI risk.
- `signer_seeds` references mutable field → flag seeds may be manipulated before signing.
- `uses_remaining_accounts == true` → flag CPI receives unvalidated accounts.

**🔴 CPI EXTRACTION LIMITATION (CRITICAL):**
> If `cpi_calls[]` is empty or incomplete, **DO NOT assume zero CPIs exist.** 
> The rust-recon extractor may miss CPIs inside:
> - Conditional branches (`if` / `match` statements)
> - Loops with dynamic calls
> - Indirect CPI invocations through helper functions
>
> **For instructions that clearly perform token transfers but CPIs are missing/incomplete:**
> Use the "IMPLICIT OPERATIONS" pattern in Section 5b to name suspected methods explicitly.
> 
> **Method Naming Rule:**
> - Instead of writing: `| buy_tokens | implicit SPL | ... |`
> - Write: `| buy_tokens | token::transfer (or token::transfer_checked) | user_ata → curve_ata | User burns input |`
> 
> **Rationale:** "implicit SPL" is vague and unhelpful for auditors. Name the specific method being called based on the account types involved (ATAs = transfer, token program CPIs = transfer_checked, etc.).
> 
> If multiple methods are possible (e.g., transfer OR transfer_checked), list both: `token::transfer or token::transfer_checked`
>
> **Audit Note to add to Section 5b if CPIs are missing:**
> ```
> ⚠ **Known Limitation:** rust-recon extracted [N] CPI calls, but instructions with token transfer logic
> (e.g., rebalance_pools, swap, claim_rewards) may invoke transfer_checked or transfer inside conditional 
> branches not yet captured by the parser. 
> 
> **Manual verification required:**
> - Search source for token::transfer and token_2022::transfer_checked
> - Verify signer_seeds do not reference mutable fields
> - Confirm all CPIs have correct PDA derivations
> ```

#### 5c. Token-2022

If `token_2022` or `spl_token_2022` appears in any `cpi_calls[].program` or account name:
- List every extension type referenced.
- Flag: `transfer_checked` required instead of `transfer`.
- Flag: fee-on-transfer extension means amount_received ≠ amount_sent — verify accounting.

---

