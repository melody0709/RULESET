# RULESET — melody0709 自建规则集

供 `2027kuromis/2027kuromis.ai-chain.yaml`（OpenClash / mihomo 内核 v1.19.x）引用的规则集，
全部为 `behavior: classical`（DOMAIN / IP-CIDR / SRC-IP-CIDR / DST-PORT 混合），格式为 mihomo yaml payload。

引用方式（**不用 jsdelivr**，原因见 `2027kuromis/manual.md` 变更记录「31 条 provider 源：换过又撤回」）：

```
https://raw.githubusercontent.com/melody0709/RULESET/refs/heads/main/<路径>/<表名>.yaml
```

## ⚠️ 先看：序号口径（2026-09-14 统一）

下表 **「位置」列 = 该表在 `rules:` YAML 数组里的绝对下标（1-based）**，基准是
`2027kuromis.ai-chain.yaml`（**2026-09-14 v10.3 实施后，`rules` 共 65 条**）。
此前文档里存在另外几套口径（只数 RULE-SET 的、打完补丁后的），**引用时请注明口径**。

配套条目：`ruleset_self_DIRECT` **26** / `ruleset_self_AI_bulk` **27** / `mrs-category-ai-cn` **28** /
`ruleset_self_AI` **29** / `ruleset_self_US` **30** / `ruleset_self_JP` **31** / `ruleset_self_PROXY` **32** /
`ruleset_self_nas&tv` **33** / `ruleset_self_miner` **34** / `RULE-SET,telegramip` **41** /
`RULE-SET,media` **55** / `RULE-SET,mediaip` **56** / `RULE-SET,Crypto` **57** / `RULE-SET,games-cn` **58** /
`RULE-SET,Game` **59** / `RULE-SET,NVIDIA` **60** / `MATCH` **65**。

> 原 `RULE-SET,proxy` / `RULE-SET,tld-proxy` 已随 v10.3 删除 —— `proxy.mrs` 的域名有 99.98%
> 由 `geosite:geolocation-!cn`（第 61 条）接住。

## ⚠️ 顺序就是命中优先级

这些表在 `rules:` 数组里的**先后位置**决定谁先命中。本目录 9 条自建表连续占据 **第 26–34 位**，
（`DIRECT` 第 26 条、`AI_bulk` 第 27 条、`AI` 第 29 条），**排在所有公共 GEOSITE 之前**：

- 里面的 `DOMAIN-KEYWORD` / `DST-PORT` / `SRC-IP-CIDR` 都是宽匹配，会**静默抢走**公共表已收录的域名，
  不会报错、不会告警；
- **不要**在 `DIRECT` 表里加 `DOMAIN-KEYWORD,smtp` 之类的宽关键字（2026-09-13 试过又撤了，
  详见该文件内注释与 `2027kuromis/manual.md`）；
- 调整顺序前先做改前/改后位移验证。

## 各表一览（按在 `rules:` 里的位置排序）

