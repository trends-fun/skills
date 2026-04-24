name: trends
description: Help users install, configure, use, and troubleshoot @trends-fun/trends-skill-tool, including IAO agent workflows. Use this skill whenever the user mentions trends or trends-skill commands (create, buy, sell, quote, reward, wallet, config, holdings, created, transactions, iao agent/project get/project list/create), asks how to install or upgrade this specific CLI, asks about agent wallet registration, or reports errors from this CLI. Prefer triggering even if the user says "trends command" or "trends tool" without the full package name
---

# Trends CLI User Guide

## Scope

- This skill is for end users operating the installed `trends-skill-tool` command.
- Installation channel is global npm only.

## IAO Model

- IAO means **Initial (Attention) Agent Offering**: a model where agents issue initial attention tokens for projects.
- For all IAO tasks, use `references/iao-model.md` as the primary behavior source.
- Tokens created via `trends-skill-tool iao create` are still standard Trends tokens; original commands such as `created`, `quote buy|sell`, `buy`, `sell`, `holdings`, and `transactions` remain valid for them.

## Trigger guardrails

- **Primary Intent Trigger**: Trigger whenever the user expresses intent related to:
  - **Launching**: "Launch a token", "create a memecoin", "launch on trends.fun", "deploy a coin".
  - **Earnings**: "Check my coin creator earnings", "how much have I earned?", "claim my rewards".
  - **Launch History**: "What tokens have I launched?", "my launch history", "show my created coins".
  - **Portfolio**: "What tokens do I hold?", "my holdings", "show my balance".
  - **Transactions**: "What transactions did I make?", "my transaction history", "recent trades".
- **Trading/Quotes**: "Get a swap quote on trends.fun", "swap on trends.fun", "buy/sell on trends".
- **IAO Agent Flow**: "register agent wallet", "iao agent", "iao project get", "iao project list", "iao create", "project-url", "description-url", "project-submitter-bps".
- **Command-Based Trigger**: Trigger for explicit commands: `create`, `buy`, `sell`, `quote`, `reward`, `wallet`, `config`, `balance`, `holdings`, `created`, `transactions`, `iao`, `iao agent get`, `iao agent create`, `iao agent update`, `iao project get`, `iao project list`, `iao create`.
- **Package-Based Trigger**: Trigger when user mentions `@trends-fun/trends-skill-tool`, "trends command", or "trends tool".
- **Lifecycle Trigger**: Trigger when user asks to install, upgrade, or uninstall this CLI.
- **Error Trigger**: Trigger when user shares errors containing "trends-skill", IAO backend JSON errors (for example `error_code: 4055` / `4030`), raw SDK migration errors such as `PoolMigrationPending: migrate the pool before trading.` / `Raydium CPMM exact-out quotes are not supported yet.`, or related bonding curve failures.
  
## Source-of-truth order

1. `references/install-and-setup.md`
2. `references/command-recipes.md`
3. `references/iao-model.md`
4. `references/iao-cron.md`
5. `references/error-playbook.md`

## Workflow

1. Identify the user intent:

- Install/setup
- Config/wallet
- Read-only query
- Trade flow
- IAO agent flow
- Troubleshooting

1. Pick the matching recipe and provide runnable commands.
2. State expected observable outcomes.
3. Provide one concrete fallback path tied to known CLI behavior.
4. Recommend the next action only after the current step is verifiable.

## Write Confirmation Gate (Trade + IAO Publish)

The following commands are write operations and require a confirmation gate by default:

- `trends-skill-tool create`
- `trends-skill-tool buy`
- `trends-skill-tool sell`
- `trends-skill-tool reward claim`
- `trends-skill-tool iao create`

Non-gated commands (normal flow):

- `trends-skill-tool balance`
- `trends-skill-tool holdings`
- `trends-skill-tool created`
- `trends-skill-tool transactions`
- `trends-skill-tool quote buy`
- `trends-skill-tool quote sell`
- `trends-skill-tool reward status`
- `trends-skill-tool wallet init`
- `trends-skill-tool wallet address`
- `trends-skill-tool config list|get|set|reset`
- `trends-skill-tool iao agent get`
- `trends-skill-tool iao agent create`
- `trends-skill-tool iao agent update`
- `trends-skill-tool iao project get`
- `trends-skill-tool iao project list`
- install/upgrade/uninstall commands

Use a staged flow for gated write commands:

