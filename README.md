# wxy-rules

Personal traffic-routing rule configurations, organized by client. Used for traffic splitting and rule optimization across multiple providers.

## Repository Layout

| Directory | Purpose |
| :--- | :--- |
| `Shadowrocket/` | iOS / macOS Shadowrocket configurations (`.conf`). |
| `Clash/` | Clash / Mihomo / OpenClash configurations (`.yaml`). |

### `Shadowrocket/`

| Filename | Description |
| :--- | :--- |
| **`wxy-sr-rules.conf`** | Final Shadowrocket configuration currently in use. Customized splitting rules and policy groups. |
| `Nexitally.conf` | Original provider configuration, used as a basic reference. |
| `a-nomad.conf` | Reference configuration copied from [YouTubeResources](https://github.com/wxy8866/YouTubeResources). |

### `Clash/`

| Filename | Description |
| :--- | :--- |
| **`wxy-clash.yaml`** | Final Clash (OpenClash / Mihomo) configuration. Integrates multiple subscriptions via `proxy-providers` with regional auto-grouping and detailed service policy groups. |
| `Nexitally.yaml` | Original single-provider Clash YAML, used as reference. |
| `test.yaml` | Custom AI / Claude / Anthropic rule fragment. |

## Key Features

- **Policy Group Design**:
  - `AI` / `AI专用节点`: Optimized node selection for AI services (Claude / Anthropic, OpenAI, Copilot).
  - `自动选择` (Auto-Select): Switches to the lowest-latency node via `url-test`.
  - Streaming: `YouTube` / `Netflix` / `Disney+` / `HBO` / `Bahamut` / `Bilibili` / `MyTVSuper` / `Tiktok` — independent policy per service.
  - Apps: `Telegram` / `Microsoft` / `Apple` / `Google` / `Scholar` / `Crypto` — direct or proxied per use case.
  - Games: `Steam` / `Epic` / `PlayStation` / `Xbox` / `游戏平台`.
- **Regional Grouping**: Auto speed-test groups for Hong Kong, Japan, Singapore, USA, Korea, Taiwan, UK, Germany.
- **Multi-Provider Aggregation** (Clash only): `proxy-providers` accepts multiple airport subscriptions; nodes are routed into regional groups via regex `filter`.
- **Auto-Update**: Configurations are hosted on GitHub and support remote updates inside their respective clients.

## Rule Sources (excluding ad-blocking rules)

Both configs reference and integrate the following rule sets:
- [ACL4SSR](https://github.com/ACL4SSR/ACL4SSR)
- [Loyalsoldier/surge-rules](https://github.com/Loyalsoldier/surge-rules)
- [Loyalsoldier/clash-rules](https://github.com/Loyalsoldier/clash-rules) (Clash only)
- Custom rules (Claude / Anthropic enhancement, local process filtering, etc.)

See [`规则汇总.md`](规则汇总.md) for the rule-source index.

## How to Use

### Shadowrocket
1. Download or copy the raw URL of `Shadowrocket/wxy-sr-rules.conf`.
2. In Shadowrocket: **Remote Files** → **Add Remote File**.

### Clash / OpenClash / Mihomo
1. Replace the three `<REPLACE_ME_AIRPORT_*>` placeholders in `Clash/wxy-clash.yaml` with your subscription URLs.
2. Import as a profile (OpenClash: **Server Settings** → **Add**, point at the local file or its raw URL).
3. If your provider uses non-standard node naming, tweak the regional `filter:` regex on the regional `url-test` groups.



[openclash_custom_overwrite.sh](Clash/openclash_custom_overwrite.sh) 是OpenClause启动的自定义脚本
[default](Clash/default)是OpenClash为可设置的插件配置参数。是OpenClash提供的模版。
[Loyalsoldier](Clash/Loyalsoldier)是我自定义的额外给予Loyalsoldier提供的规则集的配置文件。
[add_proxy_groups](Clash/add_proxy_groups)是我自定义的额外添加的策略组配置文件。
---
*Personal study and use only.*
