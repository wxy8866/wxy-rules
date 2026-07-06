# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A personal collection of traffic-routing rule configurations for two clients. **There is no source code, build system, package manager, test suite, or CI.** Don't look for `package.json`, `Makefile`, `pyproject.toml`, etc. — they don't exist. The artifacts are config files consumed directly by the clients:

- `Shadowrocket/*.conf` — Surge-style INI loaded by Shadowrocket (iOS / macOS).
- `Clash/*.yaml` and `Clash/*` fragments — Mihomo / Clash.Meta config loaded by Clash, OpenClash, or other Mihomo-based clients.

The two folders are **parallel implementations of the same routing intent** for different clients, not unrelated configs. Treat them as coupled when making behavioral changes.

## Shared design invariants

Both clients carry the same routing taxonomy. Preserve symmetry when changing one side — drift between SR and Clash configs is the most common source of bugs here.

- **AI / Anthropic / OpenAI traffic** goes to a dedicated group (`AI专用节点` in SR, `AI` in Clash) that defaults to Japan nodes. Custom AI rule blocks live inline in both files (Claude domains, CDN, auth, telemetry, IP-CIDR/ASN fallbacks); these are the canonical source. `Clash/自定义规则-优先匹配.yaml` mirrors the same AI rules as a paste-in fragment (see "OpenClash custom overwrite workflow" below) — it is not itself canonical.
- **Direct-by-name domains/processes** are kept identical across both configs: `jetbrains.com`, `ugreengroup.com`, `blueair.io`, `wxy8866.com`, `zerotier.com`, processes `Feishu` / `Lark` / `WeChat` / `Weixin`. Add to both when adding to either.
- **Regional groups** (港/日/新/美/韩/台/英/德) are url-test by latency. SR uses `policy-regex-filter`; Clash uses `filter:` on a `url-test` group consuming `proxy-providers`. Regex alternations should match by both Chinese and English substrings (`港|hk|hong\s?kong`).
- **Rule sources** are remote (no inline rule dumps). ACL4SSR + Loyalsoldier — see `规则汇总.md`. SR pulls Surge format from `ACL4SSR/Clash/*.list` and `Loyalsoldier/surge-rules`; Clash pulls behavior-classified sets from `Loyalsoldier/clash-rules` plus the same ACL4SSR `Clash/*.list` as `behavior: classical`.

## Clash config specifics (`Clash/wxy-clash.yaml`)

- **Multi-airport via `proxy-providers`**: three slots with placeholder URLs `<REPLACE_ME_AIRPORT_1/2/3>`. Real subscription URLs are intentionally **not committed** — keep the placeholders in the repo copy. If the user wants to test with real URLs, they should be added locally and not pushed.
- **YAML anchor pattern**: `x-ut-template: &ut` at top level holds shared `url-test` fields (`type/url/interval/tolerance/lazy/use`). All 9 url-test groups (the global auto-select + 8 regional groups) consume it via `<<: *ut` and only override `name:` / `filter:`. When adding a regional group, follow this pattern — don't repeat the full url-test block. The `x-ut-template` key is non-standard; Mihomo ignores unknown top-level keys.
- **`exclude-filter` on each provider** strips marketing/quota pseudo-nodes (`流量|到期|过期|官网|套餐|traffic|expire|reset`) that Nexitally-style airports inject as "nodes".
- **Validating after edits**: pyyaml and yq are not installed; use Ruby's psych which is available system-wide:
  ```
  ruby -ryaml -e "d = YAML.load_file('Clash/wxy-clash.yaml'); puts 'OK'; puts d['proxy-groups'].map{|g|g['name']}"
  ```

### OpenClash custom overwrite workflow (`Clash/openclash_custom_overwrite.sh` + fragments)

There's a second, independent mechanism for extending the Clash config that only applies to the router (OpenClash), separate from editing `wxy-clash.yaml` directly:

