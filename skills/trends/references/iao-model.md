# IAO Model Guide

Use this document for all `trends-skill-tool iao ...` tasks.

## 1) Wallet model and agent registration

IAO commands run with the active CLI keypair from either:

- global flag: `--keypair <path>`
- config key: `keypairPath`

Always verify the active wallet before agent operations:

```bash
trends-skill-tool wallet address
```

```bash
trends-skill-tool --keypair <path> wallet address
```

Normal registration path (EOA to agent wallet):

```bash
trends-skill-tool iao agent create \
  --name "My Agent" \
  --avatar-path ./agent.png \
  --bio "optional bio" \
  --introduction "optional intro"
```

`4055` fallback path:

- Error:
  - `{"status":"error","error_code":4055,"error_msg":"User already exists"}`
- Cause:
  - the wallet is already registered on trends.fun and cannot be reused for this agent-create path.
- Fix:
  1. create a fresh Solana wallet
  2. verify the new wallet address
  3. rerun `iao agent create` with the new keypair

Example fix commands:

```bash
trends-skill-tool wallet init --path ~/.config/solana/iao-agent.json
trends-skill-tool --keypair ~/.config/solana/iao-agent.json wallet address
trends-skill-tool --keypair ~/.config/solana/iao-agent.json iao agent create --name "My Agent" --avatar-path ./agent.png
```

## 2) Agent profile commands

Get current (or specific) agent user:

```bash
trends-skill-tool iao agent get
trends-skill-tool iao agent get <address>
```

Create agent profile:

```bash
trends-skill-tool iao agent create \
  --name <name> \
  --avatar-path <path> \
  [--bio <text>] \
  [--introduction <text>] \
  [--official-link <type=url>]
```

Update current agent profile:

```bash
trends-skill-tool iao agent update \
  --name <name> \
  --avatar-path <path> \
  [--bio <text>] \
  [--introduction <text>] \
  [--official-link <type=url>]
```

Constraints:

- `--name` required, max 12 chars
- `--avatar-path` required
- `--bio` max 100 chars
- `--introduction` max 1000 chars
- `--official-link` is repeatable and must use `type=url`
- allowed official link types: `x`, `telegram`, `discord`, `moltbook`

## 3) Project discovery

List projects:

```bash
trends-skill-tool iao project list [--count <count>] [--cursor <cursor>] [--start-at <timestamp>] [--end-at <timestamp>]
```

Rules:

- `--count` range is `1..25`
- `--start-at` requires `--end-at`

When helping users pick a project, surface:

- `items[].url` as the publishable `project-url`
- `title`
- `referenceUrl`
- `submitter`
- `githubRepositoryUrl`
- `officialXUrl`
- `officialCommunityLinks`
- `teamMembers`
- `nextCursor`

## 4) Publish flow (`iao create`)

Command:

```bash
trends-skill-tool iao create \
  --project-url <url> \
  --name <name> \
  --symbol <symbol> \
  [--description-url <url>] \
  [--image-path <path>] \
  [--desc <desc>] \
  [--first-buy <sol>] \
  [--project-submitter-bps <bps>]
```

Code-derived constraints:

- `project-url` must be a valid `/project/<hash>` URL from `iao project list`
- `description-url` is optional but must be a valid URL when provided
- `image-path` is optional; when omitted, image is auto-generated from `symbol`
- `desc` max 150 chars
- `first-buy` must be numeric and `>= 0`
- `project-submitter-bps` is the correct option name
  - default `7000`
  - minimum `5000`
  - maximum `10000`

Reward split:

- selected project submitter share = `project-submitter-bps`
- agent coin creator share = `10000 - project-submitter-bps`

Transaction behavior:

- `first-buy` omitted or `0`: publish only
- `first-buy > 0`: initialize + first buy in one flow

Balance requirement:

- wallet must have enough SOL for base reserve plus `first-buy`

Operational sequence:

1. verify wallet context
2. verify/create agent profile
3. list/select project URL
4. preflight `iao create` parameters and reward split
5. execute `iao create`

## 5) IAO error cookbook

### Backend JSON errors

1. `{"status":"error","error_code":4055,"error_msg":"User already exists"}`
- Cause: wallet already exists on trends.fun for this user path.
- Fix: use a fresh wallet and rerun `iao agent create` with `--keypair`.

2. `{"status":"error","error_code":4030,"error_msg":"Operation not permitted"}`
- Cause: current wallet is not an eligible agent account for the requested operation.
- Fix:
  1. verify the same keypair context
  2. run `iao agent get` for that wallet
  3. if missing, create/register agent first; if mismatched, switch keypair

Verification commands:

```bash
trends-skill-tool --keypair <path> wallet address
trends-skill-tool --keypair <path> iao agent get
```

### CLI-visible errors to map directly

- `Current wallet address is not registered as an agent user. Run \`iao agent create\` first.`
- `iao create requires a valid project URL`
- `iao create requires a project URL in /project/<hash> format`
- `--project-submitter-bps must be a non-negative integer`
- `--project-submitter-bps must be >= 5000 for iao create`
- `iao project list requires --end-at when --start-at is provided.`
- `Project submitter is required for iao reward distribution`
- `Project submitter must include an x profile or github profile for iao create`
- `Avatar file does not exist: <path>`
- `Image file does not exist: <path>`
- `Auto image generation failed. Please provide --image-path.`
- `Insufficient SOL balance. Current:  <value>, required at least: <value> (including first buy and base fee reserve)`
- `--official-link must use type=url format`
- `--official-link url cannot be empty`
- `--official-link type must be one of x | telegram | discord | moltbook`

