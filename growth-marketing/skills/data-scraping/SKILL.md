# Skill: Data Scraping

Compliant, rate-limited, defensible data collection for marketing work: SERP capture, competitor content audits, backlink discovery, pricing monitoring, social listening.

**Non-negotiables:** respect `robots.txt`, honour site ToS, identify with a truthful user-agent, rate-limit, back off on errors, never harvest PII without a lawful basis, never republish copyrighted content. If a site's ToS forbids scraping and no API exists, surface that to the user and stop.

---

## Decision tree — before you scrape

```
Is there an official API?                       → use the API
   ↓ no
Is there a paid data provider (Ahrefs, SerpApi,
Semrush, Similarweb, BuiltWith) that covers it?  → use the provider
   ↓ no (or cost-prohibitive)
Does the site's ToS prohibit scraping?           → stop, surface to user,
                                                   propose lawful alternatives
   ↓ no
Does robots.txt disallow the path?               → do not crawl that path
   ↓ no
Is the content behind auth?                      → do NOT bypass auth
   ↓ no
Is the data PII?                                 → require explicit lawful basis
                                                   (GDPR / CCPA) before proceeding
   ↓ no
Proceed — with rate limits, UA, caching, backoff
```

Always log the decision with date + reasoning. If challenged, you want a paper trail.

---

## robots.txt check

```bash
curl -fsS --max-time 10 -A "GrowthMarketingAgent/1.0 (+mailto:ops@example.com)" \
  "${DOMAIN}/robots.txt"
```

