# Skill: SEO

Organic Google/Bing search playbook — technical, on-page, topical authority, and link equity. No black-hat. No vanity. Every recommendation labelled with confidence and the metric to watch.

---

## SEO Stack (in audit order)

### 1 · Indexability
If Google can't crawl or index a page, nothing else matters.

- `robots.txt` — check `Disallow` rules against what should be indexed. Test via Search Console robots.txt tester.
- `meta robots` + `X-Robots-Tag` header — look for accidental `noindex` on commercial pages (a common post-deploy regression).
- `canonical` — every URL points to one canonical. Self-referencing canonicals are correct for unique pages. Cross-domain canonicals are valid when intentional.
- `sitemap.xml` — submitted to Search Console, updated on deploy, no 404s / redirects / non-canonical URLs inside.
- `hreflang` — if multi-region, reciprocal tags, valid language-region codes, self-referencing.
- `pagination` — `rel=next/prev` is ignored by Google since 2019; use clean canonicals + clear internal linking instead.
- **Crawl budget** (large sites): review server logs for Googlebot hit patterns. Block faceted/parameter URLs that waste budget.

Commands:
```bash
# robots.txt
curl -sI "${DOMAIN}/robots.txt" && curl -s "${DOMAIN}/robots.txt"

# meta robots + canonical
curl -s "${URL}" | grep -iE '<meta[^>]+name="robots"|<link[^>]+rel="canonical"'

# X-Robots-Tag header
curl -sI "${URL}" | grep -i x-robots-tag

# Sitemap pull
curl -s "${DOMAIN}/sitemap.xml" | xmllint --format - | head -50
```

### 2 · Core Web Vitals
Ranking factor for Google (since 2021, refined through 2024). User-experience signal first, ranking bonus second.

| Metric | Good | Needs work | Poor |
|---|---|---|---|
| **LCP** (Largest Contentful Paint) | ≤ 2.5s | 2.5–4s | > 4s |
| **CLS** (Cumulative Layout Shift) | ≤ 0.1 | 0.1–0.25 | > 0.25 |
| **INP** (Interaction to Next Paint) — replaced FID in March 2024 | ≤ 200ms | 200–500ms | > 500ms |

Tools:
- **PageSpeed Insights**: `https://pagespeed.web.dev/analysis?url={URL}` — lab + field (CrUX)
- **CrUX API**: `https://chromeuxreport.googleapis.com/v1/records:queryRecord` (needs key) — 28-day real-user data
- **Search Console → Core Web Vitals report** — page-group level field data

Field data beats lab data. If CrUX disagrees with Lighthouse, trust CrUX.

### 3 · On-page
- **Title** (≤ 60 chars rendered) — primary keyword near the front, matches search intent, unique per page
- **H1** — one per page, aligned with title, reflects the user's query
- **Intent match** — informational / commercial / transactional / navigational. A page targeting a commercial keyword with a purely informational format will not rank.
- **Depth** — not word count for its own sake. Answer the question completely. If the top-3 pages average 2,000 words, a 400-word page is unlikely to compete unless it's materially better.
- **Internal linking** — every new page gets ≥ 3 internal links from topically relevant existing pages. Anchor text descriptive (avoid "click here").
- **Structured data** — JSON-LD preferred over microdata. Validate with `https://validator.schema.org/` and Search Console → Rich Results.
- **Images** — `alt` text is accessibility first, SEO side-benefit. Descriptive, not stuffed.

### 4 · Topical authority
Google rewards sites that comprehensively cover a topic, not sites that bolt on one high-value page.

- Pillar-and-cluster model: one pillar page per topic, 10–30 cluster pages linking up/down
- Information-gain score (Google patent language): does this page add anything the top results don't already cover?
- Named author with bio, credentials, proof of expertise — especially for YMYL (Your Money or Your Life) topics
- Original data / studies / surveys outperform rewordings of existing content

### 5 · E-E-A-T
Experience, Expertise, Authoritativeness, Trust. Not a direct ranking signal — an umbrella for many signals.

- Named authors with real bios (LinkedIn link, credentials, past writing)
- About page with verifiable team
- Citations to primary sources (not 10 circular citations to other blogs)
- First-party data and screenshots (experience signal)
- HTTPS, contact info, privacy policy, terms — baseline trust
- Reviews / press coverage / Wikipedia-eligible presence — authoritativeness

