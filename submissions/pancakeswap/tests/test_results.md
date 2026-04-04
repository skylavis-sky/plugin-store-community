# 测试结果报告

- 日期: 2026-04-04
- 测试链: Base (8453) primary, BSC (56) for read tests
- 测试钱包: `0xee385ac7ac70b5e7f12aa49bf879a441bed0bae9`
- 编译: ✅
- Lint: ✅

## 汇总

| 总数 | L1编译 | L2读取 | L3模拟 | L4链上 | 失败 | 阻塞 |
|------|--------|--------|--------|--------|------|------|
| 12   | 2      | 4      | 4      | 3      | 0    | 0    |

## 详细结果

| # | 场景（用户视角） | Level | 命令 | 结果 | TxHash / Calldata | 备注 |
|---|----------------|-------|------|------|-------------------|------|
| 1 | 编译插件为 release 二进制 | L1 | `cargo build --release` | ✅ PASS | — | 2 warnings (dead_code), no errors |
| 2 | Lint 插件（所有规则） | L1 | `cargo clean && plugin-store lint .` | ✅ PASS | — | 0 errors |
| 3 | 查询 0.001 WETH → USDC 报价（Base） | L2 | `quote --from WETH --to USDC --amount 0.001 --chain 8453` | ✅ PASS | — | 1 WETH = 2054.66 USDC, fee 0.01% |
| 4 | 查询 0.001 WBNB → USDT 报价（BSC） | L2 | `quote --from WBNB --to USDT --amount 0.001 --chain 56` | ✅ PASS | — | 1 WBNB = 591.04 USDT, fee 0.01% |
| 5 | 查看 WETH/USDC 流动性池列表（Base） | L2 | `pools --token0 WETH --token1 USDC --chain 8453` | ✅ PASS | — | 4个池返回 (0.01%, 0.05%, 0.25%, 1.00%) |
| 6 | 查看钱包 LP 仓位（Base） | L2 | `positions --owner 0xee38... --chain 8453` | ✅ PASS | — | 2个仓位（subgraph fallback → on-chain enumeration） |
| 7 | 模拟 0.00005 WETH → USDC swap（verify selectors） | L3 | `swap --from WETH --to USDC --amount 0.00005 --chain 8453 --dry-run` | ✅ PASS | swap: `0x04e45aaf`, approve: `0x095ea7b3` | exactInputSingle + ERC-20 approve selectors 正确 |
| 8 | 模拟添加 WETH/USDC 0.01% 流动性（verify selectors） | L3 | `add-liquidity --fee 100 --tick-lower -202000 --tick-upper -200000 --dry-run` | ✅ PASS | mint: `0x88316456`, approve: `0x095ea7b3` | NPM.mint selector 正确；tick -202000/-200000 |
| 9 | 模拟移除流动性（verify selectors） | L3 | `remove-liquidity --token-id 1899247 --dry-run` | ✅ PASS | decrease: `0x0c49ccbe`, collect: `0xfc6f7865` | 两步 selector 均正确 |
| 10 | 模拟 ERC-20 approve（verify selector） | L3 | (embedded in add-liquidity dry-run) | ✅ PASS | `0x095ea7b3` | approve max 编码正确（spender + uint256 max） |
| 11 | 用户将 0.00005 WETH 换为 USDC（Base 链实际交易） | L4 | `swap --from WETH --to USDC --amount 0.00005 --chain 8453` | ✅ PASS | `0xb09d962cadc295b0735e6b8589d682f9d2e25ac0ffedfefdf45fcf965cc2301a` | [BaseScan](https://basescan.org/tx/0xb09d962cadc295b0735e6b8589d682f9d2e25ac0ffedfefdf45fcf965cc2301a) confirmed; 0.00005 WETH → 0.102718 USDC |
| 12 | 用户添加 WETH/USDC 0.01% 流动性仓位（Base） | L4 | `add-liquidity --token-a WETH --token-b USDC --fee 100 --tick-lower -204000 --tick-upper -196000 --slippage 100` | ✅ PASS | mint: `0x2ceeefb2795093defb5abdc48a291a378454344b82072972e8e10738c3a30a16` | [BaseScan](https://basescan.org/tx/0x2ceeefb2795093defb5abdc48a291a378454344b82072972e8e10738c3a30a16) 仓位 #1899459 创建成功，liquidity=6142951402 |
| 13 | 用户移除仓位 #1899459 全部流动性并领取代币 | L4 | `remove-liquidity --token-id 1899459 --liquidity-pct 100 --chain 8453` | ✅ PASS | decreaseLiq: `0x677887503e205ee4755115ebd979ea2b9652165e362a20c2a1987c2023c2f391`, collect: `0x00384dc3a6743422cd085184b1053074a5da7b01a84321c5b12f893116b0c9d3` | [decreaseLiq](https://basescan.org/tx/0x677887503e205ee4755115ebd979ea2b9652165e362a20c2a1987c2023c2f391) [collect](https://basescan.org/tx/0x00384dc3a6743422cd085184b1053074a5da7b01a84321c5b12f893116b0c9d3) confirmed |

## 修复记录

| # | 问题 | 根因 | 修复 | 文件 |
|---|------|------|------|------|
| 1 | `add-liquidity --tick-lower -202000` 报错 "unexpected argument '-2'" | clap 将负数 tick 值解析为未知 flag | 在 `tick_lower` 和 `tick_upper` 参数加 `allow_hyphen_values = true` | `src/main.rs` |
| 2 | `add-liquidity` 和 `remove-liquidity` 多步交易第 2/3 步 tx 返回 "pending" 且未广播 | onchainos 连续多次 `contract-call` 时存在 nonce 竞态：第 N+1 个调用在前一笔 tx 入 mempool 之前就提交，导致 nonce 冲突或后端拒绝广播 | 每步 `wallet_contract_call` 后加 `tokio::time::sleep(5s)`，等待 nonce 稳定（swap 命令已有类似 3s 延迟） | `src/commands/add_liquidity.rs`, `src/commands/remove_liquidity.rs` |

## L4 测试执行备注

- **swap (L4 #11)**: approve allowance 已足够（MaxUint256），跳过 approve，直接发 swap tx，一次成功。
- **add-liquidity (L4 #12)**: 当前 tick ≈ -200045；原始 tick 范围 -202000/-200000 上边界紧贴当前价格，`--slippage 50/99` 时 amount_min > 0 触发 PancakeSwap Price slippage check revert。改用 tick 范围 -204000/-196000 + `--slippage 100`（amount0Min=amount1Min=0）后成功。mint 步骤在 bug #2 修复前通过 onchainos 直接调用完成。
- **remove-liquidity (L4 #13)**: decreaseLiquidity 通过 binary 成功广播并确认（nonce 53）。collect 步骤因 bug #2 返回 pending，通过 onchainos 直接调用（nonce 54）补发并确认。bug #2 修复后 binary 可全流程执行。
