# Error Playbook

Use exact message matching first. Provide deterministic fixes before broad diagnosis.

For wallet-bound failures (especially `iao`), keep the same keypair context as the failing run by reusing `--keypair <path>` in fix and verification commands.

## Message-to-action map

| Error text (exact or pattern) | Probable cause | Immediate fix command(s) | Verification command |
| --- | --- | --- | --- |
| `Invalid mint address format` | Mint argument is malformed or not a valid Solana public key | Replace `<mint>` with a valid mint address and rerun quote first. Example: `trends-skill-tool quote buy <mint> --in-sol 0.1` | `trends-skill-tool quote buy <mint> --in-sol 0.1` |
| `Invalid address format` or `Invalid address format. Please check mint or address arguments.` | Public key parsing failed for address or mint argument (domain validation error or normalized unexpected error) | Re-check address/mint source and rerun with corrected value | `trends-skill-tool balance <address>` or `trends-skill-tool quote buy <mint> --in-sol 0.1` |
| `Network request failed. Please check RPC/API endpoints and connectivity.` | RPC/API endpoint unavailable, DNS/timeout/connectivity issue | Set known-good endpoints and retry. Example: `trends-skill-tool config set rpcUrl https://api.mainnet-beta.solana.com` and `trends-skill-tool config set apiBaseUrl https://api.trends.fun/v1` | `trends-skill-tool quote buy <mint> --in-sol 0.1` |
| `{\"status\":\"error\",\"error_code\":4055,\"error_msg\":\"User already exists\"}` | Wallet is already registered on trends.fun for this user path and cannot be reused for `iao agent create` | Create a fresh wallet and rerun with explicit keypair. Example: `trends-skill-tool wallet init --path ~/.config/solana/iao-agent.json` then `trends-skill-tool --keypair ~/.config/solana/iao-agent.json iao agent create --name \"My Agent\" --avatar-path ./agent.png` | `trends-skill-tool --keypair ~/.config/solana/iao-agent.json iao agent get` |
| `{\"status\":\"error\",\"error_code\":4030,\"error_msg\":\"Operation not permitted\"}` | Current wallet is not an eligible/registered agent account for this IAO operation | Verify same keypair context and register/switch account. Example: `trends-skill-tool --keypair <path> wallet address`, `trends-skill-tool --keypair <path> iao agent get`, then `trends-skill-tool --keypair <path> iao agent create --name \"My Agent\" --avatar-path ./agent.png` if needed | `trends-skill-tool --keypair <path> iao agent get` |
| `Current wallet address is not registered as an agent user. Run \`iao agent create\` first.` | `iao create` was run by a wallet that is not registered as an agent user | Register the same wallet first or switch to the correct agent wallet. Example: `trends-skill-tool --keypair <path> iao agent create --name \"My Agent\" --avatar-path ./agent.png` | `trends-skill-tool --keypair <path> iao agent get` |
| `iao create requires a valid project URL` or `iao create requires a project URL in /project/<hash> format` | Invalid `--project-url` input | Select a valid URL from `iao project list`, then rerun `iao create` with that URL | `trends-skill-tool iao project list --count 5` |
| `--project-submitter-bps must be a non-negative integer` | Invalid strict integer value for `--project-submitter-bps` (text suffix, decimal, negative, scientific notation) | Re-enter plain integer digits only | Re-run same `iao create` command with corrected `--project-submitter-bps` |
| `--project-submitter-bps must be >= 5000 for iao create` | `--project-submitter-bps` below minimum | Use `--project-submitter-bps` in `5000..10000` or omit to use default `7000` | Re-run same `iao create` command |
| `iao project list requires --end-at when --start-at is provided.` | `--start-at` passed without `--end-at` | Provide both values together or remove time filtering | `trends-skill-tool iao project list --start-at <ts> --end-at <ts>` |
| `Project submitter is required for iao reward distribution` | Selected project payload does not provide submitter info needed for IAO reward recipients | Select a different project that includes submitter profile details | `trends-skill-tool iao project list --count 20` |
| `Project submitter must include an x profile or github profile for iao create` | Project submitter exists but has no usable `x`/`github` profile id | Select a different project with valid submitter social profile | `trends-skill-tool iao project list --count 20` |
| `Avatar file does not exist: <path>` | Invalid or missing local path for `--avatar-path` | Fix the local file path and rerun `iao agent create`/`iao agent update` | `trends-skill-tool iao agent create --name \"My Agent\" --avatar-path <existing-local-path>` |
| `Image file does not exist: <path>` | Invalid local path for `--image-path` in `iao create` | Use an existing local image path or remove `--image-path` to auto-generate by symbol | `trends-skill-tool iao create --project-url <url> --name \"My Coin\" --symbol \"MYC\"` |
| `Auto image generation failed. Please provide --image-path.` | Auto-generated image pipeline failed when `--image-path` was omitted | Provide a local `--image-path` explicitly and retry | `trends-skill-tool iao create --project-url <url> --name \"My Coin\" --symbol \"MYC\" --image-path ./logo.png` |
| `Insufficient SOL balance. Current:  <value>, required at least: <value> (including first buy and base fee reserve)` | Wallet lacks enough SOL for publish reserve plus optional first buy | Use wallet with higher SOL balance or reduce `--first-buy` | `trends-skill-tool --keypair <path> balance` |
| `--official-link must use type=url format` | `--official-link` not in `type=url` shape | Rewrite each `--official-link` argument as `type=url` and retry | `trends-skill-tool iao agent create --name \"My Agent\" --avatar-path ./agent.png --official-link x=https://x.com/example` |
| `--official-link url cannot be empty` | Missing URL portion after `=` in `--official-link` | Supply non-empty URL | `trends-skill-tool iao agent update --name \"My Agent\" --avatar-path ./agent.png --official-link x=https://x.com/example` |
| `--official-link type must be one of x | telegram | discord | moltbook` | Unsupported official link type | Use only supported types | `trends-skill-tool iao agent create --name \"My Agent\" --avatar-path ./agent.png --official-link telegram=https://t.me/example` |
| `No claimable coin creator rewards found because reward account is not initialized` | Reward PDA for current wallet has not been created yet | Query status first and stop claim. Example: `trends-skill-tool reward status` | `trends-skill-tool reward status` (expect `accountExists=false`) |
| `No claimable coin creator rewards found because reward is 0` | Reward account exists but accumulated reward is zero | Query status and wait for reward accrual before retrying claim | `trends-skill-tool reward status` (expect `rewardLamports>0` before claim) |
| `error: unknown option '--slippage-bps'` (under `trends-skill-tool create`) | `create` no longer supports slippage flag | Remove `--slippage-bps` from create command and retry | `trends-skill-tool create --name \"My Coin\" --symbol \"MYC\" --first-buy 0.01` |
| `--<option> must be a non-negative integer` | Integer option received invalid format (`1e2`, decimals, text suffix, negative) | Re-enter as plain integer digits. Example: `--slippage-bps 100`, `--count 20` | Re-run the same command with corrected integer values |
| `--count must be <= 25` | `created` or `transactions` page size out of range | Use `--count` between `1` and `25` | `trends-skill-tool created <address> --count 20` |
| `--count must be >= 1` | `--count` is zero or negative | Use `--count` at least `1` | `trends-skill-tool holdings <address> --count 20` |
| `buy requires exactly one of --in-sol or --out-token.` | Both route flags were provided or neither was provided | Keep exactly one route flag | `trends-skill-tool buy <mint> --in-sol 0.01` |
| `sell requires exactly one of --in-token or --out-sol.` | Both route flags were provided or neither was provided | Keep exactly one route flag | `trends-skill-tool sell <mint> --in-token 1` |
| `quote buy requires exactly one of --in-sol or --out-token.` | Both route flags were provided or neither was provided | Keep exactly one route flag | `trends-skill-tool quote buy <mint> --in-sol 0.1` |
| `quote sell requires exactly one of --in-token or --out-sol.` | Both route flags were provided or neither was provided | Keep exactly one route flag | `trends-skill-tool quote sell <mint> --in-token 1` |
| `Key file does not exist: <path>` | `keypairPath` points to a missing file | Initialize wallet or set a valid keypair path. Example: `trends-skill-tool wallet init` then `trends-skill-tool config set keypairPath ~/.config/solana/id.json` | `trends-skill-tool wallet address` |
| `Target file already exists: <path>, use --force to overwrite` | `wallet init` target exists and overwrite flag missing | Re-run with `--force` if overwrite is intended | `trends-skill-tool wallet init --force` then `trends-skill-tool wallet address` |
| `Unsupported config key: <key>` | `config set/get` key is not allowed | Use supported keys only (`rpcUrl`, `apiBaseUrl`, `keypairPath`, `commitment`, `defaultSlippageBps`, `computeUnitLimit`, `computeUnitPriceMicroLamports`) | `trends-skill-tool config list` |
| `commitment only supports processed | confirmed | finalized` | Invalid commitment value | Set one of supported values only | `trends-skill-tool config set commitment confirmed` and `trends-skill-tool config get commitment` |

## Fallback diagnosis when no exact match exists

Run in this order:

```bash
trends-skill-tool --version
trends-skill-tool --help
trends-skill-tool config list
trends-skill-tool wallet address
```

Then rerun the failing command with explicit values for endpoints and route flags.

Example:

```bash
trends-skill-tool quote buy <mint> --in-sol 0.1 --rpc-url https://api.mainnet-beta.solana.com --api-base-url https://api.trends.fun/v1
```