### 6 · Link equity
- Referring domains matter more than raw backlinks. 10 DR 60+ domains > 100 DR 20 domains.
- Anchor text — natural distribution. Over-optimised commercial anchors look manipulated.
- Toxic exposure — one spammy campaign can put a site into Penguin-territory. Disavow only if manual action received or clear negative-SEO attack.
- **Digital PR beats link building.** Pitch journalists with original data via HARO / Qwoted / Featured / Connectively.
- **Refuse:** PBNs, link-buying, comment spam, forum link drops, reciprocal link trades. These trigger manual actions and cost more to clean up than they're worth.

### 7 · Pages that win in 2026 tend to have
- Clear entity / topic focus (one intent per URL)
- First-party data or original research
- Named authors with credentials
- Explicit comparison tables with named competitors
- Direct-answer blocks in the first 60 words (AEO overlap)
- Citations to primary sources
- Updated dates + version history (freshness + trust)
- Schema: Article / FAQ / HowTo / Product / Review as appropriate
- Above-fold value (no "grab-before-I-scroll" interstitials)

---

## Audit template

```
SEO AUDIT — {DOMAIN}
====================
Audit date: {UTC date}
Scope: {URLs / sections / full site}

INDEXABILITY
- robots.txt: {findings}
- meta robots / X-Robots-Tag: {findings}
- canonicals: {findings}
- sitemap.xml: {submitted / last-modified / issues}
- hreflang: {if applicable}
- crawl budget: {notes or "not assessed — no log access"}

CORE WEB VITALS (field data, CrUX 28-day)
- LCP: {value} ({good/needs-work/poor})
- CLS: {value}
- INP: {value}
- Top pages failing: {list}

ON-PAGE (sample: {N} URLs)
- Title issues: {count + examples}
- H1 issues: {count + examples}
- Intent mismatches: {examples}
- Thin content (< competitor median - 40%): {list}
- Missing/invalid schema: {list}

TOPICAL AUTHORITY
- Pillar pages: {count, named}
- Orphan pages (no internal links): {count}
- Weak clusters (< 3 supporting pages): {list}

E-E-A-T
- Named authors: {yes/no/partial}
- About page verifiability: {notes}
- YMYL-relevant: {yes/no — if yes, stricter review}

BACKLINKS (source: {Ahrefs/Semrush/Moz/GSC})
- Referring domains: {N}
- DR distribution: {median, p90}
- Toxic exposure: {notes}
- Recent link velocity: {N in last 90d}

PRIORITISED FIX LIST
--------------------
1. [impact: HIGH, effort: S] {action} — metric: {what to watch}
2. [impact: HIGH, effort: M] ...
3. [impact: MED, effort: S] ...
...

OVERALL POSTURE: {strong / needs-work / critical gaps}
90-DAY FOCUS: {1–3 themes}
```

---

## Common pitfalls

- **Content pruning done wrong** — removing pages with impressions but no clicks often hurts more than it helps. Re-optimise first, prune only after 2+ quarters of zero impressions.
- **Migration without redirect mapping** — lose 90% of traffic overnight. Always: 1:1 301 map, run pre-migration crawl, post-migration audit within 48h.
- **Javascript-rendered content** — if the server returns a shell and the content comes from JS, verify Google renders it (`Inspect URL` in Search Console). SSR or pre-rendering for critical content.
- **Duplicate content via parameters / case / trailing-slash** — consolidate via canonical + rewrite rules.
- **AI-generated content without a human layer** — Google's March 2024 and December 2024 core updates specifically targeted unedited mass-produced AI pages. Fine to draft with AI, not fine to publish as-is at scale.

---

## Tooling cheatsheet

| Task | Free | Paid |
|---|---|---|
| Crawl | Screaming Frog (free ≤ 500 URLs) | Sitebulb, OnCrawl, Screaming Frog (full) |
| Keyword research | Keyword Planner, GSC queries, AlsoAsked (limited) | Ahrefs, Semrush, Moz |
| Backlinks | GSC links report | Ahrefs, Semrush, Majestic |
| Rank tracking | GSC position | SerpRobot, AccuRanker, Wincher |
| Core Web Vitals | PageSpeed Insights, CrUX | SpeedCurve, Calibre |
| Schema validation | validator.schema.org, GSC Rich Results | — |
| Log analysis | GoAccess (free CLI) | Botify, OnCrawl |