- Phase 0 (parameter completion): collect missing parameters first. If anything is unresolved, ask focused follow-up questions and stop. Do not ask execution confirmation yet.
- Phase A (not confirmed): preflight analysis + fully resolved parameter echo. Do not provide final executable write steps.
- Phase B (confirmed): provide executable write steps.

For `create` and `iao create`, Phase 0 is mandatory unless the user already provided all required fields or explicitly accepted defaults for missing fields.

Valid confirmation phrases include:

- `confirm`
- `yes proceed`
- `确认执行`
- `直接执行`

Bypass exception:

- If the user explicitly requests direct execution (`direct write`, `直接执行`, `不要确认`, `skip confirmation`), you may bypass the confirmation gate.
- Default bypass scope is current request only.
- If the user explicitly states session-level preference, apply bypass for the session.
- Even with bypass, still perform preflight disclosure for `buy`/`sell` quotes and `reward claim` status.
- Bypass does not skip Phase 0 parameter completion for missing required/optional create fields.
- Bypass does not skip Phase 0 parameter completion for missing required/optional `iao create` fields.

If confirmation is unclear:

- Ask for explicit confirmation and do not move to Phase B.

## Write Intent Preflight Contracts

Before any gated write command, always provide a preflight summary.

For `create` preflight, always echo:

- `name`
- `symbol`
- `url`
- `desc`
- `image source`
- `first-buy` (initial buy amount at create time)
- `dev-bps` (token deployer share / 代币部署者分成) and split explanation

For `iao create` preflight, always echo:

- current keypair/address context (which wallet will execute the command)
- `project-url`
- `name`
- `symbol`
- `description-url`
- `desc`
- `image source`
- `first-buy`
- `project-submitter-bps`
- reward split summary:
  - project submitter share = `project-submitter-bps` (default `7000`)
  - agent coin creator share = `10000 - project-submitter-bps`

For `create`, treat users as beginners and explain fields in plain language:

- `name`: token name users will see.
- `symbol`: token ticker symbol users will see in trading views.
- `desc`: short reason/story for why the token exists.
- `image`: token icon source. If `--image-path` is missing, icon is auto-generated from symbol.
- `url`: related X profile or X tweet URL (`x.com`/`twitter.com`, profile or `/status/...`).
- `first-buy`: first buy amount (SOL) executed right after create.
- `dev-bps`: token deployer share (代币部署者分成, the user share). X creator share is `10000-dev-bps` when `url` exists.

`create` parameter completion contract (must happen before confirmation):

- Do not ask `confirm` / `yes proceed` / `确认执行` until each field is resolved.
- A field is "resolved" only when:
  - user provided a value, or
  - user explicitly accepted the default/empty behavior.
- Resolve in this order:
  1. `name` and `symbol` (required, cannot be empty)
  2. `desc` (user text or explicit empty)
  3. `image source` (local `--image-path` or explicit auto-generate)
  4. `url` (valid X profile/tweet URL or explicit none)
  5. `first-buy` (explicit number or explicit `0`)
  6. `dev-bps` (required decision only when `url` exists; otherwise mark ignored)
- If `url` exists and `dev-bps` not provided, ask whether to use default `9600`.
- If `url` does not exist, explicitly state `dev-bps` is ignored and effective split is token deployer `100%`, X `0%`.
- If responding in Chinese during missing-parameter collection, use exact wording for URL prompt:
  - `请输入你想指向的费用受益人X profile 或推文链接（可留空）`
- If responding in Chinese, label `dev-bps` as:
  - `代币部署者分成(dev-bps)`
- During Phase 0, ask only missing-field questions plus default choices; do not ask execution confirmation.

`create` preflight rules:

- If `--image-path` is not provided, explicitly state image will be auto-generated from `symbol`.
- If `first-buy` is omitted or `0`, explicitly state: create/pool initialization still happens, but no immediate first buy is executed.
- If `first-buy` is greater than `0`, explicitly state: create and first buy are submitted atomically in one flow.
- If `url` is provided, show split between token deployer and X creator:
  - token deployer share = `dev-bps` (default `9600` if omitted)
  - X creator share = `10000 - dev-bps`
- If `url` is not provided, explicitly state:
  - `--dev-bps` is ignored
  - effective split is token deployer `100%`, X creator `0%`

For `buy`/`sell` preflight, always:

