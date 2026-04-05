# Flap Meme Token Launchpad Plugin

Meme token launchpad on BSC via Flap Protocol ([flap.sh](https://flap.sh)). Create standard or tax tokens with bonding curves, buy/sell on the bonding curve, and track token status.

## Commands

- `get-token-info` — Query bonding curve state: status, price, reserve, tax rates, graduation progress
- `create-token` — Launch a new standard ERC-20 or tax token; grinds CREATE2 salt for vanity address suffix (8888 / 7777)
- `buy` — Buy tokens with BNB via bonding curve
- `sell` — Sell tokens for BNB; automatically handles ERC-20 approve before selling

## Chain

BSC Mainnet (chain 56). Portal proxy: `0xe2cE6ab80874Fa9Fa2aAE65D277Dd6B8e65C9De0`

## Usage

```bash
# Query token info
flap get-token-info --token 0xAbCd...

# Preview creating a token (always dry-run first)
flap create-token --name "Moon Hamster" --symbol "MHAMS" --dry-run

# Preview buying 0.1 BNB worth of tokens
flap buy --token 0xAbCd... --bnb-amount 100000000000000000 --dry-run

# Preview selling tokens
flap sell --token 0xAbCd... --token-amount 1000000000000000000000 --dry-run
```

All write commands (`create-token`, `buy`, `sell`) require explicit user confirmation before executing on-chain. Always run with `--dry-run` first.
