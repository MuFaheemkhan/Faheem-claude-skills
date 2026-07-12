# Lesser-known techniques

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
