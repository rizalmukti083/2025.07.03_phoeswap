# PHNX Presale Implementation Notes

This document lists the required changes to bring the Phoenix presale workflow to a fully functional state.

## 1. Smart contract updates

- Implement token distribution logic in `claim_tokens` inside `presale/programs/presale/src/lib.rs`. The contract should
  - Mint or transfer PHNX tokens to each purchaser according to `amount_spent / price_lamports`.
  - Mark purchase records as claimed to prevent double claiming.
  - Verify `sale.is_active` and time period.
- Add `token_mint` and `sale_token_account` fields to the `Sale` account to track the PHNX mint and the vault that holds presale tokens.
- Update instruction accounts and IDL to include token program accounts required for minting/transfers.
- Ensure `withdraw_sol` only works after `end_time` and that there are no SOL refunds.

## 2. Front‑end pages

- Create a dedicated presale page (e.g. `pages/presale.tsx`) that allows users to:
  - Connect Phantom wallet using `@solana/wallet-adapter-react` and `MyMultiButton`.
  - Display sale parameters (price, caps, start/end time) fetched from the on-chain `Sale` account.
  - Submit SOL via `program.methods.purchase()`.
  - Claim PHNX after the sale using `program.methods.claimTokens()`.
- Update `create-presale.tsx` to include the token mint address and to show meaningful status/errors.
- Expand `manage-presale.tsx` to display sale totals and to disable withdrawals until the sale ends.

## 3. Configuration and deployment

- Provide environment variables in `.env.local` for custom RPC endpoints and program IDs.
- Update `Anchor.toml` with the deployed program ID after running `anchor deploy`.
- Ensure the project uses Anchor CLI `0.30.1` or newer (`avm install 0.30.1`).

## 4. Future presales

- Parameterize sale seeds so multiple `Sale` accounts can coexist, e.g. derive `sale` and `vault` PDA using a unique name passed in `initialize`.
- Store sale details (token mint, price, caps) on chain so that new presales require only calling `initialize` with new parameters.

## 5. Developer notes

- Keep IDL (`src/idl/phoenix_presale.json`) in sync with the on-chain program using `anchor idl fetch` after deployment.
- Use unit tests in `presale/tests/` to cover edge cases (start time checks, hard cap enforcement, token claiming).
- Consider adding CI scripts to run `anchor test` and front‑end type checks.

