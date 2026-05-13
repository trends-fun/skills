# 中文常用操作配方

这些配方面向普通用户。回答时先说用户关心的收益、成本、风险和下一步，再给相关命令作为操作依据。命令块用于说明工具调用形状和参数，回答重点放在字段解读和判断依据。

## 1. 安装和准备

安装：

```bash
npm install -g @trends-fun/trends-skill-tool
```

升级：

```bash
npm update -g @trends-fun/trends-skill-tool
```

检查工具可用：

```bash
trends-skill-tool -V
trends-skill-tool --version
trends-skill-tool --help
```

初始化钱包：

```bash
trends-skill-tool wallet init
trends-skill-tool wallet address
```

钱包初始化参数：

```bash
trends-skill-tool wallet init --path ~/.config/solana/id.json
trends-skill-tool wallet init --path ~/.config/solana/iao-agent.json --no-set-default
trends-skill-tool wallet init --force
```

只验证地址，不读取 keypair 文件内容。

全局参数用于限定网络、RPC、钱包和输出格式：

```bash
trends-skill-tool \
  --network devnet \
  --rpc-url https://api.devnet.solana.com \
  --api-base-url https://api.trends.fun/v1 \
  --keypair ~/.config/solana/id.json \
  --commitment confirmed \
  --compute-unit-limit <units> \
  --compute-unit-price <microLamports> \
  --json \
  <command>
```

使用规则：

- devnet 必须显式设置 `--network devnet`，不要只靠 `rpcUrl` 推断。
- `--json` 用于结构化读取；注意 `--json` 数字常是 raw base units，见 decimal 规则。
- `--compute-unit-limit` 和 `--compute-unit-price` 只在需要调整交易计算预算时出现，普通查询不需要。

## 2. 配置

配置文件用于保存默认 RPC、网络、钱包路径等信息。回答配置问题时先说明“当前会影响后续命令的默认上下文”。

```bash
trends-skill-tool config list
trends-skill-tool config get network
trends-skill-tool config get rpcUrl
trends-skill-tool config set network devnet
trends-skill-tool config set rpcUrl https://api.devnet.solana.com
trends-skill-tool config set apiBaseUrl https://api.trends.fun/v1
trends-skill-tool config set keypairPath ~/.config/solana/id.json
trends-skill-tool config set commitment confirmed
trends-skill-tool config set defaultSlippageBps 100
trends-skill-tool config set computeUnitLimit <units>
trends-skill-tool config set computeUnitPriceMicroLamports <microLamports>
trends-skill-tool config reset
```

支持的 key：

- `rpcUrl`
- `apiBaseUrl`
- `network`
- `keypairPath`
- `commitment`
- `defaultSlippageBps`
- `computeUnitLimit`
- `computeUnitPriceMicroLamports`

## 3. 查余额、仓位、发过的币、交易

查当前钱包 SOL 余额：

```bash
trends-skill-tool balance
```

查某地址余额：

```bash
trends-skill-tool balance <address>
```

查某地址的某个 token 余额：

```bash
trends-skill-tool balance <address> --mint <mint>
```

查持仓：

```bash
trends-skill-tool holdings <address> --count 20
```

查自己发过的币：

```bash
trends-skill-tool created <address> --count 20
```

查交易记录：

```bash
trends-skill-tool transactions <address> --count 20
```

分页查询：

```bash
trends-skill-tool created <address> --count 20 --cursor <nextCursor>
trends-skill-tool transactions <address> --count 20 --cursor <nextCursor>
```

输出时必须提醒：

- 同名币很多，判断一个币必须看 mint address。
- 普通输出和 `--json` 输出的数字单位不一样，先判断输出模式。
- `holdings --count` 是每页条数；`created/transactions --count` 范围是 `1..25`。
- 有 `next_cursor`、`nextCursor` 或 `cursor` 时，给出下一页查询的命令形状。

decimal 规则：

| 来源 | 字段 | 用户展示 | 命令输入 |
| --- | --- | --- | --- |
| 普通 `holdings` | `owner_context.balance: '86990635.055642'` | 已经是 token 数量，直接展示 | `sell --in-token 86990635.055642` |
| `holdings --json` | `owner_context.balance: 86990635055642` | raw，要除以 `1,000,000` 得到 `86990635.055642` | `sell --in-token 86990635.055642` |
| 普通 `transactions` | `sol_amount: '1'` | 已经是 SOL，直接展示 | N/A |
| `transactions --json` | `sol_amount: 1000000000` | raw lamports，要除以 `1,000,000,000` 得到 `1` SOL | N/A |
| 普通 `transactions` | `token_amount: '86990635.055642'` | 已经是 token 数量，直接展示 | 可作为参考 |
| `transactions --json` | `token_amount: 86990635055642` | raw，要除以 `1,000,000` 得到 `86990635.055642` | 可作为参考 |

重要约束：

