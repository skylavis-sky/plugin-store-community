# Morpho — Test Results

- Date: 2026-04-04
- Test chain: Base (8453) + Ethereum (1)
- Wallet: `0xee385ac7ac70b5e7f12aa49bf879a441bed0bae9`
- Vault: `0xbeeF010f9cb27031ad51e3333f9aF9C6B1228183` (Steakhouse USDC, Base)
- USDC: `0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913`
- Compile: PASS
- Lint: PASS

## Summary

| Total | L1 Compile | L2 Read | L3 Simulate | L4 On-chain | Fail | Blocked |
|-------|------------|---------|-------------|-------------|------|---------|
| 12    | 2          | 4       | 3           | 3           | 0    | 0       |

## Detailed Results

| # | Scenario (user intent) | Level | Command | Result | TxHash / Calldata | Notes |
|---|------------------------|-------|---------|--------|-------------------|-------|
| TC-1 | Plugin compiles cleanly | L1 | `cargo build --release` | PASS | — | Warnings only, no errors |
| TC-2 | Plugin passes lint checks | L1 | `plugin-store lint .` | PASS | — | 0 lint errors |
| TC-3 | User wants to see lending markets on Ethereum | L2 | `morpho --chain 1 markets` | PASS | — | 9 markets returned |
| TC-4 | User wants to see lending markets on Base | L2 | `morpho --chain 8453 markets` | PASS | — | 50 markets returned |
| TC-5 | User wants to browse yield vaults on Base | L2 | `morpho --chain 8453 vaults` | PASS | — | 50 vaults returned |
| TC-6 | User wants to view their Morpho positions | L2 | `morpho --chain 8453 positions` | PASS | — | Empty positions (no active Blue positions) |
| TC-7 | User wants to supply USDC to steakUSDC (simulate only) | L3 | `morpho --chain 8453 --dry-run supply --vault 0xbeeF010f9cb27031ad51e3333f9aF9C6B1228183 --asset 0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913 --amount 0.01` | PASS | approve: `0x095ea7b3...`, deposit: `0x6e553f65...` | ERC-20 approve + ERC-4626 deposit calldata verified |
| TC-8 | User wants to withdraw USDC from steakUSDC (simulate only) | L3 | `morpho --chain 8453 --dry-run withdraw --vault 0xbeeF010f9cb27031ad51e3333f9aF9C6B1228183 --asset 0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913 --amount 0.01` | PASS | `0xb460af94...` | ERC-4626 withdraw(uint256,address,address) calldata verified |
| TC-9 | User wants to borrow against collateral (simulate only) | L3 | `morpho --chain 8453 --dry-run borrow ...` | PASS | `0x50d8cd4b...` | Morpho Blue borrow calldata verified (dry-run only per GUARDRAILS.md) |
| TC-10 | User supplies 0.01 USDC to steakUSDC vault — approve step | L4 | `morpho --chain 8453 supply --vault 0xbeeF010f9cb27031ad51e3333f9aF9C6B1228183 --asset 0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913 --amount 0.01` | PASS | `0xcca4c8fa412d9866d2707a53797789541d9e0dd7edafc647ec2d8092fb3a084f` | ERC-20 approve confirmed on Base |
| TC-11 | User supplies 0.01 USDC to steakUSDC vault — deposit step | L4 | (same command as TC-10, step 2/2) | PASS | `0xb4cb33545165beb9a42cf671dab4bdc16d72dd1d00f8da968b317f9c83d41ec1` | ERC-4626 deposit confirmed on Base — 3s delay fix resolved nonce conflict |
| TC-12 | User withdraws 0.01 USDC from steakUSDC vault | L4 | `morpho --chain 8453 withdraw --vault 0xbeeF010f9cb27031ad51e3333f9aF9C6B1228183 --asset 0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913 --amount 0.01` | PASS | `0x4f4a0c016435172db2ed6c2d0bb8d19631b5280da263dfdf95327fd05bff0dec` | ERC-4626 withdraw confirmed on Base |

## Fix Log

| # | Issue | Root Cause | Fix | File |
|---|-------|------------|-----|------|
| 1 | `--dry-run` flag passed to `onchainos wallet contract-call` which doesn't support it | Plugin was passing `--dry-run` to onchainos, but that CLI flag doesn't exist on `wallet contract-call` | Removed `args.push("--dry-run")` and added early-return with simulated response when `dry_run=true` | `src/onchainos.rs` |
| 2 | `markets --chain 8453` returned 0 markets | GraphQL `orderBy: TotalSupplyUsd` enum value does not exist in Morpho API schema | Removed `orderBy` and `orderDirection` from markets query | `src/api.rs` |
| 3 | `vaults --chain 8453` returned only 1 test vault | Same GraphQL issue: `orderBy: TotalAssetsUsd` enum does not exist | Removed `orderBy` and `orderDirection` from vaults query | `src/api.rs` |
| 4 | `markets` and `vaults` deserializing 0 items despite valid API responses | API returns numeric fields (`supplyAssets`, `borrowAssets`, etc.) as JSON numbers, but Rust structs expected `Option<String>` — `serde` drops items that fail deserialization | Added `deser_number_or_string` custom deserializer; applied to `MarketState`, `PositionState`, `VaultState`, `VaultPosition` numeric fields | `src/api.rs` |
| 5 | `positions --chain 8453` returned GraphQL error on `healthFactor` field | `healthFactor` is not a field on `MarketPositionState` in the Morpho API schema (confirmed via introspection) | Removed `healthFactor` from GraphQL query and from `PositionState` struct; removed `health_factor_status` function | `src/api.rs`, `src/commands/positions.rs` |
| 6 | `supply` and `withdraw` sent funds to/from address(0) when `--from` not provided | `from.unwrap_or("0x0000...0000")` was used as receiver/owner — silently lost funds to zero address | Added `resolve_wallet(from, chain_id)` in `onchainos.rs` that queries active wallet via `onchainos wallet balance`; used in `supply.rs`, `withdraw.rs`, and `positions.rs` | `src/onchainos.rs`, `src/commands/supply.rs`, `src/commands/withdraw.rs`, `src/commands/positions.rs` |
| 7 | Token symbol shows "UNKNOWN" for USDC on Base | `mainnet.base.org` rate-limits aggressively causing `erc20_symbol()` RPC calls to fail | Changed Base RPC URL to `https://base-rpc.publicnode.com` (reliable public endpoint) | `src/config.rs` |
| 8 | `wallet_contract_call()` returned `txHash: "pending"` instead of broadcasting on live runs | `--force` flag missing from `onchainos wallet contract-call` args — required for all on-chain write operations | Added `args.push("--force")` before `Command::new("onchainos")` call (after dry-run early-return) | `src/onchainos.rs` |
| 9 | Deposit tx dropped with "replacement transaction underpriced" (nonce conflict) | Approve and deposit were submitted too close together — onchainos queued both at the same nonce before approve was confirmed | Added `tokio::time::sleep(Duration::from_secs(3))` between approve and deposit calls (same pattern as PancakeSwap swap.rs) | `src/commands/supply.rs` |