| 表 | 指向组 | 位置 | 条数 | 用途 |
|---|---|---|---|---|
| `DIRECT/ruleset_self_DIRECT.yaml` | `DIRECT` | 第 26 条 | **81** | 强制直连：SRC-IP `10.10.10.70/26` 整段设备（**用户自设的有意设计，勿动勿再复查**）、IPTV / 直播源、jsdelivr·ghproxy、面板域名、国内模型/镜像站、国行 Steam（`steamchina.net`）、NTP 对时（`ntp.org` / `time.windows.com` / `time.apple.com`）、Epic 下载 CDN、**国产 AI 保活（2026-09-14 +7：`kimi.ai` `moonshot.ai` `siliconflow.com` `stepfun.com` `01.ai` `wanzhi.com` `lingyiwanwu.com`）**。**2026-10-04 +5 WorkBuddy & CodeBuddy**（腾讯 AI 智能体/代码套件加固直连：`workbuddy.cn` `codebuddy.cn` `tencentbuddy.com` `workbuddy.link` `tcsdk.com`）。**含宽匹配**：`DOMAIN-KEYWORD,wogg` `360zy` `libvio` `argotunnel`、`DST-PORT,7844`。**2026-09-14 +10 面板域**（补 `geosite:private` 未收录）：`yacd.metacubex.one` `metacubexd.pages.dev` `metacubex.github.io` `zephyruso.github.io` `dash.sing-box.app` `sing-box-dashboard.sagernet.org` `board.zash.run.place` `p.to` `acl4.ssr` `wifi.cmcc` |
| `AI/ruleset_self_AI_bulk.yaml` | `🔰US` | 第 27 条 | 7 | AI 大流量下载（HuggingFace / Colab / Civitai / Ollama registry）→ 机场，**必须早于 `ruleset_self_AI`**，否则会退回 AI 落地（300G/月 配额） |
| `AI/ruleset_self_AI.yaml` | `🅰️AI` | 第 29 条 | **50** | AI 主域（Claude / Cursor / xAI / Gemini 入口 / Perplexity / OpenRouter …）+ AI 编程 CLI（OpenCode / CommandCode / models.dev，2026-09-13 实测 21 份公共表零收录）+ **AI 边界 / 遥测 / 外挂服务（2026-09-26 +14，见下）**。`colab.*` 与 `huggingface.co` / `hf.co` 与 `AI_bulk` **有意重复**（本表被单独引用时语义才完整），勿按"重复项"删 |
| `nas&tv/ruleset_self_nas&tv.yaml` | `⚓️nas&tv` | 第 33 条 | 1 | 按 SRC-IP 把 `10.10.10.189` 的流量交给 NAS/TV 专用出口 |
| `miner/ruleset_self_miner.yaml` | `🔰miner` | 第 34 条 | 15 | 矿机（SRC-IP 5 台）+ 矿池域名直连分流 |
| `PROXY/ruleset_self_PROXY.yaml` | `🥖Proxy` | 第 32 条 | **12** | 个人常用代理域；`challenges.cloudflare.com`（Turnstile）收窄版，勿再扩成整域 `cloudflare.com`。**2026-09-14 +2**：`mstea.ms` / `outlookgroups.ms`（`geosite:cn` 含 TLD 级 `ms` 宽条目，这两条不在 `geolocation-!cn` 内，需显式钉回本组） |
| `US/ruleset_self_US.yaml` | `🔰US` | 第 30 条 | 21 | 固定走美国的个人域（docker / v2ex / vercel / 车主域 …）+ **云与基础设施防漂移（2026-09-14 +4：`amazonaws.com` `aws.amazon.com` `console.aws.amazon.com` `docker.io`）**；购物站 `amazon.com` 有意**不**进本表，继续由 `geosite:geolocation-!cn` 兜底。`auth0.com` 已移除（会抢跑 AI 登录域）—— 2026-09-26 起由 `ruleset_self_AI` 正式接管，意图闭环。⚠️ 因本表早于 `ruleset_self_AI`，其宽条目 `amazonaws.com` 仍会抢走上游 AI 表的 S3 域，故 `ppl-ai-file-upload.s3.amazonaws.com` 在 AI 表内**个案补** |
| `JP/ruleset_self_JP.yaml` | `🔰JP&KR` | 第 31 条 | 11 | 日本区站点 |
| `HK/ruleset_self_HK.yaml` | —（**未引用**） | — | 0 | 预留空表，只有一行注释。要启用时在 `rules` 里加 `RULE-SET,ruleset_self_HK,🔰HK` |

> **条数为实测值**（YAML `payload` 数组长度），不是行数；本表此前的条数列有偏差，已按实测更正。
> **2026-09-14 v10.3 复测**：`DIRECT` 66 → **76**（+10 面板域）、`PROXY` 10 → **12**（+2 微软短链），其余不变。
> **2026-09-26 复测**：`AI` 36 → **47**（+11，AI 边界 / 遥测 / 外挂服务），其余 8 表不变；`rules:` 顺序未动。
> **2026-09-26 追加**：`AI` 47 → **50**（+3，Claude 风控 / 埋点：Sift / Segment），其余 8 表不变；`rules:` 顺序未动。

### 2026-09-26 · AI 边界 / 遥测 / 外挂服务（`ruleset_self_AI` +11）

起因：iPhone 端发现 ChatGPT 相关流量落 `ruleset_self_DIRECT`（后查明该设备当时在 `SRC-IP 10.10.10.70/26` 强制直连段内，已改为 `.232`）。顺带审计四条 AI 线（OpenAI / Claude / Gemini / Grok）的**边界域、遥测域、CDN 边缘**。

根因：**2026-09-14 v10.3 用 MetaCubeX `mrs-category-ai-!cn` 换掉了 DustinWin `ai.mrs` + HotKids `GenAI`**，换表时丢掉的不只是冗余，还有一批「厂商外挂的第三方服务域」。方法是把两份旧表与现行表全量回归比对（旧 203 条 / 现行 179 条 / **仅存旧表 28 条**），再逐条跑落点判定。

新增 11 条（全部并入 `AI/ruleset_self_AI.yaml`，**两份配置的 `rules:` 与 `rule-providers:` 零改动**）：