- `Clash/openclash_custom_overwrite.sh` is the actual script installed as OpenClash's "Custom Overwrite Script" — it runs at OpenClash startup and patches the *generated* running config using `ruby_edit` / `ruby_merge_hash` / `ruby_map_edit` calls (documented via commented demos in the file itself). It merges in `rule-providers`, `rules`, and `proxy-groups` that aren't part of the base `wxy-clash.yaml`.
- `Clash/Loyalsoldier` and `Clash/add_proxy_groups` are extension-fragment files (no `.yaml` extension, but YAML content) holding the actual `rule-providers`/`rules`/`proxy-groups` payloads that get pasted into the overwrite script or OpenClash's custom-rules UI field. `Loyalsoldier` adds Loyalsoldier's icloud/apple/direct/private/telegramcidr/cncidr/lancidr rule-sets; `add_proxy_groups` adds one-off proxy groups (e.g. `AppleTVplus`, `JAVDB`).
- `Clash/规则变更.md` is **not a changelog** — despite the name, it's a standing instruction prompt for Claude describing exactly how to generate new `ruby_merge_hash` rule-provider/rule snippets when the user asks to add a new routed service. **If the user asks to add a new proxy rule/rule-provider for OpenClash, read this file and follow its procedure** (output format, source repos to pull from — MetaCubeX meta-rules-dat / ACL4SSR / blackmatrix7 — URL verification requirement, and policy-group-name handling).
- `Clash/自定义规则-优先匹配.yaml` is the live "paste into custom rules, matched before rule-providers" fragment — currently mirrors the AI/Anthropic/JetBrains/Crypto/DDNS/Blueair rules also present in `wxy-clash.yaml`.
- `Clash/自定义规则-样例.yaml` is a syntax cheat-sheet (entirely commented out) enumerating Clash/Mihomo rule types (`DOMAIN-REGEX`, `SRC-IP-CIDR`, `AND`/`OR`/`NOT`/`SUB-RULE`, etc.) — reference only, not live config.
- `Clash/default` is OpenClash's own plugin-config template (LuCI-exposed env vars like `CONFIG_FILE`, `CORE_TYPE`, port settings) — vendor-provided reference, not authored content.

## Shadowrocket config specifics (`Shadowrocket/wxy-sr-rules.conf`)

- Surge INI sections: `[General]`, `[Proxy Group]`, `[Rule]`, `[Host]`, `[URL Rewrite]`.
- `update-url` in `[General]` points at the GitHub raw URL of this file. **When the file path changes (e.g. the folder reorg into `Shadowrocket/`), this URL must be updated** or subscribed devices keep pulling the old path until they fail.
- `[Proxy Group]` uses `policy-select-name=` to pin a specific default node into a url-test group (e.g. AI defaults to a specific Japan-Toyota node). Renaming nodes upstream breaks this — leave pinned names alone unless explicitly changing them.

## Reference / scratch files

These are inputs, not outputs — don't rewrite them as part of "fixing" the main configs:

- `Shadowrocket/Nexitally.conf`, `Clash/Nexitally.yaml` — upstream single-provider configs from the Nexitally service; reference only.
- `Shadowrocket/a-nomad.conf` — external reference from `wxy8866/YouTubeResources`.
- Root-level `Nexitally.conf` and `a-nomad.conf` are byte-identical stray duplicates of the `Shadowrocket/` copies (leftovers from before the folder reorg) — the `Shadowrocket/` copies are canonical; don't edit the root ones as if they were separate.

## Conventions

- **No emojis in authored content.** This repo previously used flag/icon emojis in proxy-group names (`🎯 Direct`, `♻️ 自动选择`, `🇭🇰 香港节点`) — they've been stripped. Don't reintroduce them in group names, comments, or markdown. The one allowed exception is emojis appearing inside regex literals whose purpose is to **match** upstream node names from third-party subscriptions (those are matching content, not authored decoration) — confirm with the user before adding any.
- Reference configs (Nexitally.* and a-nomad.conf) still contain emojis from upstream; don't "clean them up" — they're not ours.

## Publishing model

These files are consumed by clients over HTTPS (GitHub raw URLs), not packaged or installed. Path changes are user-facing breaking changes — any rename or folder restructure requires updating subscription URLs on every device using them.