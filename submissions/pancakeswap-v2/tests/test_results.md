# Test Results Report — PancakeSwap V2

- **Date:** 2026-04-05
- **Tester:** Phase 3 Tester Agent
- **Plugin:** pancakeswap-v2 v0.1.0
- **Test chains:** BSC (56), Base (8453)
- **Compile:** ✅
- **Lint:** ✅

---

## Summary

| Total | L1 Compile | L2 Read | L3 Simulate | L4 On-chain | Failed | Blocked |
|-------|-----------|---------|------------|------------|--------|---------|
| 17    | 2 ✅       | 8 ✅     | 3 ✅        | 2 ✅ (Base) + 2 ⛔ BLOCKED (BSC) | 0      | 2       |

---

## Detailed Results

| # | Scenario (user view) | Level | Command | Result | TxHash / Calldata | Notes |
|---|---------------------|-------|---------|--------|------------------|-------|
| 1 | Build plugin binary | L1 | `cargo build --release` | ✅ PASS | — | 1 dead_code warning (validate_router, harmless) |
| 2 | Plugin lint check | L1 | `cargo clean && plugin-store lint .` | ✅ PASS | — | 0 errors, all checks passed |
| 3 | "How much USDT for 1 WBNB on PancakeSwap V2?" | L2 | `--chain 56 quote --token-in WBNB --token-out USDT --amount-in 1000000000000000000` | ✅ PASS | — | amountOut=592327660679599984516 (~592 USDT), direct path WBNB→USDT |
| 4 | "Quote WETH→USDC on Base PancakeSwap V2" | L2 | `--chain 8453 quote --token-in WETH --token-out USDC --amount-in 1000000000000000` | ✅ PASS | — | amountOut=2049543 (~2.05 USDC), direct path WETH→USDC |
| 5 | "What is the WBNB/USDT pair address on BSC?" | L2 | `--chain 56 get-pair --token-a WBNB --token-b USDT` | ✅ PASS | — | pair=0x16b9a82891338f9ba80e2d6970fdda79d1eb0dae, exists=true |
| 6 | "What is the WETH/USDC pair address on Base?" | L2 | `--chain 8453 get-pair --token-a WETH --token-b USDC` | ✅ PASS | — | pair=0x79474223aedd0339780bacce75abda0be84dcbf9, exists=true |
| 7 | "What are the reserves in the WBNB/USDT pool on BSC?" | L2 | `--chain 56 get-reserves --token-a WBNB --token-b USDT` | ✅ PASS | — | reserveA=28680 WBNB, reserveB=17.0M USDT, price=593.84 USDT/WBNB |
| 8 | "What are the reserves in the WETH/USDC pool on Base?" | L2 | `--chain 8453 get-reserves --token-a WETH --token-b USDC` | ✅ PASS | — | reserveA=0.174 WETH, reserveB=360 USDC, price=2066 USDC/WETH |
| 9 | "How much LP do I have in WBNB/USDT on BSC?" | L2 | `--chain 56 lp-balance --token-a WBNB --token-b USDT --wallet 0xee385ac7...` | ✅ PASS | — | lpBalance=0 (wallet not LP), totalSupply=272379606984276614941852 |
| 10 | "How much LP do I have in WETH/USDC on Base?" | L2 | `--chain 8453 lp-balance --token-a WETH --token-b USDC --wallet 0xee385ac7...` | ✅ PASS | — | lpBalance=0, totalSupply=5757069332383 |
| 11 | "Simulate swapping WBNB→USDT on PancakeSwap V2" | L3 | `--chain 56 --dry-run swap --token-in WBNB --token-out USDT --amount-in 1000000000000000000` | ✅ PASS | calldata selector: `0x38ed1739` (swapExactTokensForTokens) | dry_run=true, no broadcast, correct selector verified in source |
| 12 | "Preview adding WBNB/USDT liquidity on BSC" | L3 | `--chain 56 --dry-run add-liquidity --token-a WBNB --token-b USDT --amount-a 1000000000000000 --amount-b 500000000000000000` | ✅ PASS | calldata selector: `0xe8e33700` (addLiquidity) | dry_run=true, steps include approve_tokenA, approve_tokenB, addLiquidity |
| 13 | "Preview removing WBNB/USDT liquidity on BSC" | L3 | `--chain 56 --dry-run remove-liquidity --token-a WBNB --token-b USDT --liquidity 1000000000000000` | ✅ PASS | calldata selector: `0xbaa2abde` (removeLiquidity) | dry_run=true, steps include approve_lp, removeLiquidity |
| 14 | "Swap WBNB→USDT with minimum amount on BSC" | L4 | `--chain 56 swap --token-in WBNB --token-out USDT --amount-in <min>` | ⛔ BLOCKED | — | No BSC funds in test wallet (0 BNB, 0 WBNB, 0 USDT on chain 56) |
| 15 | "Swap USDT→WBNB with minimum amount on BSC" | L4 | `--chain 56 swap --token-in USDT --token-out WBNB --amount-in 10000000000000000000` | ⛔ BLOCKED | — | No BSC funds in test wallet |
| 16 | "Swap 0.01 USDC → WETH on Base PancakeSwap V2" | L4 | `--chain 8453 --from 0xee385... swap --token-in USDC --token-out WETH --amount-in 10000` | ✅ PASS | Approve: `0x3eec60959c82f1be38d1ea9ca1c9313e0e6fb6ee6ab563db1c9cb072d2cb9b04` Swap: [`0xaf843e802027e652095a1f84a76e849d447a0de747ffbc6f135fa2ab7ea7a5db`](https://basescan.org/tx/0xaf843e802027e652095a1f84a76e849d447a0de747ffbc6f135fa2ab7ea7a5db) | amountIn=10000 (0.01 USDC), amountOutExpected=4827037013997 (~0.00000000483 WETH), path: USDC→WETH direct, slippage 0.5%, chain 8453 |
| 17 | "LP balance WETH/USDC on Base post-swap" | L4 | `--chain 8453 lp-balance --token-a WETH --token-b USDC --wallet 0xee385...` | ✅ PASS | — (read-only) | lpBalance=0, totalSupply=5757069332383, pair=0x79474223aedd0339780bacce75abda0be84dcbf9, pool functional |

---

## Calldata Selector Verification (L3)

| Operation | Expected Selector | Source Location | Status |
|-----------|------------------|-----------------|--------|
| `swapExactTokensForTokens` | `0x38ed1739` | `src/commands/swap.rs:220` | ✅ Confirmed |
| `addLiquidity` | `0xe8e33700` | `src/commands/add_liquidity.rs:161` | ✅ Confirmed |
| `removeLiquidity` | `0xbaa2abde` | `src/commands/remove_liquidity.rs:173` | ✅ Confirmed |

---

## L4 On-Chain Results (Base, chain 8453)

**Date:** 2026-04-05
**Wallet:** `0xee385ac7ac70b5e7f12aa49bf879a441bed0bae9`
**Pre-swap balance:** 1.245393 USDC, ~0.00425 ETH on Base

### Test 16 — Swap 0.01 USDC → WETH (PASS)

- Command: `./target/release/pancakeswap-v2 --chain 8453 --from 0xee385ac7ac70b5e7f12aa49bf879a441bed0bae9 swap --token-in USDC --token-out WETH --amount-in 10000`
- Step 1 (approve ERC-20): txHash `0x3eec60959c82f1be38d1ea9ca1c9313e0e6fb6ee6ab563db1c9cb072d2cb9b04`
- Step 2 (swapExactTokensForTokens): txHash `0xaf843e802027e652095a1f84a76e849d447a0de747ffbc6f135fa2ab7ea7a5db`
- BaseScan: https://basescan.org/tx/0xaf843e802027e652095a1f84a76e849d447a0de747ffbc6f135fa2ab7ea7a5db
- amountOutExpected: 4827037013997 (~0.00000000483 WETH), slippage 0.5%

### Test 17 — LP Balance WETH/USDC post-swap (PASS)

- Command: `./target/release/pancakeswap-v2 --chain 8453 lp-balance --token-a WETH --token-b USDC --wallet 0xee385ac7ac70b5e7f12aa49bf879a441bed0bae9`
- Result: lpBalance=0, totalSupply=5757069332383, pool at pair `0x79474223aedd0339780bacce75abda0be84dcbf9` functional

---

## Blocked: L4 On-Chain Tests (BSC, chain 56)

**Root Cause:** Test wallet `0xee385ac7ac70b5e7f12aa49bf879a441bed0bae9` has zero balance on BSC (chain 56):
- Native BNB: 0
- WBNB (ERC-20): 0
- USDT (ERC-20): 0

**Resolution Required:** Fund the test wallet on BSC with at least ~0.001 BNB (for gas) and either:
- 0.001 WBNB (to swap WBNB→USDT), or
- 0.01 USDT (to swap USDT→WBNB)

---

## Fix Record

| # | Issue | Root Cause | Fix | File |
|---|-------|-----------|-----|------|
| — | No bugs found | — | — | — |

---

## Notes

- Dead code warning for `validate_router` in `rpc.rs` — harmless, function exists for optional router validation. Not a bug.
- L3 tests confirm the correct PancakeSwap V2 function selectors are used in all write operations.
- BSC RPC endpoint (`bsc-rpc.publicnode.com`) is correctly configured per KNOWLEDGE_HUB guidance.
- `--force` flag is correctly applied in all `wallet_contract_call` invocations (verified in `onchainos.rs`).
- Dry-run is handled in wrapper layer (not passed to onchainos CLI) per known behavior.

---

---

# Test Results Report — PancakeSwap V2 v0.2.0

- **Date:** 2026-04-11
- **Tester:** PR_Claw
- **Plugin:** pancakeswap-v2 v0.2.0
- **Test chains:** BSC (56), Base (8453)
- **Wallet:** `0xee385ac7ac70b5e7f12aa49bf879a441bed0bae9`
- **Compile:** ✅

---

## Context

v0.2.0 fixes two bugs in `remove-liquidity` discovered via user report on 2026-04-10:
1. `lpBalance` showed zero-address garbage in dry-run instead of user's real LP balance.
2. `expectedTokenA/B` overflowed u128 for large pools (BSC BNB/USDT ~$17M TVL), producing garbage estimates.

This test run validates the fixes and completes the first full live-transaction coverage of all commands on both chains.

---

## Summary

| Total | L1/L2 Read | L4 Live TX | Failed | Blocked |
|-------|-----------|------------|--------|---------|
| 12    | 5 ✅       | 7 ✅        | 0      | 0       |

---

## Track A — BSC (chain 56) — Native BNB path

| # | Scenario | Level | Command | Result | TxHash |
|---|----------|-------|---------|--------|--------|
| T1 | Get USDT/BNB pair address | L2 read | `get-pair --token-a USDT --token-b BNB` | ✅ PASS | pair=`0x16b9a82891338f9ba80e2d6970fdda79d1eb0dae` |
| T2 | Get USDT/BNB reserves | L2 read | `get-reserves --token-a USDT --token-b BNB` | ✅ PASS | reserveA=17.2M USDT, reserveB=28.4K BNB |
| T3 | Quote 0.001 BNB → USDT | L2 read | `quote --token-in BNB --token-out USDT --amount-in 1000000000000000` | ✅ PASS | amountOut=604467735315342539 (~0.604 USDT) |
| T4 | LP balance pre-add | L2 read | `lp-balance --token-a USDT --token-b BNB` | ✅ PASS | lpBalance=3978722976785361 (baseline) |
| T5 | Swap 0.001 BNB → USDT | L4 live | `swap --token-in BNB --token-out USDT --amount-in 1000000000000000` | ✅ PASS | `swapExactETHForTokens` [`0xc5407b46204f7f733e0bb58678786d5ba325744f944cf5065dc765e53da75445`](https://bscscan.com/tx/0xc5407b46204f7f733e0bb58678786d5ba325744f944cf5065dc765e53da75445) |
| T6 | Add liquidity 0.5 USDT + 0.000825 BNB | L4 live | `add-liquidity --token-a USDT --token-b BNB --amount-a 500000000000000000 --amount-b 825000000000000` | ✅ PASS | `addLiquidityETH` [`0x2c5016143163c5ef471c2b39631c56493617ca5c02bf2417e3a3505d2b6f8cca`](https://bscscan.com/tx/0x2c5016143163c5ef471c2b39631c56493617ca5c02bf2417e3a3505d2b6f8cca) |
| T7 | LP balance post-add | L2 read | `lp-balance --token-a USDT --token-b BNB` | ✅ PASS | lpBalance=11884600671528879 (up from T4 baseline ✓) |
| T8 | Remove all USDT/BNB liquidity | L4 live | `remove-liquidity --token-a USDT --token-b BNB` | ✅ PASS | `approve_lp` + `removeLiquidityETH` [`0x855ecdc44c7da4da71180fa34379f09d8ff10bd49d1c03fc27024cef9d9861f0`](https://bscscan.com/tx/0x855ecdc44c7da4da71180fa34379f09d8ff10bd49d1c03fc27024cef9d9861f0) |

**T8 regression check (dry-run with `--from` before live removal):**
- `lpBalance: 11884600671528879` ✅ — real wallet balance (was `348500000001000` zero-address garbage before fix)
- `expectedTokenA: 751540052701226112` = 0.7515 USDT ✅ — no overflow (was garbage before fix)
- `expectedTokenB: 1240209035178087` = 0.001240 BNB ✅ — no overflow (was garbage before fix)
- Step: `removeLiquidityETH` ✅ — native BNB path selected correctly

**Post-T8 LP balance:** 0 ✅

---

## Track B — Base (chain 8453) — ERC-20 path

| # | Scenario | Level | Command | Result | TxHash |
|---|----------|-------|---------|--------|--------|
| T9 | Quote 1 USDC → WETH | L2 read | `--chain 8453 quote --token-in USDC --token-out WETH --amount-in 1000000` | ✅ PASS | amountOut=443231548165516 (~0.000443 WETH) |
| T10 | Swap 0.03 USDC → WETH | L4 live | `--chain 8453 swap --token-in USDC --token-out WETH --amount-in 30000` | ✅ PASS | `swapExactTokensForTokens` [`0xdc7abc6edaa8c2137238fba0a8b9fc12fcd05ecf69a6ec1e6ca5c264a6444d07`](https://basescan.org/tx/0xdc7abc6edaa8c2137238fba0a8b9fc12fcd05ecf69a6ec1e6ca5c264a6444d07) |
| T11 | Add liquidity 0.05 USDC + 0.0000223 WETH | L4 live | `--chain 8453 add-liquidity --token-a USDC --token-b WETH --amount-a 50000 --amount-b 22276000000000` | ✅ PASS | `approve_tokenB` + `addLiquidity` [`0x0d7fdb367b832f2dab569b373f7d6b8a369297d0b83d65696efe771e62e32e8e`](https://basescan.org/tx/0x0d7fdb367b832f2dab569b373f7d6b8a369297d0b83d65696efe771e62e32e8e) |
| T12 | Remove 766474186 USDC/WETH LP | L4 live | `--chain 8453 remove-liquidity --token-a USDC --token-b WETH --liquidity 766474186` | ✅ PASS | `approve_lp` + `removeLiquidity` [`0x8ac59f39cc2f3cf39dbc571bc3d8cfb3591ff987b0d34a8cb4f214e54066c31d`](https://basescan.org/tx/0x8ac59f39cc2f3cf39dbc571bc3d8cfb3591ff987b0d34a8cb4f214e54066c31d) |

**T12 output verification:**
- `expectedTokenA: 49999` = 0.049999 USDC ✅ (matches T11 input of 50000 minus fees)
- `expectedTokenB: 22272207889152` ≈ 0.0000223 WETH ✅

**Post-T12 LP balance:** 0 ✅

---

## Code Paths Exercised (v0.2.0, first ever live coverage)

| Selector | Function | Chain | Status |
|----------|----------|-------|--------|
| `0x7ff36ab5` | `swapExactETHForTokens` | BSC | ✅ Live |
| `0x38ed1739` | `swapExactTokensForTokens` | Base | ✅ Live |
| `0xf305d719` | `addLiquidityETH` | BSC | ✅ Live |
| `0xe8e33700` | `addLiquidity` | Base | ✅ Live |
| `0x02751cec` | `removeLiquidityETH` | BSC | ✅ Live |
| `0xbaa2abde` | `removeLiquidity` | Base | ✅ Live |

---

## Fix Record

| # | Issue | Root Cause | Fix | File |
|---|-------|-----------|-----|------|
| 1 | `lpBalance` shows zero-address balance in dry-run | `wallet` always set to `0x0` in dry-run, used for all reads | Resolve real wallet from `--from` / `onchainos::resolve_wallet`; use zero addr only as last resort | `src/commands/remove_liquidity.rs:29-43` |
| 2 | `expectedTokenA/B` garbage for large pools | `reserve * lp_burned` overflows u128 for pools with reserve > ~10²² raw units | `safe_mul_div()`: `checked_mul` with f64 fallback on overflow | `src/commands/remove_liquidity.rs:207-216` |

---

## Notes

- `--from <address>` is required for all write operations in this environment (onchainos wallet auto-resolution not active). All write commands accept `--from`.
- BSC native BNB swap/add/remove paths (`swapExactETHForTokens`, `addLiquidityETH`, `removeLiquidityETH`) confirmed live for the first time in this run.
- `swapExactTokensForETH` (token → native BNB swap) not tested live; selector `0x18cbafe5` was verified in source review.
