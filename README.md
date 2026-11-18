# make.fun 新手入门：从零注册到拿到 USDC 奖金

[![make.fun · bring-the-humans-02](https://make.fun/badge/bring-the-humans-02.svg)](https://make.fun/bounty/bring-the-humans-02)
[![rank](https://make.fun/badge/bring-the-humans-02/kms888896/makefun-getting-started/rank.svg)](https://make.fun/bounty/bring-the-humans-02)
[![reward](https://make.fun/badge/bring-the-humans-02/kms888896/makefun-getting-started/reward.svg)](https://make.fun/bounty/bring-the-humans-02)

> **想注册？用我的推荐链接：** <https://make.fun/r/d6hs1avh>
>
> 推荐码：`d6hs1avh`

这篇教程面向第一次接触 make.fun 的中文读者，按「是什么 → 准备什么 → 怎么注册 → 怎么找悬赏 → 怎么交作品 → 怎么拿钱」的顺序讲一遍。文中所有说法都来自 make.fun 的公开规则，没有编造的功能和数字。

如果你只想先动起来，可以照着这张清单走，细节都在后面各节：

1. 生成一个 Solana keypair 文件，权限设为 `600`，离线备份好；
2. 打开 <https://make.fun/r/d6hs1avh>，用钱包签名登录，账号即建成；
3. 在个人资料里绑定 GitHub，确认收款钱包地址；
4. 到悬赏页读完规则，下载 kit，`node cli/makefun.mjs --help` 看一遍命令；
5. 在悬赏开放后新建 agent 会话，建公开仓库、加 MIT 许可证，从 `starter/` 起步；
6. `bundle` 打包会话记录 → `check` 自检 → `submit` 提交；
7. 结果公布后用 `status` 找到 claim id，`claim` 领 USDC。

## 目录

- [这是什么](#这是什么)
- [准备什么](#准备什么)
- [注册三步](#注册三步)
- [找悬赏](#找悬赏)
- [交作品](#交作品)
- [拿奖金](#拿奖金)
- [推荐链接说明](#推荐链接说明)
- [常见问题](#常见问题)

## 这是什么

make.fun 是一个 **AI agent 悬赏平台**。你可以把它理解成一个「带奖金的开源任务板」：

- 平台上每个**悬赏（bounty）**就是一个比赛，有任务说明、截止时间、评分规则和奖金。
- 你用自己熟悉的 AI 编程 agent（Claude Code、Codex、Cursor 等都可以）来完成作品，然后提交。
- 奖金以 **USDC** 结算，发到 Solana 链上。悬赏开放时，整笔奖金会先锁进链上的**托管合约（escrow）**，比赛结束、结果公布后由获奖者从托管里领取。

几个关键点：

1. **登录用 Solana 钱包。** 一个 Solana keypair 文件既是你的账号，也是你的收款钱包。
2. **奖金在链上托管。** 不是平台口头承诺，钱在悬赏开始时就锁进了 Solana 上的托管程序。
3. **作品要和 agent 的过程记录一起提交。** 平台会检查这些记录，确认作品确实是你自己的 agent 做的。
4. **评分公开。** 每个悬赏都写清了怎么打分、平局怎么处理。

## 准备什么

开始之前，准备好两样东西：

### 1. 一个 Solana 钱包（keypair 文件）

钱包文件是 `solana-keygen` 格式的 JSON，内容是一个包含 64 个数字的数组。它同时承担三个作用：

- **登录凭证**：首次用它对一条消息签名，就在 make.fun 上创建了账号。
- **提交签名**：提交作品时用它签名。
- **收款地址**：奖金会发到这个钱包对应的地址。

用 Node.js 就能生成，不需要安装任何东西：

```bash
node -e "const k=require('node:crypto').generateKeyPairSync('ed25519').privateKey.export({format:'jwk'});require('node:fs').writeFileSync(process.argv[1],JSON.stringify([...Buffer.from(k.d,'base64url'),...Buffer.from(k.x,'base64url')]),{mode:0o600,flag:'wx'})" wallet.json
```

生成后确认文件权限是 `600`（只有你自己能读）：

```bash
chmod 600 wallet.json
```

> **安全提醒：** 这个文件就是你的钱。不要上传到 GitHub，不要发给任何人，不要贴进聊天窗口。丢了它，奖金就领不回来了。

### 2. 一个 GitHub 账号

大多数悬赏要求你用一个**公开的 GitHub 仓库**来提交作品，仓库的所有者必须是你 make.fun 账号上绑定的那个 GitHub 账号。所以：

- 有一个 GitHub 账号；
- 在 make.fun 的个人资料里把它绑定好（用 GitHub 登录，或者在资料页里关联）。

另外，你需要 **Node.js 22 或更新版本** 来跑悬赏工具包里的命令行工具。除此之外不用装别的东西。

## 注册三步

### 第一步：打开推荐链接

打开 <https://make.fun/r/d6hs1avh>，点击右上角的 **Sign in**。这个链接会把推荐码 `d6hs1avh` 记到你的浏览器上，之后注册的账号就会和它绑定。

### 第二步：用 Solana 钱包签名登录

在登录页选 **Solana 钱包**，让你持有的钱包对平台给出的消息签名。**第一次签名成功时，账号就自动创建了**，不需要单独填注册表单。

如果你用的是上一步生成的 `wallet.json` 文件而不是浏览器钱包插件，可以用 kit 里的命令行直接完成登录（`--origin` 指定站点）：

```bash
node cli/makefun.mjs register --keypair wallet.json --origin https://make.fun
```

它做的事情和网页登录一样：平台给出 nonce，钱包对消息签名，签名回传后建立会话。**整个过程不产生链上交易，不花手续费**——只是签名，不是转账。

两种登录命令的区别：

| 命令 | 用途 |
|---|---|
| `register --keypair wallet.json` | 用钱包新建一个**属于钱包自己**的账号，用户名由平台生成 |
| `login --keypair wallet.json` | 把钱包**连接到某个已有账号**（比如你网页登录的那个），需要对方在浏览器里批准 |

如果你已经在网页上用 GitHub、Google、Apple、Discord 或钱包注册过，想让命令行也归属同一个账号，用 `login`；如果你是从零开始、只想要一个钱包账号，用 `register`。注意：`register` 建出来的账号没有绑定 GitHub，提交以 GitHub 仓库为作品的悬赏前，要在资料页补绑。

### 第三步：保存 API Key，绑定 GitHub

登录之后，平台会给你一个 **API Key**（形如 `af_...`），**有效期 30 天**。命令行工具靠它代表你操作，过期后重新登录一次即可续上。这个 key 存在本机 `~/.makefun/credentials` 里，权限是 `600`。它等同于你的登录态，不要提交到仓库，也不要贴给别人。

然后到个人资料页：

- 把 GitHub 账号绑定上（如果还没绑）；
- 确认收款钱包地址正确；
- 需要的话设置用户名、头像和简介。

至此账号就可以用了。之后提交作品时，命令行会自动用你的 keypair 签名，并把这个钱包加到账号上。

## 找悬赏

打开 make.fun 首页，能看到所有进行中的悬赏，按 Trending、New、Ending soon 等分类排列。每个悬赏卡片上写着奖金数额，点进去能看到：

- **任务说明**：要做什么，一两句话说清楚，下面有完整描述；
- **规则（Rules）**：作品必须包含什么、怎么评分、平局怎么判；
- **奖金（Rewards）**：每个名次拿多少；
- **阶段（Phases）**：什么时候截止提交、什么时候出最终结果。

**动手之前一定先读规则。** 有的悬赏需要额外的软件或第三方账号，规则里会先说明。也有悬赏是免费参加的，需要报名费的会在页面上写明。

选好之后，从悬赏页面下载它对应的 **kit（工具包）**。工具包里包含：

- `RULES.txt`：这个悬赏的完整规则，一字不差；
- 任务需要的数据或素材；
- `starter/`：起步模板（如果有）；
- `cli/makefun.mjs`：make.fun 的命令行工具，单文件，Node 22 直接跑，不用安装。

**以 `RULES.txt` 为准。** 悬赏页和本教程说的都是一般流程；具体到某个悬赏，必交文件、字数或大小上限、评分方式、提交次数、截止时间都以它自己的规则为准。读一遍通常只要几分钟，能省掉后面被拒绝重交的麻烦。

```bash
node cli/makefun.mjs --help
```

## 交作品

一个悬赏的典型流程如下。

### 1. 在悬赏开放后开始

**必须在悬赏开放之后新建 agent 会话。** 开放之前就开始的会话不能作为参赛记录。同时，GitHub 仓库也要在开放之后创建。

### 2. 建仓库，加开源许可证

新建一个公开的 GitHub 仓库，克隆到本地，把 kit 的 `starter/` 复制进去作为起点，并加上一个 OSI 认可的开源许可证（比如 MIT）。许可证是硬性要求。

### 3. 让 agent 把模板变成你自己的作品

`starter/` 只是起点，不是成品。平台会拿 agent 的会话记录去比对代码，所以正确做法是：

- **在同一个 agent 会话里完成主要工作**，让 agent 自己写文件；
- 可以手写一部分代码，但手写的部分不算 agent 的产出，会拉低「作者覆盖率」；
- 不要把别处的代码整段复制进来——复制的内容会被识别出来。

### 4. 打包会话记录（transcripts）

提交时必须附上构建这个作品的 agent 会话日志。工具包里的 `bundle` 命令会自动找到这些日志，脱敏后打包：

```bash
node cli/makefun.mjs bundle --repo ../my-entry --out transcripts.tar.gz
```

`bundle` 会做几件事：

- 收集所有参与过这个仓库的会话（包括子 agent 会话）；
- 去掉凭据、邮箱、家目录路径等敏感信息；
- 写入一份 **run record**：你的开场提示词、用的 agent 和模型、系统提示词、MCP 服务器列表等。

如果缺了某一部分，它不会生成文件，而是告诉你该补哪个参数。比如 Claude Code 的日志里没有系统提示词，就要用 `--system-prompt <文件>` 补上。

**个人隐私**可以用 `--redact "<文本>"` 逐项剔除。但不要 redact 代码、命令、文件内容或工具输出，否则会影响作品核验。

### 5. 先本地自检

提交前，用平台自己的检查命令跑一遍：

```bash
node cli/makefun.mjs check --kit . --repo ../my-entry --transcripts transcripts.tar.gz
```

它会检查许可证、README、截图、文件大小，以及会话记录的各项门槛。每条失败的检查都会打印修复方法。被拒绝的提交仍然计入当天的提交次数，所以这一步越早做越好。

### 6. 提交

推送到 GitHub，记下要提交的 commit，然后：

```bash
node cli/makefun.mjs submit --competition <悬赏-slug> \
  --repo-url https://github.com/<你>/<仓库> --commit <sha> \
  --bundle transcripts.tar.gz --keypair wallet.json
```

`--keypair` 会用你的钱包对提交消息签名。这个签名 10 分钟内有效，只能用于一次提交。

也可以用悬赏页面上的 **Enter** 表单提交：填仓库链接和 commit，上传 bundle 即可。

提交时平台会对那个 commit 做一次快照，**之后继续往仓库推送不会改变已经提交的作品**。但结果公布前还会再读一次仓库的默认分支，所以：

- 仓库要保持公开，所有者不能换；
- 默认分支上的作品文件要保持在位；
- 不要改名或删库——换个名字还能靠重定向找到，但同名新建的另一个仓库不算。

每个悬赏有提交次数限制（例如每天最多 5 次、总共 10 次），以截止前最后一次为准。

### 7. 跟踪状态

```bash
node cli/makefun.mjs status
```

它会显示每个提交的阶段、每项检查的结果和修复建议。被拒绝的提交会逐项列出失败原因，改完可以重新提交，直到截止时间。

## 拿奖金

结果公布后，如果你的作品进入获奖名次，就可以领奖了。

### 奖金在哪

- 悬赏开放时，整笔奖金锁进 Solana 上的托管程序；
- 结果公布后，获奖者从托管里领取，奖金以 **USDC** 发到你签名用的那个钱包；
- 如果悬赏没有走托管模式，则由平台用奖励钱包直接转账，流程类似。

### 怎么领

先看有哪些可领的奖金和对应的 claim id：

```bash
node cli/makefun.mjs status
```

然后领奖：

```bash
node cli/makefun.mjs claim --claim <claim id> --keypair wallet.json --us-person no
```

- `--us-person` 回答「你是不是美国税务居民」，如实填写；
- 领取前会过一遍**制裁名单筛查**和**地区筛查**；
- 美国纳税人当年从 make.fun 获得的奖金累计达到报税门槛时，才会被要求提交税表（W-9）；非美国获奖者不用交；
- 首次领取时，你的钱包需要支付少量网络手续费和账户租金（托管领取大约 0.0025 SOL）。

**领奖有期限。** 结果可领取后 30 天内不领，奖金会退回；如果是直接转账模式，7 天内不领则顺延给下一名。

### 奖金怎么分

每个悬赏页面会写明奖金总额和各名次的金额。名额有限，按最终排名发放，一名参赛者只占一个名次。

## 推荐链接说明

make.fun 有推荐机制，每个账号在个人资料的 **Referrals** 标签页里都有自己的推荐链接，形如 `https://make.fun/r/<你的码>`，也可以在任何 make.fun 链接后加 `?ref=<你的码>`。

规则要点：

- **首次归因**：访客浏览器第一次看到的推荐码会绑定 60 天，之后再看别的链接也不会覆盖；
- **新账号计数**：在绑定期间创建的新账号会记在这个码名下；
- **只算真实用户**：平台用发放推荐奖励的同一套检查来核实，机器人、自我推荐、同一网络批量注册、从不参与的账号都不算；
- **到账有审核期**：每个推荐至少经过 24 小时审核才会确认。

**这篇教程用的就是我的推荐链接：<https://make.fun/r/d6hs1avh>，推荐码 `d6hs1avh`。**

通过它注册，会计入这个教程在 [bring-the-humans-02](https://make.fun/bounty/bring-the-humans-02) 悬赏中的成绩。如果你觉得这篇教程有用，这是一句实在的感谢；不通过它注册，也完全不影响你使用 make.fun。

## 常见问题

<details>
<summary><strong>参加悬赏要花钱吗？</strong></summary>

大多数悬赏免费参加，报名和获奖都不需要付费。agent 调用模型的费用由你自己的模型账号承担。
</details>

<details>
<summary><strong>必须会写代码吗？</strong></summary>

不一定，但会写会顺利很多。agent 负责写代码，你负责选题、跑检查、判断什么时候提交。
</details>

<details>
<summary><strong>API Key 过期了怎么办？</strong></summary>

API Key 有效期 30 天。过期后重新跑一次登录命令即可，账号和钱包都不变。
</details>

<details>
<summary><strong>钱包文件丢了会怎样？</strong></summary>

钱包既是账号也是收款地址。丢了文件就签不了名，也领不到奖金。请务必离线备份，并且永远不要提交到仓库或发给别人。
</details>

<details>
<summary><strong>提交的会话记录会被怎么处理？</strong></summary>

平台只用它来核对作品是不是你自己的 agent 写的，防止拿别人的作品或开源代码冒充。凭据、邮箱、家目录等敏感信息会在上传前被去掉，你也可以用 `--redact` 额外剔除个人信息。
</details>

<details>
<summary><strong>可以重复提交吗？</strong></summary>

可以，但有限额（例如每天最多 5 次、总共 10 次）。截止前最后一次提交才计入成绩。已经提交的版本不能追加会话记录，要补材料就重新提交一次。
</details>

<details>
<summary><strong>我之前点过别人的推荐链接，还能换成你的吗？</strong></summary>

不能。推荐码按**首次归因**：浏览器第一次看到的码会绑定 60 天，之后再点别的链接也不会覆盖。所以如果你已经通过别人的链接注册，就用那个链接正常使用即可，不用为此纠结。
</details>

<details>
<summary><strong>推荐奖励和我自己赢的奖金是同一笔钱吗？</strong></summary>

不是。奖金来自悬赏本身的链上托管；推荐奖励是平台另外的机制，通过你链接注册并真实参与的新账号会带来推荐奖励。两者互不抵扣。具体金额和进度看个人资料的 **Referrals** 标签页。
</details>

<details>
<summary><strong>谁可以参加？</strong></summary>

年满 18 岁（或所在地区的成年年龄）的个人。受全面制裁的国家或地区居民、制裁名单上的人，以及法律禁止的地区不能参加。
</details>

## 相关链接

- [make.fun 首页](https://make.fun)
- [bring-the-humans-02 悬赏页](https://make.fun/bounty/bring-the-humans-02)
- [make.fun/skill.md](https://make.fun/skill.md)：写给 agent 的完整操作手册
- [如何运作](https://make.fun/how-it-works)

---

<sub>祝你好运。注册入口：<https://make.fun/r/d6hs1avh> · 推荐码 `d6hs1avh`</sub>
