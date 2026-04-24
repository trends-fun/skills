# Install and Setup

## Prerequisites

- Node.js `>= 20`
- Network access to:
  - Solana RPC endpoint (default: `https://api.mainnet-beta.solana.com`)
  - Trends API endpoint (default: `https://api.trends.fun/v1`)
- SDK network defaults to `mainnet-beta`; use `--network devnet` or `config set network devnet` explicitly when operating on devnet

## Global install lifecycle

Install:

```bash
npm install -g @trends-fun/trends-skill-tool
```

Upgrade:

```bash
npm update -g @trends-fun/trends-skill-tool
```

Uninstall:

```bash
npm uninstall -g @trends-fun/trends-skill-tool
```

## Sanity checks

```bash
trends-skill-tool --version
trends-skill-tool --help
```

If either command fails, reinstall globally and verify `node -v` is `>= 20`.

## Wallet bootstrap

Initialize wallet:

```bash
trends-skill-tool wallet init
```

Initialize with custom path:

```bash
trends-skill-tool wallet init --path ~/.config/solana/id.json
```

Force overwrite when key file exists:

```bash
trends-skill-tool wallet init --force
```

Show current wallet address:

```bash
trends-skill-tool wallet address
```

Notes:

- Default wallet path is `~/.config/solana/id.json`.
- By default, `wallet init` updates `keypairPath` in local trends config.
- Use `--no-set-default` if you do not want to write `keypairPath` into trends config.
- Never display, export, or paste private key / seed phrase / `secretKey` content.
- Use `trends-skill-tool wallet address` for address-only verification.
- If keypair path needs confirmation, show only the path string; do not read key file contents.

## Configuration model

Priority (high to low):

1. CLI arguments
2. Environment variables
3. Local config file
4. Built-in defaults

Config file path:

- `~/.config/trends-skill/config.json`

Supported config keys:

- `rpcUrl`
- `apiBaseUrl`
- `network`
- `keypairPath`
- `commitment`
- `defaultSlippageBps`
- `computeUnitLimit`
- `computeUnitPriceMicroLamports`

Common config commands:

```bash
trends-skill-tool config list
trends-skill-tool config get network
trends-skill-tool config get rpcUrl
trends-skill-tool config set network devnet
trends-skill-tool config set rpcUrl https://api.mainnet-beta.solana.com
trends-skill-tool config set commitment finalized
trends-skill-tool config reset
```

Network notes:

- `network` only accepts `mainnet-beta` or `devnet`
- CLI option `--network`, environment variable `TRENDS_NETWORK`, and config key `network` share the same priority model as other config values
- `rpcUrl` does not auto-select the SDK network; devnet requires both a devnet RPC and `--network devnet` (or equivalent config/env)

Environment variables:

- `TRENDS_RPC_URL`
- `TRENDS_API_BASE_URL`
- `TRENDS_NETWORK`
- `TRENDS_KEYPAIR_PATH`
- `TRENDS_COMMITMENT`

Defaults:

- `rpcUrl`: `https://api.mainnet-beta.solana.com`
- `apiBaseUrl`: `https://api.trends.fun/v1`
- `network`: `mainnet-beta`
- `keypairPath`: `~/.config/solana/id.json`
- `commitment`: `confirmed`
- `defaultSlippageBps`: `100`

Devnet quote example:

```bash
trends-skill-tool --network devnet --rpc-url https://api.devnet.solana.com quote buy <mint> --in-sol 0.1
```

## Minimal ready-state checklist

1. `trends-skill-tool --version` prints version.
2. `trends-skill-tool --help` prints command list.
3. `trends-skill-tool reward --help` prints reward command group.
4. `trends-skill-tool wallet address` prints a valid address.
5. `trends-skill-tool config list` shows effective local config file content.
