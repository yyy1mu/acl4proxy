# Repository Instructions

## Scope

This repository maintains one standalone Shadowrocket configuration:
`shadowrocket.conf`.

- Keep all runtime rules inline in `shadowrocket.conf`.
- Do not recreate `proxy-all.list` or `china-direct.list`.
- Do not add remote `RULE-SET` dependencies.
- Do not add `update-url` unless the user explicitly requests remote updates.
- Keep `README.md` and `NOTICE.md` consistent with the configuration.

## Routing Policy

Shadowrocket uses first-match semantics. Preserve this order in `[Rule]`:

1. LAN, loopback, captive portal, and DNS bootstrap rules use `DIRECT`.
2. Explicit overseas-service rules use `PROXY`.
3. Explicit mainland-service rules use `DIRECT`.
4. Mainland TLD rules use `DIRECT`.
5. `GEOIP,CN,DIRECT,no-resolve` handles known mainland destination IPs.
6. `FINAL,PROXY` is the mandatory fallback.

Do not move `GEOIP,CN` above explicit proxy rules. Overseas services must keep
priority even when their CDN resolves to a mainland IP.

Prefer `DOMAIN` and `DOMAIN-SUFFIX`. Avoid `DOMAIN-KEYWORD`, broad IP ranges,
and service IP rules unless the user explicitly requests them and the range is
authoritatively maintained.

## DNS Policy

Maintain split encrypted DNS:

- Default, proxy, and fallback DNS:
  - `https://1.1.1.1/dns-query#no-h3`
  - `https://8.8.8.8/dns-query#no-h3`
- Direct mainland DNS:
  - `https://doh.pub/dns-query#no-h3`
  - `https://dns.alidns.com/dns-query#no-h3`

Keep `doh.pub` and `dns.alidns.com` as early exact `DIRECT` rules. Do not pin
their IP addresses: the providers may change their production clusters.

Keep system DNS fallback disabled, DNS port 53 interception enabled, and DoH
HTTP/3 disabled unless the user explicitly changes this policy.

## Domain Maintenance

- Confirm new service domains using the company website or official API/network
  documentation.
- Mainland services may use non-`.cn` domains; add those explicitly as
  `DIRECT` when verified.
- Overseas and region-restricted services should be explicit `PROXY` rules,
  while `FINAL,PROXY` remains the safety net for omissions.
- Do not add changing CDN CNAME targets merely because they appear in a DNS
  lookup.
- Avoid duplicate rules and conflicting policies for the same domain.
- When a narrow exception overlaps a broad suffix, place the exception first
  and document why.

## Validation

After every rule or DNS change:

1. Run `git diff --check`.
2. Confirm there are no active `RULE-SET` or `update-url` entries.
3. Check for duplicate `DOMAIN` and `DOMAIN-SUFFIX` rules.
4. Check parent/child suffix overlaps across `PROXY` and `DIRECT` rules.
5. Confirm the tail still contains, in order:
   - mainland TLD rules;
   - `GEOIP,CN,DIRECT,no-resolve`;
   - `FINAL,PROXY`.
6. When DNS endpoints change, test each endpoint with a standards-compliant DoH
   wire-format request when network access is available.

## Git and Remotes

- The intended remote is `local` (`local/main`).
- Do not add a GitHub remote or push to GitHub unless the user explicitly asks.
- Do not commit or push automatically. Commit and push only on explicit user
  request.
- Preserve unrelated working-tree changes.

