---
name: scraper-discipline
description: Use when a scraper or unofficial API client gets empty/4xx/5xx responses the real site doesn't, before concluding an endpoint "doesn't support" something, when probing an undocumented API's parameters, or when blaming throttling/bot-protection. Triggers include "scraper broken", "returns empty", "works in the browser but not in curl", "reverse engineer the API", "hidden API", "find the site's API", "Cloudflare/Akamai blocking", "throttled", "capture the XHR".
---

# Scraper / Hidden-API Discipline

Bug class this skill targets: **false folklore about someone else's API**.
Conclusions like "this endpoint only works for leaf categories", "the server
has a quirk on pageSize=36", "we're being throttled" that are actually
artifacts of a confounded probe. They get written into docstrings as facts,
survive sessions, and cost orders of magnitude in wasted requests (a real
case below turned 257 queries into 11).

## Rule 1 — Capture the oracle before probing

The vendor's own client makes a request that provably works. That request is
ground truth; your probes are guesses. Before forming ANY theory about the
API, capture the site's exact request — full URL, every query param, headers.

For SPAs this takes persistence, not luck:
- First page load is often SSR — no XHR fires. Trigger a **client-side**
  action (pagination, filter, sort) to force one.
- If network capture shows nothing (service workers, extension blind spots),
  spy from the console:

```js
const spy = [];
const of = window.fetch;
window.fetch = (...a) => { spy.push(String(a[0]?.url || a[0])); return of(...a); };
const oo = XMLHttpRequest.prototype.open;
XMLHttpRequest.prototype.open = function(m, u, ...r) { spy.push(m + ' ' + u); return oo.call(this, m, u, ...r); };
// interact with the page, then inspect `spy`
```

Archive the captured request (URL template + params) in the scraper's
docstring. When your request later misbehaves, you diff against it instead
of re-deriving the API from scratch.

## Rule 2 — When yours fails and theirs works, diff EVERYTHING

List every difference between your failing request and the oracle: each
query param, headers, cookies, HTTP version, TLS fingerprint. Then bisect —
morph yours toward theirs one difference at a time.

The guilty parameter is usually one you never varied because it lives in a
shared constant (`COMMON_PARAMS`, a session default, a base URL). Sweeping
one variable while another sits frozen produces confident wrong answers:
two "independent quirks" that are really one bug in the probe design.

## Rule 3 — "Broken on exactly the vendor's value" indicts your request

If the value the vendor ships in production (their page size, their category
id, their field list) fails for you, the server is not quirky — your request
differs somewhere else. This inference needs no tooling, only noticing.
Treat any conclusion shaped like "the origin deterministically fails on this
one magic value" as a red flag: go back to Rule 2.

## Rule 4 — Deterministic vs stochastic, before blaming bot protection

Throttling explanations are seductive because they excuse any failure.
Before accepting one, replay the identical request ~5 times from a fresh
session, spaced out:
- **100% reproducible** → it's your request (or a server rule). Not a throttle.
- **Intermittent** → infrastructure: real throttle, flaky egress, origin flap.

Never debug both at once. On a noisy path (flaky exit node, shared IP,
soft-blocked range) every anomaly gets misattributed to the noise — move to
clean egress first or you cannot classify failures at all.

## Rule 5 — Anomalies are open questions, not facts

Docstrings and memory notes outlive the session and become the next
session's priors. Write unexplained behavior as unexplained:

```
# WEIRD, unexplained: pageSize=36 returns {} while 30/40 work — retest
# before building on this; likely a confound in our probe, not the server.
```

Never as settled fact ("a server-side quirk on that exact value"). A
confident wrong explanation blocks the re-test that would kill it.

## Rule 6 — Ask for less, get more (enterprise APIs)

Response-shaping params are load-bearing, not cosmetic. On SAP Hybris OCC
(`fields=`), OData (`$select=`), and GraphQL, a maximal projection
(`fields=FULL`) can hit a serialization error on one poisoned item and
return an empty/failed response for the whole page — while a trimmed
projection of only the fields you parse works everywhere. After capturing
the oracle (Rule 1), also probe with its projection:
- the page-size ceiling (often far above the site's own value),
- levels of the taxonomy the docs/probing said were "unbrowseable",
- per-item metadata (categories, variants) that can replace tree walks.

## Rule 7 — Bisect the request DOWN to its minimal form

Rule 2 morphs your request toward the oracle. Once it works, go the other
way: strip headers, cookies, and params one at a time until it breaks, then
add the last one back. What survives is the true required set. This is worth
the extra passes because:
- Every header/cookie you carry is a thing that can drift, expire, or get
  fingerprinted. A 3-param request is more robust across deploys than the
  40-header browser copy you started from.
- It tells you what's actually load-bearing (a `Referer`/`Origin` gate, one
  auth cookie) versus cargo-culted, so future breakage has a short suspect list.

Copy-as-cURL gives you the maximal request; minimizing gives you the real one.

## Rule 8 — Pair every probe with a control

A failing request in isolation is uninterpretable: you cannot tell "my request
is wrong" from "they're blocking me right this second." So never read a failure
alone — fire a **control** whose outcome you already know, right next to the
probe. The control is the oracle, or any query you're certain returns data.

- Probe fails, control also fails → environmental (throttle, IP flag, origin
  flap). Your request isn't the problem; mutating params is wasted motion.
- Probe fails, control succeeds → your specific request is wrong. Back to
  Rules 2–3.

This turns "is it me or them?" from a hunch into a one-shot controlled
experiment, and it's the fastest exit from the Rule 4 confound: replaying the
same suspect N times tells you only *whether* it's flaky, while a paired control
tells you *which side* owns the failure. Bonus — when the control is the oracle,
every probe also re-confirms ground truth still holds (Rule 5: oracles drift).

Interleave the control on a cadence during long runs, not just once: a block
that starts mid-scrape shows up as the control flipping from pass to fail,
pinning the moment you got flagged instead of leaving you to guess which page
"went empty."

## Worked example (Priceline, 2026-07)

| Folklore (3 sessions old) | Truth (one oracle capture) |
|---|---|
| "Only 257 leaf categories browse; departments return `{}`" | Departments browse fine; `fields=FULL` was the breakage |
| "pageSize=36 is a server-side quirk" (36 = the site's own value!) | Same fields bug; pageSize caps at 100, tripling throughput |
| "Empty body = Cloudflare throttle" | Deterministic empties were the projection; only intermittent ones were CF |

Fix discovered in ~10 browser probes after monkey-patching `window.fetch`
(Rule 1): the site sent `allCategories:mens` with a custom `fields=` list.
Result: whole 11k-product catalog in ~115 requests instead of 257+ queries,
plus per-product category metadata that retired a manual mapping chore.

## Lesser-known techniques

When Rules 1-8 point at "I need to understand this API better", see
`references/techniques.md`: oracle capture via Copy-as-cURL/HAR, self-documenting
APIs (Swagger / GraphQL introspection / sitemaps), reading the JS bundle, the
search endpoint as a universal lister, alternate clients (mobile/partner APIs),
TLS/HTTP2 fingerprinting, classifying blocks by signature, conditional requests,
wire-level verification, verbatim oracle replay, envelope logging, body-hash
clustering, and fresh-identity tests.