Parse for:
- `User-agent: *` default rules
- `User-agent: YourBotName` specific rules (if you've declared one)
- `Disallow:` paths
- `Crawl-delay:` — honour it even if long
- `Sitemap:` — use for scoped crawling

If a `Disallow` matches your intended path, do not fetch it — even if the URL is linkable.

---

## ToS check

Scrape with the Terms of Service, not against them. Common restrictive clauses:

- Google, Bing, LinkedIn, X/Twitter, Instagram, Facebook, TikTok: **scraping logged-in or gated pages is prohibited** and actively litigated
- Amazon: API only for pricing; scraping product pages at scale is forbidden
- Crunchbase: API only
- Many news publishers: prohibited, but also covered by copyright beyond fair use

If a site publishes a `TOS` URL and explicitly forbids scraping, the compliant path is:
- Use the official API
- Use a licensed data provider (SerpApi, ScraperAPI for SERPs; Similarweb, BuiltWith for traffic / tech stack)
- Ask the user if they have a commercial agreement
- For competitive intel, consider manual one-time capture vs. automated scraping

---

## SERP capture — always via an API, never direct

Google's ToS explicitly prohibits automated SERP scraping. Don't.

Use:
- **SerpApi** — `https://serpapi.com/search.json?engine=google&q={Q}&api_key={K}`
- **ScraperAPI** — `https://api.scraperapi.com/?api_key={K}&url=...`
- **ValueSerp**, **DataForSEO** — alternatives

```bash
# SerpApi — Google SERP
curl -fsS --max-time 30 -G \
  --data-urlencode "engine=google" \
  --data-urlencode "q=${QUERY}" \
  --data-urlencode "location=${LOCATION}" \
  --data-urlencode "gl=${COUNTRY_CODE}" \
  --data-urlencode "hl=${LANGUAGE}" \
  --data-urlencode "include_paa=true" \
  --data-urlencode "api_key=${SERPAPI_KEY}" \
  https://serpapi.com/search.json
```

Cache by `(query, location, country, language, date)`. SERPs shift daily — cache TTL ≤ 24h for volatile queries.

---

## Page-level scraping (allowed sites only)

Polite scraping template:

```bash
scrape_url() {
  local url="$1"
  local ua="GrowthMarketingAgent/1.0 (+mailto:ops@example.com)"

  for delay in 1 2 4 8 16; do
    resp=$(curl -sS --max-time 30 -A "$ua" \
      -H "Accept: text/html,application/xhtml+xml" \
      -w "\n%{http_code}" \
      "$url")
    code="${resp: -3}"
    body="${resp::-4}"

    case "$code" in
      200) echo "$body"; return 0 ;;
      403|401) echo "BLOCKED $url" >&2; return 1 ;;
      429|5??) sleep $((delay + RANDOM % 3)) ;;  # backoff with jitter
      *) echo "HTTP $code on $url" >&2; return 1 ;;
    esac
  done
  echo "Gave up on $url after retries" >&2
  return 1
}

# Respect a polite base rate: 1 req / 2s
for url in "${URLS[@]}"; do
  scrape_url "$url" | ...
  sleep 2
done
```

Rules of thumb:
- Max **1 request per 2 seconds** per origin unless the site's `Crawl-delay` says slower
- Concurrency = 1 per origin. Parallelise across origins only, never within.
- Cache by URL for the session (and persist to disk if running multi-day)
- Honour HTTP cache headers (`ETag`, `Last-Modified`) — send `If-None-Match` and skip on 304

---

## JS-rendered content

Static HTML misses React/Next/Vue apps that render client-side. Options:
- **Headless Chrome** (`playwright`, `puppeteer`) — slow, expensive, stealth-prone to detection
- **Prerender.io / SplashAPI / ScrapingBee / ScraperAPI** — managed headless rendering
- **Inspect the network tab** — the page almost always calls an internal JSON API. Scraping that API is cleaner than rendering the page (though the same ToS / rate-limit rules apply)

If a site specifically fights headless-browser scraping (Cloudflare challenges, Imperva, Akamai Bot Manager), that's a signal the site does not want automated access. Respect that.

---

## Extraction patterns

```bash
# Title + H1
pup 'title text{}' < page.html   # https://github.com/ericchiang/pup
pup 'h1 text{}' < page.html

# All links
pup 'a attr{href}' < page.html

# Schema JSON-LD
pup 'script[type="application/ld+json"] text{}' < page.html | jq .

# Meta description
pup 'meta[name="description"] attr{content}' < page.html
```

For structured data, always extract from `<script type="application/ld+json">` blocks — it's the clean source of truth.

---

## LLM-answer capture (GEO monitoring)

See `geo/SKILL.md` for the API calls. Record: query, engine, date, cited domains, cited URLs, answer summary. Never publish raw LLM answers as your own content — copyright / attribution problems.

---

## Backlink discovery

Authoritative sources:
- **Ahrefs API** (`https://api.ahrefs.com/v3/`) — gold standard, paid
- **Semrush API** — comparable
- **Moz Link Explorer API**
- **Majestic API**
- **Google Search Console → Links report** — your own site only; free, authoritative for links Google has actually indexed

GSC for your own site + one of (Ahrefs / Semrush) for competitors is the standard combination.

---

## Social / community monitoring

Compliant paths:
- **Reddit API** (OAuth, rate-limited): `https://oauth.reddit.com/r/{subreddit}/search.json?q={Q}&limit=100`
- **X/Twitter API** — now paid, rate-limited at the basic tier
- **Bluesky API** — open, keyless for public posts
- **Mastodon / Fediverse** — per-instance APIs, generally permissive
- **YouTube Data API** — comments, video metadata
- **LinkedIn** — API is gated; do NOT scrape LinkedIn UI, it is explicitly prohibited and actively enforced

Never scrape DMs, private groups, or gated content.

---

## PII and GDPR / CCPA

If scraped data contains personal identifiers (names, emails, phone numbers, physical addresses, social handles tied to real identities):
- You need a lawful basis — almost always Article 6(1)(f) legitimate-interest for B2B, documented in a Legitimate Interest Assessment (LIA)
- Inform data subjects if you contact them (Article 14 within 30 days)
- Never persist PII into long-term memory without the user's explicit instruction + lawful basis documented
- Strip PII from aggregated reports unless needed
- Honour Do-Not-Track / right-to-erasure requests

For consumer (CCPA) data, similar principles apply — consumer right to know, delete, opt-out.

---

## Fallback hierarchy when a scrape fails

1. Try the official API if one exists
2. Try an alternate data source (Ahrefs if Semrush fails, Blockstream if blockchain.info fails)
3. Try SerpApi / ScraperAPI (for SERP or JS-heavy targets)
4. Try a manual one-off fetch via browser (document the finding, don't automate)
5. Surface the failure to the user — say why, propose an alternative

Never silently fabricate data when a scrape fails.

---

## Output template

```
SCRAPE REPORT — {TARGET}
========================
Date: {UTC}
Compliance check:
  - robots.txt: {allowed/disallowed/partial}
  - ToS: {permitted / unclear / forbidden}
  - API alternative: {yes/no — name}
Method: {direct / SerpApi / ScraperAPI / headless / API}
User-agent: {string}
Rate: {N req / sec, backoff policy}
Pages fetched: {N}
Errors: {N — breakdown by status code}
Cache hits: {N}
PII handling: {none collected / present, LIA filed / present, stripped before storage}

DATA SUMMARY
------------
{table or structured extraction}

CONFIDENCE
----------
HIGH if: source is authoritative + data is structured + no silent failures
MED if: one of the above is weak
LOW if: headless scraping, anti-bot signals, partial coverage

LIMITATIONS
-----------
{rate-limit hit? partial pagination? JS content that didn't render?}
```