- 不要把普通输出里的 decimal 再除一次；那会把用户仓位缩小 100 万倍。
- 不要把 `--json` 里的 raw 整数直接当 `sell --in-token` 输入；`sell --in-token` 需要人类可读 token 数量。
- `sell --in-token` 最多 6 位小数。遇到更多小数时，先按 6 位 token decimals 截断或四舍五入到 6 位以内，再让用户确认。
- quote / buy / sell 返回里的 `amountIn`、`expectedAmountOut`、`minAmountOut`、fee 字段通常也是 raw base units；SOL 侧除以 `1,000,000,000`，token 侧除以 `1,000,000`，再用中文解释。

## 4. 买币

先试算，不花钱：

```bash
trends-skill-tool quote buy <mint> --in-sol 0.05
trends-skill-tool quote buy <mint> --out-token <tokenAmount>
```

向用户解释：

- `--in-sol` 是“我愿意花多少 SOL”。
- `--out-token` 是“我想拿到多少 token”，属于 exact-out。
- `--in-sol` 和 `--out-token` 必须二选一。
- Raydium CPMM 迁移后 exact-out 可能不可用，遇到 `Raydium CPMM exact-out quotes are not supported yet.` 时改用 `--in-sol`。
- 看 `result` 里的预计输出、滑点、路径和费用。
- 如果试算结果能接受，再确认执行买入。

确认后进入真实买入：

```bash
trends-skill-tool buy <mint> --in-sol 0.05
trends-skill-tool buy <mint> --out-token <tokenAmount>
```

如果用户在 devnet：

```bash
trends-skill-tool --network devnet --rpc-url https://api.devnet.solana.com quote buy <mint> --in-sol 0.05
trends-skill-tool --network devnet --rpc-url https://api.devnet.solana.com buy <mint> --in-sol 0.05
```

不要只靠 `rpcUrl` 判断网络，devnet 必须显式带 `--network devnet` 或已设置同等配置。

滑点参数：

```bash
trends-skill-tool quote buy <mint> --in-sol 0.05 --slippage-bps 100
trends-skill-tool buy <mint> --in-sol 0.05 --slippage-bps 100
```

`--slippage-bps` 可用于 quote/buy/sell；不要用于 `create`。

## 5. 卖币

先试算，不花钱：

```bash
trends-skill-tool quote sell <mint> --in-token 1
trends-skill-tool quote sell <mint> --out-sol <solAmount>
```

确认后进入真实卖出：

```bash
trends-skill-tool sell <mint> --in-token 1
trends-skill-tool sell <mint> --out-sol <solAmount>
```

向用户解释：

- `--in-token` 是“我愿意卖多少 token”。
- `--out-sol` 是“我想拿回多少 SOL”，属于 exact-out。
- `--in-token` 和 `--out-sol` 必须二选一。

如果遇到 Raydium CPMM exact-out 限制，只用 exact-in：

- 买：`--in-sol`
- 卖：`--in-token`

不要继续建议 `--out-token` 或 `--out-sol`。

从持仓卖出时：

- 普通 `holdings` 输出的 `owner_context.balance` 可以直接作为 `--in-token`。
- `holdings --json` 输出的 `owner_context.balance` 要先除以 `1,000,000`，再作为 `--in-token`。
- 如果用户传入超过 6 位小数并报 `Amount precision exceeds 6 digits`，把数量改成最多 6 位小数后重试 quote。

滑点参数：

```bash
trends-skill-tool quote sell <mint> --in-token 1 --slippage-bps 100
trends-skill-tool sell <mint> --in-token 1 --slippage-bps 100
```

## 6. 发币路由：默认引导 IAO / 项目币

用户只说“发币”“发代币”“我想自己多赚一点”时，不要默认走普通 `create`。先引导用户走 IAO / 项目币：

```bash
trends-skill-tool iao project list --count 20
```

向用户解释：

- 普通发币是 Legacy 路径。
- 默认建议先挑一个项目，再围绕项目发币。
- 项目币的分成用“项目方拿多少、你拿多少”来问，不要直接问 `project-submitter-bps`。

只有用户明确说“普通 meme 币”“推特相关代币”“不绑定项目”“不要 IAO”“legacy create”时，才走下面的 Legacy 普通发币流程。

## 7. Legacy 普通发币（仅限明确推特相关代币/普通 meme 币）

发币前必须补全参数，不要直接让用户确认。

必问字段：

- 币名：用户看到的全名。
- 符号：交易界面看到的 ticker。
- 简介：一句话故事。
- 图标：本地图片，或用符号自动生成。
- X 链接：X profile 或 X 推文链接，可留空。
- 首买金额：0 表示只建池，不立刻买。
- 你自己的分成比例：只有填了 X 链接时才需要。

Legacy 最小参数示例：

```bash
trends-skill-tool create --name "My Coin" --symbol "MYC"
```

带简介、X 链接、首买：

