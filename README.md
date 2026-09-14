# RULESET — melody0709 自建规则集

供 `2027kuromis/2027kuromis.ai-chain.yaml`（OpenClash / mihomo 内核 v1.19.x）引用的规则集，
全部为 `behavior: classical`（DOMAIN / IP-CIDR / SRC-IP-CIDR / DST-PORT 混合），格式为 mihomo yaml payload。

引用方式（**不用 jsdelivr**，原因见 `2027kuromis/manual.md` 变更记录「31 条 provider 源：换过又撤回」）：

```
https://raw.githubusercontent.com/melody0709/RULESET/refs/heads/main/<路径>/<表名>.yaml
```

## ⚠️ 先看：序号口径（2026-09-14 统一）

下表 **「位置」列 = 该表在 `rules:` YAML 数组里的绝对下标（1-based）**，基准是
`2027kuromis.ai-chain.yaml`（v2-B 实施后，`rules` 共 **74** 条）。
此前文档里存在另外几套口径（只数 RULE-SET 的、打完补丁后的），**引用时请注明口径**。

配套条目：`ruleset_self_DIRECT` 22 / `ruleset_self_AI_bulk` 23 / `mrs-category-ai-cn` 24 /
`ruleset_self_AI` 33 / `ruleset_self_nas&tv` 34 / `ruleset_self_miner` 35 / `ruleset_self_PROXY` 36 /
`ruleset_self_US` 37 / `ruleset_self_JP` 38 / `RULE-SET,media` 56 / `RULE-SET,mediaip` 57 /
`RULE-SET,proxy` 70 / `MATCH` 74。

## ⚠️ 顺序就是命中优先级

这些表在 `rules:` 数组里的**先后位置**决定谁先命中。本目录有 3 条排得很靠前
（`DIRECT` 第 22 条、`AI_bulk` 第 23 条、`AI` 第 33 条），**排在所有公共规则集之前**：

- 里面的 `DOMAIN-KEYWORD` / `DST-PORT` / `SRC-IP-CIDR` 都是宽匹配，会**静默抢走**公共表已收录的域名，
  不会报错、不会告警；
- **不要**在 `DIRECT` 表里加 `DOMAIN-KEYWORD,smtp` 之类的宽关键字（2026-09-13 试过又撤了，
  详见该文件内注释与 `2027kuromis/manual.md`）；
- 调整顺序前先做改前/改后位移验证。

## 各表一览（按在 `rules:` 里的位置排序）

| 表 | 指向组 | 位置 | 条数 | 用途 |
|---|---|---|---|---|
| `DIRECT/ruleset_self_DIRECT.yaml` | `DIRECT` | 第 22 条 | 66 | 强制直连：SRC-IP `10.10.10.70/26` 整段设备（**用户自设的有意设计，勿动勿再复查**）、IPTV / 直播源、jsdelivr·ghproxy、面板域名、国内模型/镜像站、国行 Steam（`steamchina.net`）、NTP 对时（`ntp.org` / `time.windows.com` / `time.apple.com`）、Epic 下载 CDN、**国产 AI 保活（2026-09-14 +7：`kimi.ai` `moonshot.ai` `siliconflow.com` `stepfun.com` `01.ai` `wanzhi.com` `lingyiwanwu.com`）**。**含宽匹配**：`DOMAIN-KEYWORD,wogg` `360zy` `libvio` `argotunnel`、`DST-PORT,7844` |
| `AI/ruleset_self_AI_bulk.yaml` | `🔰US` | 第 23 条 | 7 | AI 大流量下载（HuggingFace / Colab / Civitai / Ollama registry）→ 机场，**必须早于 `ruleset_self_AI`**，否则会退回 AI 落地（300G/月 配额） |
| `AI/ruleset_self_AI.yaml` | `🅰️AI` | 第 33 条 | 36 | AI 主域（Claude / Cursor / xAI / Gemini 入口 / Perplexity / OpenRouter …）+ AI 编程 CLI（OpenCode / CommandCode / models.dev，2026-09-13 实测 21 份公共表零收录）。`colab.*` 与 `huggingface.co` / `hf.co` 与 `AI_bulk` **有意重复**（本表被单独引用时语义才完整），勿按"重复项"删 |
| `nas&tv/ruleset_self_nas&tv.yaml` | `⚓️nas&tv` | 第 34 条 | 1 | 按 SRC-IP 把 `10.10.10.189` 的流量交给 NAS/TV 专用出口 |
| `miner/ruleset_self_miner.yaml` | `🔰miner` | 第 35 条 | 15 | 矿机（SRC-IP 5 台）+ 矿池域名直连分流 |
| `PROXY/ruleset_self_PROXY.yaml` | `🥖Proxy` | 第 36 条 | 10 | 个人常用代理域；`challenges.cloudflare.com`（Turnstile）收窄版，勿再扩成整域 `cloudflare.com` |
| `US/ruleset_self_US.yaml` | `🔰US` | 第 37 条 | 21 | 固定走美国的个人域（docker / v2ex / vercel / 车主域 …）+ **云与基础设施防漂移（2026-09-14 +4：`amazonaws.com` `aws.amazon.com` `console.aws.amazon.com` `docker.io`）**；购物站 `amazon.com` 有意**不**进本表，继续由 `proxy.mrs` 兜底。`auth0.com` 已移除（会抢跑 AI 登录域） |
| `JP/ruleset_self_JP.yaml` | `🔰JP&KR` | 第 38 条 | 11 | 日本区站点 |
| `HK/ruleset_self_HK.yaml` | —（**未引用**） | — | 0 | 预留空表，只有一行注释。要启用时在 `rules` 里加 `RULE-SET,ruleset_self_HK,🔰HK` |

> **条数为实测值**（YAML `payload` 数组长度，2026-09-14 核），不是行数；本表此前的条数列有偏差，已按实测更正。

## 维护约定

- 改完**必须**先本地验证再推送：`mihomo -t` + 改前/改后位移对比（before 用 `git show HEAD:<路径>` 导出，
  不要用旧缓存 —— 缓存比 HEAD 旧一轮会虚报位移）。
- 推送后核对远端**内容**（SHA256 / 特征串），不要只看 HTTP 200。
- 本目录即公开仓库：不要在这里放分析文档、缓存、订阅内容。
- 相关分析文档在 `GIST/work/docs/` 与 `GIST/.plan/`，**不**随本仓发布；那些文档顶部须标
  `状态：现行 / 部分失效（列条目）/ 已被取代`（约定见 `GIST/AGENTS.md` §九）。