| 档 | 条目 | 原落点 |
|---|---|---|
| P0 登录/风控 | `DOMAIN-SUFFIX,arkoselabs.com`（OpenAI 登录人机验证） | ⚓️Other（MATCH 裸奔） |
| P0 | `DOMAIN,api.statsig.com` / `DOMAIN-SUFFIX,statsigapi.net` / `DOMAIN-SUFFIX,featuregates.org` | ⚓️Other |
| P0 | `DOMAIN-SUFFIX,auth0.com`（回收，闭环 `ruleset_self_US` 2026-09-13 撤下时的意图） | 🥖Proxy |
| P0 | `DOMAIN-SUFFIX,ai.azure.com` / `DOMAIN-SUFFIX,openai.azure.com` | 😶‍🌫️Windows |
| P1 遥测 | `DOMAIN-REGEX,^o33249\.ingest\.(.+\.)?sentry\.io$`（覆盖含 `.us.` 的区域变体） | 🥖Proxy |
| P2 宽条目抢跑 | `DOMAIN,d3ssuo6fbvr.cloudfront.net`（被 `media.mrs` 的 `+.cloudfront.net` 吞） | 🎬Media |
| P2 | `DOMAIN,ppl-ai-file-upload.s3.amazonaws.com`（被本仓 `ruleset_self_US` 的 `amazonaws.com` 抢先） | 🔰US |
| 用户拍板 | `DOMAIN-SUFFIX,x.com`（统一 Grok 出口，含 `api.x.com` / `grok.x.com`） | 🔰US（`geosite:twitter`） |

验证：内核 `-t` 解析通过（无告警）＋ 运行时用 `hosts` 定向探活确认 8 个新条目全部 `match RuleSet/ai`、阴性对照 `match Match using REJECT` ＋ 改前/改后位移对比（before 用 `git show HEAD:AI/ruleset_self_AI.yaml`）**13 项位移全部 → 🅰️AI、0 项未达标、11 项对照零意外位移**。

**未加（有意）**：`sentry.io` 泛域（共享 SDK，与 2026-09-13 决策一致）、`challenges.cloudflare.com`（Turnstile 非 IP 绑定型风控，保留 `ruleset_self_PROXY` 收窄设计）、`intercom` 系、`identrust.com`、`livekit.cloud` 裸域、`twimg.com`（X 媒体大流量）、`*.edgekey.net`（泛域）。完整推演见 `GIST/.plan/fix/ai-boundary-telemetry-audit-2026-09-26.md`。

### 2026-09-26（追加）· Claude 风控 / 埋点（`ruleset_self_AI` +3）

起因：iPhone 端登录 Claude 后，面板观察到 `api3.siftscience.com` 与 `api.segment.io` 走 `MATCH → ⚓️Other → 🔰US-slect`，与 AI 主站（`🅰️AI → 🌄 落地节点`）**出口不一致**。

`api3.siftscience.com` = **Sift**（反欺诈 / 风控），`api.segment.io` = **Segment**（产品埋点 / 事件管道）—— 两者均为 Claude 客户端触发的「厂商外挂第三方服务」。

| 档 | 条目 | 原落点 |
|---|---|---|
| P0 风控 | `DOMAIN-SUFFIX,siftscience.com`（Sift） | ⚓️Other（MATCH 裸奔） |
| P1 遥测 | `DOMAIN-SUFFIX,segment.io` / `DOMAIN-SUFFIX,segment.com`（Segment） | ⚓️Other |

> ⚠️ **与 Sentry 先例的口径差异（有意，用户 2026-09-26 拍板）**：`sentry.io` 是多租户数据平台（每组织独立主机 `o<org>.ingest.*.sentry.io`，泛域会扯进所有客户）→ 只收精确主机；
> `siftscience.com` / `segment.io` 是**厂商自有服务域**，整域可覆盖 `api` / `api3` / `cdn` 等变体，代价是少量非 AI 站点的风控/埋点一并进 `🅰️AI`（KB 级，可用性无影响）。

两份配置的 `rules:` 与 `rule-providers:` **零改动**。

验证（2026-09-26 追加，全部实测）：

- 内核 `mihomo-windows-amd64.exe -t`（v1.19.30）：改后规则集当 `type: file` provider + `2027DMIT_only.yaml` + `2027kuromis.ai-chain.yaml` **3/3 `test is successful`**，0 warning 0 error；
- 运行时定向探活（最小内核 + `ruleset_self_AI → DIRECT` / `MATCH → REJECT`，读内核日志）：**7/7 `match RuleSet(ruleset_self_AI)`**（6 个目标主机 + `claude.ai`），阴性对照 `this-host-is-the-control.test` → `match Match using REJECT`；
- 改前/改后位移对比（输入源隔离：before = `git show HEAD:AI/ruleset_self_AI.yaml`，after = 本地工作树，用 `work/scripts/mhaudit/sim.py`）：**6 项位移全部 `⚓️Other → 🅰️AI`、0 项未达标、11 项对照零意外位移**；AI 表 47 → 50。

## 维护约定

- 改完**必须**先本地验证再推送：`mihomo -t` + 改前/改后位移对比（before 用 `git show HEAD:<路径>` 导出，
  不要用旧缓存 —— 缓存比 HEAD 旧一轮会虚报位移）。
- 推送后核对远端**内容**（SHA256 / 特征串），不要只看 HTTP 200。
- 本目录即公开仓库：不要在这里放分析文档、缓存、订阅内容。
- 相关分析文档在 `GIST/work/docs/` 与 `GIST/.plan/`，**不**随本仓发布；那些文档顶部须标
  `状态：现行 / 部分失效（列条目）/ 已被取代`（约定见 `GIST/AGENTS.md` §九）。
