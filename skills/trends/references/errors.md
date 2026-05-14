# 错误处理表

处理报错时，先匹配原始错误文本，再给解释和下一步处理依据。可以给相关命令，但重点放在原因、判断条件和恢复路径。不要猜测根因。

## 买卖和池子状态

| 原始错误 | 中文解释 | 下一步 |
| --- | --- | --- |
| `Invalid mint address format` | mint 地址格式不对，可能录入错了。 | 用正确 mint 做 quote 校验。 |
| `Invalid address format` | 钱包地址或 mint 地址格式不对。 | 核对地址来源，再用对应查询或 quote 做验证。 |
| `PoolMigrationPending: migrate the pool before trading.` | 这个池子正在迁移，暂时不能 quote、买、卖。换路线没用。 | 等迁移完成后再跑 `quote buy` 或 `quote sell`。 |
| `Raydium CPMM exact-out quotes are not supported yet.` | 迁移到 Raydium CPMM 后，不支持“指定最终拿多少”的 exact-out 试算。 | 买用 `--in-sol`，卖用 `--in-token`。 |
| `buy requires exactly one of --in-sol or --out-token.` | 买入金额参数二选一，不能都填，也不能都不填。 | 用 `trends-skill-tool quote buy <mint> --in-sol 0.1`。 |
| `sell requires exactly one of --in-token or --out-sol.` | 卖出金额参数二选一，不能都填，也不能都不填。 | 用 `trends-skill-tool quote sell <mint> --in-token 1`。 |
| `error: unknown option '--in-token'` under `buy` | `buy` 没有 `--in-token`，买入 exact-in 用 SOL。 | 用 `trends-skill-tool quote buy <mint> --in-sol <sol>`；Raydium CPMM 下不要改用 `--out-token`。 |
| `Amount precision exceeds 6 digits: <value>` | token 输入小数超过 6 位，或把 decimal/raw 处理错了。 | 如果来自普通 `holdings`，直接用其 `owner_context.balance`；如果来自 `holdings --json`，先除以 `1,000,000`，再保留最多 6 位小数。 |
| `bigint: Failed to load bindings, pure JS will be used` | 这是 bigint native binding 加载失败后的 JS fallback 警告，不等于命令失败。 | 只要后面显示 `succeeded` 或 JSON `ok: true`，继续按结果处理；需要优化环境时再重装/rebuild。 |

## 钱包、网络、配置

| 原始错误 | 中文解释 | 下一步 |
| --- | --- | --- |
| `Network request failed. Please check RPC/API endpoints and connectivity.` | RPC 或 Trends API 连不上。 | 设置稳定 endpoint 后重试。 |
| `network only supports mainnet-beta | devnet` | 网络只能是 `mainnet-beta` 或 `devnet`。 | `trends-skill-tool config set network mainnet-beta` 或 `devnet`。 |
| `Key file does not exist: <path>` | 配置里的 keypair 文件路径不存在。 | 初始化钱包或改成存在的路径。 |
| `Target file already exists: <path>, use --force to overwrite` | 钱包文件已存在，默认不会覆盖。 | 只有明确要覆盖时才用 `trends-skill-tool wallet init --force`。 |
| `Unsupported config key: <key>` | 配置项名称不支持。 | 用 `trends-skill-tool config list` 查看支持项。 |

基础诊断：

```bash
trends-skill-tool --version
trends-skill-tool --help
trends-skill-tool config list
trends-skill-tool wallet address
```

devnet 必须同时有网络和 RPC：

```bash
trends-skill-tool --network devnet --rpc-url https://api.devnet.solana.com quote buy <mint> --in-sol 0.1
```

## 发币和奖励

| 原始错误 | 中文解释 | 下一步 |
| --- | --- | --- |
| `error: unknown option '--slippage-bps'` | `create` 不支持滑点参数。 | 删除 `--slippage-bps` 后重试。 |
| `No claimable coin creator rewards found because reward account is not initialized` | 奖励账户还没初始化，现在没法领。 | 用 `trends-skill-tool reward status` 看 `accountExists`，等它变成 `true`。 |
| `No claimable coin creator rewards found because reward is 0` | 有奖励账户，但现在可领金额为 0。 | 等 `rewardLamports>0` 后再 claim。 |
| `Insufficient SOL balance. Current:  <value>, required at least: <value> (including first buy and base fee reserve)` | 钱包 SOL 不够支付发币基础费用和首买。 | 充值 SOL，或降低 `--first-buy`。 |

## IAO / 项目币

| 原始错误 | 中文解释 | 下一步 |
| --- | --- | --- |
| `{"status":"error","error_code":4055,"error_msg":"User already exists"}` | 这个钱包已经是 trends.fun 用户，不能走当前 agent 创建路径。 | 新建一个专用 agent 钱包，再用同一个 `--keypair` 注册。 |
| `{"status":"error","error_code":4030,"error_msg":"Operation not permitted"}` | 当前钱包不是可用的 agent 钱包，或和你要操作的钱包不一致。 | 用同一个 keypair 查地址和 agent 状态。 |
| `Current wallet address is not registered as an agent user. Run \`iao agent create\` first.` | 当前钱包还没注册 agent。 | 先补 agent 注册信息，再进入 IAO 发布。 |
| `iao create requires a valid project URL` | 项目链接无效。 | 使用 `iao project list` 返回的 `items[].url`。 |
| `iao create requires a project URL in /project/<hash> format` | 项目链接必须是 `/project/<hash>` 格式。 | 使用列表结果里的完整项目 URL。 |
| `--project-submitter-bps must be a non-negative integer` | 项目方分成参数必须是整数。 | 用纯数字，比如 `7000`。 |
| `--project-submitter-bps must be >= 5000 for iao create` | 项目方最低拿 50%。 | 填 `5000..10000`，或省略用默认 `7000`。 |
| `iao create requires a project cover image when --image-path is omitted` | 选中的项目没有封面，且你没提供本地图片。 | 换有封面的项目，或加 `--image-path ./logo.png`。 |

4055 修复参考：

```bash
trends-skill-tool wallet init --path ~/.config/solana/iao-agent.json
trends-skill-tool --keypair ~/.config/solana/iao-agent.json wallet address
trends-skill-tool --keypair ~/.config/solana/iao-agent.json iao agent create --name "My Agent" --avatar-path ./agent.png
```

4030 验证参考：

```bash
trends-skill-tool --keypair <path> wallet address
trends-skill-tool --keypair <path> iao agent get
```