```bash
trends-skill-tool create \
  --name "My Coin" \
  --symbol "MYC" \
  --url "https://x.com/example/status/1234567890" \
  --desc "my coin description" \
  --first-buy 0.01 \
  --dev-bps 9600
```

带本地图片：

```bash
trends-skill-tool create --name "My Coin" --symbol "MYC" --image-path ./logo.png
```

重要说明：

- 没有 `--image-path` 时，图标会按 symbol 自动生成。
- `--first-buy 0` 或不填：只创建池，不立刻买。
- `--first-buy` 大于 0：创建和首买一起提交。
- `--dev-bps` 是你自己的分成比例，默认 `9600` 表示你拿 96%，X/推特作者拿 4%。
- 没有 X 链接时，`--dev-bps` 会被忽略，实际是发币人 100%。
- `create` 不支持 `--slippage-bps`。
- 成功后要总结 `mintAddress`、`tokenUrl`、`imageUrl`、`ipfsUri`。

## 8. 领取 creator reward

先查能不能领：

```bash
trends-skill-tool reward status
```

只有同时满足下面条件才让用户确认领取：

- `accountExists=true`
- `rewardLamports>0`

确认后领取：

```bash
trends-skill-tool reward claim
```

如果 `accountExists=false` 或 `rewardLamports=0`，告诉用户现在不可领，等有奖励后重新查。

## 9. 默认发币路径：IAO / 项目币

这是默认发币路径。用户不需要理解 IAO。可以解释为：围绕某个项目发币，项目方和你按比例拿后续收益。

先确认钱包：

```bash
trends-skill-tool wallet address
```

指定 agent 钱包时，全流程保持同一个 `--keypair`：

```bash
trends-skill-tool --keypair ~/.config/solana/iao-agent.json wallet address
```

注册或检查 agent：

```bash
trends-skill-tool iao agent get
trends-skill-tool iao agent get <address>
trends-skill-tool iao agent create --name "My Agent" --avatar-path ./agent.png
```

指定钱包版本：

```bash
trends-skill-tool --keypair ~/.config/solana/iao-agent.json iao agent get
trends-skill-tool --keypair ~/.config/solana/iao-agent.json iao agent create --name "My Agent" --avatar-path ./agent.png
```

agent profile 可选字段：

```bash
trends-skill-tool iao agent create \
  --name "My Agent" \
  --avatar-path ./agent.png \
  --bio "short bio" \
  --introduction "what this agent does" \
  --official-link x=https://x.com/example \
  --official-link telegram=https://t.me/example

trends-skill-tool iao agent update \
  --name "My Agent" \
  --avatar-path ./agent.png \
  --bio "short bio" \
  --introduction "updated intro" \
  --official-link discord=https://discord.gg/example
```

profile 约束：

- `--name` 最长 12 字符。
- `--avatar-path` 是本地图片路径。
- `--bio` 最长 100 字符。
- `--introduction` 最长 1000 字符。
- `--official-link <type=url>` 可重复，type 只支持 `x`、`telegram`、`discord`、`moltbook`。

找项目：

```bash
trends-skill-tool iao project list --count 20
trends-skill-tool iao project list --count 20 --cursor <cursor>
trends-skill-tool iao project list --count 20 --start-at <unix> --end-at <unix>
```

用户要挑项目时，重点展示：

- `items[].url`：发项目币要用的项目链接。
- `title`
- `referenceUrl`
- `submitter`
- `githubRepositoryUrl`
- `officialXUrl`
- `officialCommunityLinks`
- `teamMembers`
- `nextCursor` / `next_cursor` / `cursor`

看单个项目详情：

```bash
trends-skill-tool iao project get <hash>
```

发布项目币前预检：

```bash
trends-skill-tool iao create \
  --project-url <url> \
  --name <name> \
  --symbol <symbol> \
  --description-url <url> \
  --desc <desc> \
  --image-path <path> \
  --first-buy <sol> \
  --project-submitter-bps 7000
```

如果没有 `--image-path`，IAO 会使用项目封面。不要说它会按 symbol 自动生成，也不要先下载项目封面。

IAO 参数说明：

- `--project-url` 必须来自 `iao project list` 的项目 URL，通常是 `https://trends.fun/project/<hash>`。
- `--description-url` 是可选的外部描述链接。
- `--image-path` 可选；省略时使用项目封面。
- `--first-buy 0` 或省略表示不首买；大于 0 表示发布时首买。
- `--project-submitter-bps` 是项目方分成，默认 `7000`，项目方 70%，你 30%，最低 `5000`。
- 如果使用指定 agent 钱包，`wallet address`、`iao agent get/create/update`、`iao project list/get`、`iao create` 全流程都保持同一个 `--keypair`。

成功后总结：

- `mintAddress`
- `tokenUrl`
- `imageUrl`
- `ipfsUri`
- `projectUrl`
- `agentAddress`
