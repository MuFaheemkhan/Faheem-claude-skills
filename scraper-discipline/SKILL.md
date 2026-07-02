---
name: scraper-discipline
description: Reverse-engineer and debug hidden APIs behind websites/apps (scrapers, unofficial clients) without deriving false folklore. Use whenever a scraper gets empty/4xx/5xx responses that the real site doesn't, before concluding an endpoint "doesn't support" something, when probing an undocumented API's parameters, or when blaming throttling/bot-protection. Triggers include "scraper broken", "returns empty", "works in the browser but not in curl", "reverse engineer the API", "hidden API", "find the site's API", "Cloudflare/Akamai blocking", "throttled", "the endpoint doesn't support X", "capture the XHR".
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

Reach for these when Rules 1–7 point at "I need to understand this API better."
None are the first thing people try; each collapses a lot of guessing.

**Capture the oracle the fast way — Copy as cURL / HAR.** DevTools → Network →
right-click the request → *Copy as cURL* gives the exact working request with
every header and cookie. *Save all as HAR* captures the whole session to diff
offline. This is Rule 1 in one click; the `window.fetch` spy is the fallback
for when the request is hidden from the Network panel (service worker, early
SSR hydration, wrapped fetch).

**The API often documents itself.** Before reverse-engineering by hand, check
for a machine-readable surface:
- OpenAPI/Swagger at `/swagger.json`, `/openapi.json`, `/v2/api-docs`,
  `/api-docs` — enterprise backends (Hybris OCC, Salesforce Commerce, Spring)
  frequently expose it unauthenticated.
- GraphQL introspection (`{__schema{types{name fields{name}}}}`) when it isn't
  disabled — the entire type graph, for free.
- `sitemap.xml` / `sitemap_index.xml` — a complete list of product/category
  URLs, so you skip tree-walking entirely (and it's on the CDN, not the
  throttled API). `robots.txt` usually points to the sitemaps and sometimes
  leaks internal paths.

**Read the JS bundle, not just the traffic.** The webpack/Vite bundle contains
the API base URLs, the exact param-builder, feature flags, and — crucially —
any request-signing (HMAC/nonce/timestamp) logic. When requests are signed or
a param looks like an opaque hash, beautify the bundle and grep for the
endpoint path or param name to find where it's constructed. Also mine embedded
state before hitting the API at all: `__NEXT_DATA__`, `window.__NUXT__`,
`__INITIAL_STATE__`, Apollo/Redux dumps, and `<script type="application/ld+json">`
often hold the whole page's data already parsed.

**The search endpoint is a universal lister.** A catalog/search endpoint with
an empty, wildcard, or `:relevance` query and a large page size frequently
returns *everything*, bypassing category traversal — exactly the Priceline win.
Try it before building a crawler.

**Alternate clients are less defended.** The mobile app's API (captured via
mitmproxy/Charles with a device cert) or a partner/affiliate/RSS/`.json`
endpoint routinely has weaker bot protection, different auth, and richer
payloads than the public web API. When the web path is a Cloudflare wall, the
app path may be wide open.

**Matching headers isn't enough — the fingerprint is below them.** Bot
protection (Akamai, Cloudflare, PerimeterX) fingerprints the **TLS handshake**
(JA3/JA4) and **HTTP/2 frame + header order/casing**, which stock
`requests`/`httpx`/`aiohttp` normalize in a detectably non-browser way. That's
why identical headers still get 403'd and why `curl_cffi impersonate=` /
`tls-client` exist. If a request works in the browser and fails from Python
with the *same* headers, suspect the fingerprint layer, not your code.

**Classify the block from its signature — don't lump it as "throttling."**
(Extends Rule 4.) Distinct failures need distinct responses:
- WAF challenge: `cf-ray` / `cf-mitigated: challenge` header, HTML body with
  `cdn-cgi/challenge-platform`, error 1020. → needs a real browser/impersonation,
  not backoff.
- Rate limit: `429` + `Retry-After` / `X-RateLimit-Remaining` / `RateLimit-Reset`.
  → read the header and wait exactly that long; don't guess exponential.
- Soft-throttle: `200` with empty/partial body. → back off and retry.
- Origin flap: `5xx`, often intermittent. → retry; not your fault.
A `4xx` (except 429) is almost never transient — it's a real request error;
retrying it just wastes your quota (see `fetch_page` in `priceline_scraper_v2`).

**Be cheap on re-scrapes.** Store `ETag`/`Last-Modified` and send
`If-None-Match`/`If-Modified-Since`; a `304` costs almost nothing and is far
less likely to trip a rate limiter than re-pulling unchanged pages.

**Trust the wire, not your source code.** Your HTTP library silently rewrites
what you think you sent — it reorders and lowercases headers, injects
`Accept-Encoding`/`Connection`, re-encodes query params, drops a cookie across
a redirect. So "my request matches the oracle" is itself an unverified claim
about code, not about bytes. Route your own client through mitmproxy/Burp and
read what actually left the machine. The diff that decides the bug is
oracle-on-the-wire vs you-on-the-wire — never oracle vs your source.

**Replay the oracle verbatim before you mutate.** Run the Copy-as-cURL oracle
unchanged from your environment first. It works → the fault is in how your code
builds the request; bisect down from the known-good cURL (Rule 2). It fails →
the fault is environmental (IP, TLS fingerprint, expired cookie) and no
param-tweaking will ever help. This single replay partitions the whole problem
space before you touch a line of code, and it's the step people skip in their
rush to start changing parameters.

**Disambiguate "empty" by everything except the body.** A block and a
genuinely-empty result often carry byte-identical parsed JSON, but rarely
identical *envelopes*: content-length, response latency, `cf-cache-status`,
`server-timing`, `age`, Set-Cookie churn. Log the raw response envelope beside
the parsed result so an empty is classified at capture time — reconstructing it
later from just a parsed `{}` is exactly the guesswork that let the Priceline
"throttle" folklore survive three sessions.

**Cluster probe responses by (status, size, body-hash), not status alone.**
When sweeping many queries, hash each body. A run of "200s" that share one hash
is a single template — a soft-404, a login wall, a challenge page — wearing N
distinct URLs; real data hashes N different ways. Status codes hide this;
uniform-failure-disguised-as-success only shows up when you fingerprint the
bodies.

**A fresh identity separates stateful from stateless blocks.** When a probe
fails, replay it with a new IP / session / cookie jar. Fresh identity succeeds →
you were flagged or tarpitted (stateful: the block is on *you*, accumulated over
the run — slow down, rotate, or wait it out). Fresh identity also fails → the
block is on the *request* (stateless: wrong params, or a rule against everyone —
mutating identity won't help). The parsed response looks the same for both; only
the swap tells them apart.