- Run quote first (`trends-skill-tool quote buy` or `trends-skill-tool quote sell`)
- Echo route, input/output amount, slippage bps, and key quote fields
- If the user is on devnet, include `--network devnet` and a devnet RPC explicitly in both quote and execution commands; do not infer SDK network from `rpcUrl`
- If the user reports `PoolMigrationPending: migrate the pool before trading.`, stop the trade flow and explain that quoting/trading is unavailable until pool migration completes
- If the user reports `Raydium CPMM exact-out quotes are not supported yet.`, keep the original SDK wording and switch guidance to exact-in routes only:
  - buy / quote buy: use `--in-sol`
  - sell / quote sell: use `--in-token`
- State that `buy`/`sell` will run only after confirmation

For `reward claim` preflight, always:

- Run `trends-skill-tool reward status` first
- Echo `accountExists`, `rewardLamports`, `rewardSol`
- Only proceed to claim confirmation when `accountExists=true` and `rewardLamports>0`
- If not claimable, stop and provide retry condition

For `iao` workflow:

- `iao agent create` and `iao agent update` are non-gated profile writes; once required parameters are complete, provide runnable commands directly.
- `iao project get` is read-only and can be used to inspect one specific project by hash.
- `iao create` is a gated write; resolve missing parameters and show the full preflight before asking confirmation.
- For scheduled discovery behavior, follow `references/iao-cron.md`.
- Use `references/iao-model.md` as the primary workflow and constraint source for IAO operations.

For `holdings` / `created` / `transactions` responses, always include identifiers:

- Always echo queried wallet `address`.
- For each token row, include token mint address (prefer keys in order: `mintAddress`, `mint_addr`, `mint`).
- Never summarize by `name`/`symbol` only; disambiguate duplicate symbols with mint address.
- Include `next_cursor` for `created` and `transactions` when present.
- If a desired field is unavailable in raw data, explicitly output `N/A` and include the raw key name you used.
- For `transactions`, include transaction identifier/signature/hash field when present.
- For numeric fields in `holdings` / `created`:
  - `owner_context.balance` is raw `decimals=6`
  - human-readable value = raw value / `1_000_000`
- For numeric fields in `transactions`:
  - `sol_amount` is raw lamports (`decimals=9`), human-readable value = raw value / `1_000_000_000`
  - `token_amount` is raw token base units (`decimals=6`), human-readable value = raw value / `1_000_000`
- Always disclose conversions in the response and never present unconverted raw numbers as final user-facing amounts.

## Secret Handling Policy (Hard Block)

Never request, read, print, or echo any of the following:

- Private keys
- Mnemonic/seed phrases
- `secretKey` arrays
- Raw keypair file contents

Never suggest commands that expose private key content, including:

- `cat ~/.config/solana/id.json`
- Any equivalent command that prints key material

If user provides private key material:

- Refuse to continue handling sensitive key content
- Instruct user to rotate/revoke compromised keys
- Continue only with address-level operations, such as `trends-skill-tool wallet address`

## Mandatory response template

Use one of the following templates based on command type and confirmation state.

For read-only or non-gated tasks:

### Situation

Summarize the user goal in one or two lines.

### Commands to run

Provide copy-paste command blocks only with supported `trends-skill-tool` flags and subcommands.

### Expected result

State what the user should see (key fields, success lines, or command behavior).

### If it fails

Map to an exact error from `references/error-playbook.md`, then provide immediate fix commands.

### Next step

Give the shortest validated next command or decision.

For gated write tasks before confirmation:

### Situation

Summarize the user goal in one or two lines.

### Preflight

Provide required preflight checks and parameter echo (including quote/status where required).

### Pending write plan

Show what will be executed after confirmation. Do not provide final executable write steps yet.

### Confirm needed

Ask for explicit confirmation (`confirm` / `yes proceed` / `确认执行` / `直接执行`) only after Phase 0 parameter completion is finished.

### If it fails

Map to exact known errors and immediate fixes.

### Next step

State the shortest path to confirm and execute.

For gated write tasks after confirmation:

- Use the standard read-only template and provide executable steps.

## Command policy

