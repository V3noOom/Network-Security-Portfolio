# IDS Evasion & the Limits of Signature-Based Detection

**Course:** Network Security, Stockholm University — Spring 2026
**Environment:** Isolated lab network, Kali Linux (attacker) → Windows target, Snort running as a console IDS

## Overview

Signature-based IDS rules are only as good as the assumptions baked into them. This project systematically probes a single, deliberately simple Snort rule to find out exactly which traffic modifications evade detection — and, more importantly, *why* — then translates those findings into concrete hardening recommendations.

**The rule under test:**
```
alert tcp any any -> any 80 (msg:"Bad Request Detected"; content:"badstuff"; sid:100001;)
```
A raw-byte match for the literal string `badstuff` in any TCP payload on port 80.

## Methodology

Traffic was crafted with `netcat` and sent at the target while watching the Snort console. Each test changed exactly one variable from the baseline request so its effect could be isolated.

| # | Modification | Alert Fired? | Why |
|---|---|---|---|
| 1 | Baseline: `GET /badstuff` on port 80 | ✅ Yes | Exact string match on the expected port |
| 2 | Port changed to 8080 | ❌ No | Rule is hardcoded to port 80 |
| 3 | Path changed to `/goodstuff` | ❌ No | Trigger string no longer present |
| 4 | Path prefix: `/files/badstuff` | ✅ Yes | `badstuff` still present as a substring |
| 5 | Custom `User-Agent` header | ✅ Yes | Rule has no header awareness |
| 6 | Method changed to `POST` | ✅ Yes | Rule has no method awareness |
| 7 | Extra header (`X-Forwarded-For`) | ✅ Yes | Rule has no header awareness |
| 8 | Single character percent-encoded (`%62adstuff`) | ❌ No | Raw-byte match; encoded bytes don't match the literal string |
| 9 | Fully percent-encoded path | ❌ No | Same as above |
| 10 | Double-slash path (`//badstuff`) | ✅ Yes | String still present as raw bytes |
| 11 | Rapid burst (10 requests, no delay) | ✅ Yes, every request | Rule has no timing/rate logic |

## Key Findings

Only two changes reliably evaded detection: **changing the port** and **URL-encoding the trigger string**. Everything else — headers, HTTP method, path structure that still contained the raw string, and timing — had no effect, because the rule never inspects those dimensions in the first place.

That asymmetry is the real finding. The rule encodes several implicit assumptions about attacker behavior:
- Attackers always use port 80
- Attackers never encode their payloads
- The malicious string always appears as an unmodified literal
- Timing and headers are irrelevant to the attack itself (true here, but not universally)

Breaking just *one* of the two assumptions that actually mattered (port, encoding) was enough for complete evasion. A production attacker only needs to find one gap; a defender relying on static signatures has to anticipate every possible variation in advance — a fundamentally asymmetric position.

## Recommendations for a More Robust Rule

- Use `any` for the port instead of hardcoding 80, or scope more precisely with protocol-aware inspection instead of relying on port number as a proxy
- Enable HTTP normalization so URLs are decoded *before* content matching runs
- Add `nocase` and use `pcre` for pattern matching that tolerates case and minor structural variation
- Layer signature-based detection with anomaly-based detection to catch behavioral outliers that no single signature will ever enumerate

## Reflection

This exercise is a compact illustration of why static, signature-based defenses struggle against any adversary willing to iterate. Each attempt here functioned like a trial-and-error search: try a variation, observe whether it's detected, adjust, repeat — conceptually the same feedback loop that makes an adaptive attacker effective against a fixed defense. The practical lesson for defenders is that a single detection layer, however well-tuned, is a single point of failure; robustness comes from normalization, broader pattern matching, and defense in depth — not from a longer list of literal strings.

## Skills Demonstrated

`Snort rule authoring & tuning` `IDS/IPS evasion methodology` `protocol-level traffic crafting` `tcpdump packet analysis` `signature-based detection limitations`
