## Medium Risk
### Incorrect 1:1 Transfers Without Decimal Normalization Between Tokens
<!--
Number: 4
Cantina code repository status: fixed
Hyperlink: [Issue 4](https://cantina.xyz/code/1a5720f1-29b4-48b2-a47b-b4b1dd57796f/findings/4)
Labels: [None]
Fixed on: [commit [ac0e4878](https://github.com/coinbase/stablecoin-liquidity-audit/pull/1/commits/ac0e48787e908b87e863d098b6d9fd9313e1a4fb)]
-->


**Severity:** Medium Risk

**Context:** [lib.rs#L167-L177](https://cantina.xyz/code/1a5720f1-29b4-48b2-a47b-b4b1dd57796f/programs/scaas-liquidity/src/lib.rs#L167-L177)


**Description:** The protocol currently assumes that all supported tokens share the same decimal configuration and transfers `amount_after_fee` 1:1 between `from_mint `and `to_mint `without normalizing for their respective decimals. If a token is onboarded with a different decimals value than others, the protocol's accounting becomes inconsistent: the same integer amount may represent different real-world quantities of value. This can result in users receiving too much or too little when swapping or redeeming, causing potential loss of user funds or value extraction in misconfigured pools.


**Recommendation:**

- Option A (preferred for flexibility):
Always read decimals from both mints and normalize amounts before slippage checks, liquidity checks, and transfers (e.g., adjust by.
10 to_decimals − from_decimals 10  to_decimals−from_decimals with safe math and clear rounding rules).

- Option B (simpler operationally):
On token onboarding, enforce that all supported mints have identical decimals, rejecting any that differ, and document this invariant clearly in code and docs.

<!--
POTENTIAL RESOLUTION STATEMENT:

**Coinbase:** Fixed in commit [ac0e4878](https://github.com/coinbase/stablecoin-liquidity-audit/pull/1/commits/ac0e48787e908b87e863d098b6d9fd9313e1a4fb).

**Cantina Managed:** Fix verified.

**_COMMENTS_**:

**Coinbase:** Fix in [this commit](https://github.com/coinbase/stablecoin-liquidity-audit/pull/1/commits/ac0e48787e908b87e863d098b6d9fd9313e1a4fb)
-->



### Deppeged tokens can be used to drain the other tokens
<!--
Number: 8
Cantina code repository status: fixed
Hyperlink: [Issue 8](https://cantina.xyz/code/1a5720f1-29b4-48b2-a47b-b4b1dd57796f/findings/8)
Labels: [None]
Fixed on: [commit [de47d28c](https://github.com/coinbase/stablecoin-liquidity-audit/pull/1/commits/de47d28c68eec699d916c2cccc729ffd5152e6d9)]
-->


**Severity:** Medium Risk

**Context:** _(No context files were provided by the reviewer)_


**Description:** Swaps in the liquidity pool are implemented at a hard-coded 1:1 exchange rate between any two supported tokens, with only a fee applied on the input token:

- User provides `amount_in` of `from_mint`.
- The program computes a fee:
  - `fee_amount = amount_in * fee_rate / 10_000`.
- The net amount is:
  - `amount_after_fee = amount_in - fee_amount`.
- `amount_after_fee` of `from_mint` is transferred from the user into the `from_vault`.
- The same `amount_after_fee` of `to_mint` is transferred from the `to_vault` to the user.
- There is no pricing curve or oracle; the swap is always 1:1 on the net amount, regardless of external market prices.

At the same time, there is no mechanism to remove or disable a token once it has been added to `supported_tokens`. As long as a mint remains in the supported set, it can be used in swaps indefinitely.

This combination creates an economical risk: if one supported token depegs (e.g., a token trades under `$1-f`, where `f` is the fee, on external markets), an attacker can:

- Buy the depegged token cheaply on other markets.
- Swap it in this pool at the fixed 1:1 rate for healthy tokens.
- Systematically drain the pool's good assets by repeating this trade.

Because there is no way to delist or isolate the compromised mint, the pool remains fully exposed to this drain scenario until the authority pauses the swaps and program is upgraded to introduce a way to remove that token.



**Recommendation:** Add a mechanism for the authority to remove or disable a token from the pool (e.g., remove it from `supported_tokens` or mark it as inactive). Because the program is upgradable this issue was marked as Medium.

<!--
POTENTIAL RESOLUTION STATEMENT:

**Coinbase:** Fixed in commit [de47d28c](https://github.com/coinbase/stablecoin-liquidity-audit/pull/1/commits/de47d28c68eec699d916c2cccc729ffd5152e6d9).

**Cantina Managed:** Fix verified.

**_COMMENTS_**:

**Coinbase:** Although a user may get a "discount" on the price of custom stablecoin from an arbitrary external market, this would not count as a depegging because the 1:1 exchange ratio is the source of truth and the backing funds for the custom stable would also reflect this.

**Coinbase:** That being said, still going to add a token swap disable functionality which is a reasonable lever to have.

**Coinbase:** Added ability to disable token from swaps in [this commit](https://github.com/coinbase/stablecoin-liquidity-audit/pull/1/commits/de47d28c68eec699d916c2cccc729ffd5152e6d9)
-->



## Informational
### Minor issues and typos
<!--
Number: 1
Cantina code repository status: fixed
Hyperlink: [Issue 1](https://cantina.xyz/code/1a5720f1-29b4-48b2-a47b-b4b1dd57796f/findings/1)
Labels: [None]
Fixed on: [commit [801de38c](https://github.com/coinbase/stablecoin-liquidity-audit/pull/1/commits/801de38c29cb2d8776f1e881f1d0a90c2bab4241)]
-->


**Severity:** Informational

**Context:** [errors.rs#L23-L24](https://cantina.xyz/code/1a5720f1-29b4-48b2-a47b-b4b1dd57796f/programs/scaas-liquidity/src/errors.rs#L23-L24), [errors.rs#L25-L27](https://cantina.xyz/code/1a5720f1-29b4-48b2-a47b-b4b1dd57796f/programs/scaas-liquidity/src/errors.rs#L25-L27)


**Description:**

- [ ] We use `MAX_FEE_RATE` to represent the 10% max fee, but for 100% we use 10000, consider using a constant as well `FEE_DENOMINATOR`.

- [ ] The following errors `InvalidFeeRecipient ("Invalid fee recipient")`, `InvalidVaultAccount ("Invalid vault account")`, and `ReservationOverflow ("Reservation overflow")` are not used.

- [ ] The swap vaults accounts naming `from_vault`, `to_vault` are a bit misleading, especially in the CPI calls `from: ctx.accounts.to_vault_token_account.to_account_info(),`. Consider renaming them to `in_vault`/`out_vault`.
- [ ] The `pool` cached in the vault account is unnecessary, the pool is already in the vault's seed.


**Recommendation:** Consider fixing the issues mentioned above.

<!--
POTENTIAL RESOLUTION STATEMENT:

**Coinbase:** Fixed in commit [801de38c](https://github.com/coinbase/stablecoin-liquidity-audit/pull/1/commits/801de38c29cb2d8776f1e881f1d0a90c2bab4241).

**Cantina Managed:** Fix verified.

**_COMMENTS_**:

**Coinbase:** Fixed in [this commit](https://github.com/coinbase/stablecoin-liquidity-audit/pull/1/commits/801de38c29cb2d8776f1e881f1d0a90c2bab4241)
-->



### Missing Usage and Validation for Persisted Vault Account
<!--
Number: 2
Cantina code repository status: fixed
Hyperlink: [Issue 2](https://cantina.xyz/code/1a5720f1-29b4-48b2-a47b-b4b1dd57796f/findings/2)
Labels: [None]
Fixed on: [commit [801de38c](https://github.com/coinbase/stablecoin-liquidity-audit/pull/1/commits/801de38c29cb2d8776f1e881f1d0a90c2bab4241#diff-0098031e81d4c51cedcd3ac0cd3dd5c7efc023208c0b4baaa9bd42c153356b75)]
-->


**Severity:** Informational

**Context:** [state.rs#L24](https://cantina.xyz/code/1a5720f1-29b4-48b2-a47b-b4b1dd57796f/programs/scaas-liquidity/src/state.rs#L24)


**Description:** The program persists `TokenVault.vault_account` as a Pubkey, but this field is never read or validated against the PDA SPL token account that the program actually uses at runtime. All instructions that operate on the vault token account instead derive and constrain the PDA directly, so changing vault_account does not currently affect behavior or funds; it is effectively unused metadata. However, this mismatch between stored state and actual behavior can mislead off-chain indexers, audits, or future code that assumes `vault_account` reflects the true vault token account.


**Recommendation:** To remediate this, the program should either enforce consistency or remove the field. One option is to keep `vault_account` and add explicit checks in instructions that derive the PDA vault, asserting it equals `TokenVault.vault_account`, and initialize it to the derived PDA when the vault is created. The other option is to remove vault_account entirely if it is not needed, simplifying the account state and avoiding confusion, and to add tests that confirm vault behavior relies solely on the PDA constraints rather than the persisted field.

<!--
POTENTIAL RESOLUTION STATEMENT:

**Coinbase:** Fixed in commit [801de38c](https://github.com/coinbase/stablecoin-liquidity-audit/pull/1/commits/801de38c29cb2d8776f1e881f1d0a90c2bab4241#diff-0098031e81d4c51cedcd3ac0cd3dd5c7efc023208c0b4baaa9bd42c153356b75).

**Cantina Managed:** Fix verified.

**_COMMENTS_**:

**Coinbase:** Removed the field in [this commit](https://github.com/coinbase/stablecoin-liquidity-audit/pull/1/commits/801de38c29cb2d8776f1e881f1d0a90c2bab4241#diff-0098031e81d4c51cedcd3ac0cd3dd5c7efc023208c0b4baaa9bd42c153356b75)
-->



### Unnecessary Mutable Account in swap Context
<!--
Number: 3
Cantina code repository status: fixed
Hyperlink: [Issue 3](https://cantina.xyz/code/1a5720f1-29b4-48b2-a47b-b4b1dd57796f/findings/3)
Labels: [None]
Fixed on: [commit [0b695c94](https://github.com/coinbase/stablecoin-liquidity-audit/pull/1/commits/0b695c949ba54ffc89a7b459a1ff695fa513ecef)]
-->


**Severity:** Informational

**Context:** [lib.rs#L425-L430](https://cantina.xyz/code/1a5720f1-29b4-48b2-a47b-b4b1dd57796f/programs/scaas-liquidity/src/lib.rs#L425-L430)


**Description:** The Swap instruction marks `to_vault: Account<'info, TokenVault> `as `mut`, but the handler only reads from this account (to check reserved_amount) and never modifies it. This causes the runtime to take a write lock on `to_vault` unnecessarily, slightly increasing compute and contention without any corresponding safety benefit. It's not a direct security risk, but it is a correctness/efficiency smell and can marginally increase the program's surface for runtime failures under heavy load.


**Recommendation:** In the Swap accounts struct, remove `mut `from the `to_vault `account declaration so it's read‑only instead of mutable.


<!--
POTENTIAL RESOLUTION STATEMENT:

**Coinbase:** Fixed in commit [0b695c94](https://github.com/coinbase/stablecoin-liquidity-audit/pull/1/commits/0b695c949ba54ffc89a7b459a1ff695fa513ecef).

**Cantina Managed:** Fix verified.

**_COMMENTS_**:

**Coinbase:** Removed `mut` in [this commit](https://github.com/coinbase/stablecoin-liquidity-audit/pull/1/commits/0b695c949ba54ffc89a7b459a1ff695fa513ecef)
-->



### Unrestricted initialize Allows First Caller to Seize Administrative Roles
<!--
Number: 5
Cantina code repository status: acknowledged
Hyperlink: [Issue 5](https://cantina.xyz/code/1a5720f1-29b4-48b2-a47b-b4b1dd57796f/findings/5)
Labels: [None]
Fixed on: [None]
-->


**Severity:** Informational

**Context:** [lib.rs#L26-L28](https://cantina.xyz/code/1a5720f1-29b4-48b2-a47b-b4b1dd57796f/programs/scaas-liquidity/src/lib.rs#L26-L28)


**Description:** The initialize instruction is currently permissionless and allows any caller to set critical administrative authorities for the pool. Because there is no restriction that initialize must be invoked by a known, trusted signer (e.g., deployer or upgrade authority), the first caller can front‑run the intended deployer and set themselves as `operations_authority`, `pause_authority,` and `fee_recipient`. This effectively grants them control over pool operations (including pausing) and fee collection. While the impact may be mitigated in this deployment by the ability to redeploy a single pool, the pattern introduces unnecessary first‑caller risk and weakens the trust model around administrative control.


**Recommendation:** Restrict the initialize instruction so it can only be called by a trusted, predefined authority. Possible fixes:

- Require a specific Signer account with a hard‑coded or configuration‑stored address (e.g., deployer / upgrade authority) via an Anchor address constraint.

- Alternatively, introduce a configuration account (PDA) that stores an admin address and enforce `ctx.accounts.initializer.key() == config.admin` in initialize.

<!--
POTENTIAL RESOLUTION STATEMENT:

**Coinbase:** Acknowledged.

**Cantina Managed:** Acknowledged.

**_COMMENTS_**:

**Coinbase:** This is fine. During the process of deploying the program, we will also make sure that the intended authority initializes the pool. Worst case, the program can always be redeployed in the unlikely chance that some other unintended caller rushed initialized it.
-->



### The `reserved_amount` is obsolete
<!--
Number: 6
Cantina code repository status: fixed
Hyperlink: [Issue 6](https://cantina.xyz/code/1a5720f1-29b4-48b2-a47b-b4b1dd57796f/findings/6)
Labels: [None]
Fixed on: [commit [7f1f9279](https://github.com/coinbase/stablecoin-liquidity-audit/pull/1/commits/7f1f92793548fe4f8c501eec37416e632dcf54d7)]
-->


**Severity:** Informational

**Context:** _(No context files were provided by the reviewer)_


**Description:** From the implementation:

- `TokenVault` has `reserved_amount: u64`.
- In `swap`:
  - `available_liquidity = vault_token_account.amount - to_vault.reserved_amount`.
  - Require `available_liquidity >= amount_after_fee`.
- In `withdraw_liquidity`:
  - `available = vault_token_account.amount - vault.reserved_amount`.
  - Require `amount <= available`.
- In `update_reserved_amount`:
  - Only check `new_reserved_amount <= vault_token_account.amount`.

So this is used for a portion of the vault balance is made unavailable for swaps made by users and withdrawals by the operations authority.

We do not see what is the benefit of this variable, other than maybe stop the swaps to a specific token.


**Recommendation:** If you still want to use this variable more like a swap limit then keeping it for swaps is completely fine.
We recommend deleting it from the withdraw liquidity because this is anyway a limit that can be changed by the same authority that withdraws.

<!--
POTENTIAL RESOLUTION STATEMENT:

**Coinbase:** Fixed in commit [7f1f9279](https://github.com/coinbase/stablecoin-liquidity-audit/pull/1/commits/7f1f92793548fe4f8c501eec37416e632dcf54d7).

**Cantina Managed:** Fix verified.

**_COMMENTS_**:

**Coinbase:** Updated reserved amount to not be checked during liquidity withdraw in [this commit](https://github.com/coinbase/stablecoin-liquidity-audit/pull/1/commits/7f1f92793548fe4f8c501eec37416e632dcf54d7)
-->



### User pays for fee recipient ATA creation
<!--
Number: 7
Cantina code repository status: fixed
Hyperlink: [Issue 7](https://cantina.xyz/code/1a5720f1-29b4-48b2-a47b-b4b1dd57796f/findings/7)
Labels: [None]
Fixed on: [commit [583a9dcd](https://github.com/coinbase/stablecoin-liquidity-audit/pull/1/commits/583a9dcdd6c08b95e17d907cdccf2221eed956a8)]
-->


**Severity:** Informational

**Context:** _(No context files were provided by the reviewer)_


**Description:** At the first swap for a token that was just added, the user has to pay for the ATA creation for the fee recipient:
```rust
#[account(
      init_if_needed,
      payer = user,
      associated_token::mint = from_mint,
      associated_token::authority = fee_recipient
  )]
    pub fee_recipient_token_account: Account<'info, TokenAccount>,
```

The users should never pay for account creation of the fee recipient.


**Recommendation:** Consider creating this ATA when the token is first added in `add_supported_token`.

<!--
POTENTIAL RESOLUTION STATEMENT:

**Coinbase:** Fixed in commit [583a9dcd](https://github.com/coinbase/stablecoin-liquidity-audit/pull/1/commits/583a9dcdd6c08b95e17d907cdccf2221eed956a8).

**Cantina Managed:** Fix verified.

**_COMMENTS_**:

**Coinbase:** Operations authority pays for ATA if needed when adding supported token in [this comment](https://github.com/coinbase/stablecoin-liquidity-audit/pull/1/commits/583a9dcdd6c08b95e17d907cdccf2221eed956a8)
-->



### Fees are rounding down
<!--
Number: 9
Cantina code repository status: fixed
Hyperlink: [Issue 9](https://cantina.xyz/code/1a5720f1-29b4-48b2-a47b-b4b1dd57796f/findings/9)
Labels: [None]
Fixed on: [commit [449bcc24](https://github.com/coinbase/stablecoin-liquidity-audit/pull/1/commits/449bcc24e34257cee054b1e8209085add6064fd6)]
-->


**Severity:** Informational

**Context:** _(No context files were provided by the reviewer)_


**Description:** The fees are rounding down when it should round up:
```rust
let fee_amount = (amount_in as u128)
    .checked_mul(pool.fee_rate as u128)
    .unwrap()
    .checked_div(10000)
    .unwrap() as u64;
```

Theoretically someone can split the swaps in low amounts to avoid paying large fees but in our case as everything is 1:1, the gain is not that much to worth the fee.


**Recommendation:** Consider rounding up the fee.


<!--
POTENTIAL RESOLUTION STATEMENT:

**Coinbase:** Fixed in commit [449bcc24](https://github.com/coinbase/stablecoin-liquidity-audit/pull/1/commits/449bcc24e34257cee054b1e8209085add6064fd6).

**Cantina Managed:** Fix verified.

**_COMMENTS_**:

**Coinbase:** Fees round up in [this commit](https://github.com/coinbase/stablecoin-liquidity-audit/pull/1/commits/449bcc24e34257cee054b1e8209085add6064fd6)

**Cantina Managed:** in the latest version you used 
```rust
let fee_amount = (amount_in as u128)
.checked_mul(pool.fee_rate as u128)
.ok_or(LiquidityError::DecimalNormalizationOverflow)?
.checked_add(FEE_DENOMINATOR as u128 - 1)
.ok_or(LiquidityError::DecimalNormalizationOverflow)?
.checked_div(FEE_DENOMINATOR as u128)
.ok_or(LiquidityError::DecimalNormalizationOverflow)? as u64;
```

this `DecimalNormalizationOverflow` error seems incorrect @saliou-coinbase 

**Coinbase:** tell me more please

**Cantina Managed:** I mean like the error returned in `ok_or` being `DecimalNormalizationOverflow` seems wrong, should be something like `FeeCalculationOverflow` maybe

**Coinbase:** oh got it. will update

**Coinbase:** updated [here](https://github.com/coinbase/stablecoin-liquidity-audit/pull/1/commits/4aaf2d1c85587562e30845f9e78c3d1da4ee78b6) thanks
-->



### Token Accounts are not checked that they correspond to the user/authority
<!--
Number: 10
Cantina code repository status: fixed
Hyperlink: [Issue 10](https://cantina.xyz/code/1a5720f1-29b4-48b2-a47b-b4b1dd57796f/findings/10)
Labels: [None]
Fixed on: [commit [086a31ea](https://github.com/coinbase/stablecoin-liquidity-audit/pull/1/commits/086a31ea268de05ffed853f587380ffec1429dbd)]
-->


**Severity:** Informational

**Context:** _(No context files were provided by the reviewer)_


**Description:** In `swap` and other instructions, SPL token accounts are only constrained by mint, not that their owner is the user, for example:

```rust
#[account(
    mut,
    token::mint = from_mint,
)]
pub user_from_token_account: Account<'info, TokenAccount>,

#[account(
    mut,
    token::mint = to_mint,
)]
pub user_to_token_account: Account<'info, TokenAccount>,
```

Similar patterns appear in other instructions (e.g. deposit/withdraw), where we only check `token::mint = ...` but not `token::authority = user` (or the respective signer / pool / operations authority).

This means the program does not enforce on-chain that these token accounts are actually controlled by the expected wallet.


**Recommendation:** If this is a design choice then nothing should be done, otherwise account constraints should be implemented for every token account.


<!--
POTENTIAL RESOLUTION STATEMENT:

**Coinbase:** Fixed in commit [086a31ea](https://github.com/coinbase/stablecoin-liquidity-audit/pull/1/commits/086a31ea268de05ffed853f587380ffec1429dbd).

**Cantina Managed:** Fix verified.

**_COMMENTS_**:

**Coinbase:** Renamed the var in [this commit](https://github.com/coinbase/stablecoin-liquidity-audit/pull/1/commits/086a31ea268de05ffed853f587380ffec1429dbd)

**Cantina Managed:** Has been fixed by renaming the `user_to_token_account` into `to_token_account`
-->



### Deposit instruction checks can be circumvented
<!--
Number: 11
Cantina code repository status: fixed
Hyperlink: [Issue 11](https://cantina.xyz/code/1a5720f1-29b4-48b2-a47b-b4b1dd57796f/findings/11)
Labels: [None]
Fixed on: [commit [ce442889](https://github.com/coinbase/stablecoin-liquidity-audit/pull/1/commits/ce44288969c71b760fd570f2fc08acbaa2ccc1f1)]
-->


**Severity:** Informational

**Context:** _(No context files were provided by the reviewer)_


**Description:** The deposit instruction does not provide control over how liquidity enters the vaults because the actual vault balance can be increased at any time by the operations authority (or anyone) by calling the SPL Token program directly to transfer into `vault_token_account`, completely bypassing the `pool.liquidity_paused` check.



**Recommendation:** We do not recommend any action regarding this issue other than documenting the fact that deposit paused check can be circumvented.

<!--
POTENTIAL RESOLUTION STATEMENT:

**Coinbase:** Fixed in commit [ce442889](https://github.com/coinbase/stablecoin-liquidity-audit/pull/1/commits/ce44288969c71b760fd570f2fc08acbaa2ccc1f1).

**Cantina Managed:** Fix verified.

**_COMMENTS_**:

**Coinbase:** The operations authority would likely largely use the deposit instruction from the program to deposit liquidity, but if they decide to perform a direct transfer then that is also fine.
Still a good callout so I added a comment in the code to mention this so that we know to consider it when needed.

**Coinbase:** Added note in [this commit](https://github.com/coinbase/stablecoin-liquidity-audit/pull/1/commits/ce44288969c71b760fd570f2fc08acbaa2ccc1f1)
-->