- Never invent commands or flags.
- Use only the supported command surface:
  - `trends-skill-tool create`
  - `trends-skill-tool buy`
  - `trends-skill-tool sell`
  - `trends-skill-tool quote buy`
  - `trends-skill-tool quote sell`
  - `trends-skill-tool balance`
  - `trends-skill-tool holdings`
  - `trends-skill-tool created`
  - `trends-skill-tool transactions`
  - `trends-skill-tool reward status`
  - `trends-skill-tool reward claim`
  - `trends-skill-tool iao agent get`
  - `trends-skill-tool iao agent create`
  - `trends-skill-tool iao agent update`
  - `trends-skill-tool iao project get`
  - `trends-skill-tool iao project list`
  - `trends-skill-tool iao create`
  - `trends-skill-tool wallet init`
  - `trends-skill-tool wallet address`
  - `trends-skill-tool config list|get|set|reset`
- Use supported global options when needed:
  - `--rpc-url <url>`
  - `--api-base-url <url>`
  - `--network <mainnet-beta|devnet>`
  - `--keypair <path>`
  - `--commitment <processed|confirmed|finalized>`
  - `--compute-unit-limit <units>`
  - `--compute-unit-price <microLamports>`
  - `--json`
- Enforce mutually exclusive amount routes:
  - `buy`: exactly one of `--in-sol` or `--out-token`
  - `sell`: exactly one of `--in-token` or `--out-sol`
  - `quote buy`: exactly one of `--in-sol` or `--out-token`
  - `quote sell`: exactly one of `--in-token` or `--out-sol`
- Respect integer constraints:
  - Strict non-negative integers for options such as `--slippage-bps`, `--dev-bps`, `--count`, `--compute-unit-limit`, `--compute-unit-price`
  - IAO strict integers include `--project-submitter-bps`, `--count`, `--start-at`, `--end-at`
  - `created/transactions --count` range is `1..25`
  - `holdings --count` is `>= 1`
- Create-specific constraints:
  - Do not suggest `--slippage-bps` for `trends-skill-tool create`.
  - `--first-buy` means initial buy amount at create time:
    - omitted or `0`: create only (no initial buy execution)
    - greater than `0`: create + first buy submitted atomically
- Network/config constraints:
  - SDK network must be set explicitly via `--network`, `TRENDS_NETWORK`, or config key `network`; do not infer it from `rpcUrl`
  - Supported config keys are `rpcUrl`, `apiBaseUrl`, `network`, `keypairPath`, `commitment`, `defaultSlippageBps`, `computeUnitLimit`, `computeUnitPriceMicroLamports`
  - Default network is `mainnet-beta`
  - If the user is operating on devnet, prefer explicit commands such as `trends-skill-tool --network devnet --rpc-url https://api.devnet.solana.com ...`
- Migration-specific trade constraints:
  - `PoolMigrationPending: migrate the pool before trading.` means `buy`, `sell`, `quote buy`, and `quote sell` are temporarily unavailable for that pool
  - `Raydium CPMM exact-out quotes are not supported yet.` means exact-out routes are unavailable after migration:
    - buy / quote buy: do not suggest `--out-token`; use `--in-sol`
    - sell / quote sell: do not suggest `--out-sol`; use `--in-token`
- Reward-specific constraints:
  - `trends-skill-tool reward status` is read-only and should be the first check before claim.
  - `trends-skill-tool reward claim` should be suggested only when reward account exists and reward is greater than `0`.
  - If reward account is missing or reward is `0`, explain that claim returns validation error and no transaction is sent.

## Output policy

- Keep steps concise and operational.
- Prefer `--json` when the user requests parsing, automation, or script integration.
- Include concrete verification commands (`trends-skill-tool --version`, `trends-skill-tool --help`, targeted `quote`/query checks) when relevant.

## Troubleshooting policy

- First, match exact known errors from `references/error-playbook.md`.
- Provide deterministic fix commands before broad diagnosis.
- If no exact match exists, run a minimal diagnosis set in this order:
  1. `trends-skill-tool --version`
  2. `trends-skill-tool --help`
  3. `trends-skill-tool config list`
  4. `trends-skill-tool wallet address`
  5. Re-run the failing command with explicit endpoints / network or `--json` as needed
- Do not provide speculative root causes without a validation step.

## Reference routing

- For install, upgrade, uninstall, wallet bootstrap, config precedence: read `references/install-and-setup.md`.
- For normal task execution patterns: read `references/command-recipes.md`.
- For IAO agent registration, project selection, publish flow, and IAO-specific errors: read `references/iao-model.md`.
- For scheduled discovery and cron-runbook behavior: read `references/iao-cron.md`.
- For error-to-fix mappings and verification commands: read `references/error-playbook.md`.
