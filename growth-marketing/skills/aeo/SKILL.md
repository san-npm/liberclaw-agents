# Skill: AEO — Answer Engine Optimisation

Optimising for **featured snippets**, **People Also Ask**, **voice assistants** (Siri, Alexa, Google Assistant), and **zero-click** surfaces. AEO lives between SEO and GEO — the search engine still picks the winning answer from indexed pages, but it's rendering that answer directly instead of sending a click.

Zero-click is not a loss when it's a brand impression on a high-intent query — track it.

---

## Snippet types and how to win each

### Paragraph snippet (most common)
- Direct answer in the first 40–60 words, immediately after the H2/H3 that restates the question
- Define the term explicitly: "X is …" / "Y refers to …"
- Avoid pronouns and context-dependent phrasing ("this", "that", "as mentioned")

```markdown
## What is a Service Level Agreement?

A Service Level Agreement (SLA) is a contract between a service provider and
a customer that specifies performance guarantees, measurement methods, and
remedies when the provider fails to meet them. Common SLA metrics include
uptime percentage, response time, and resolution time.
```

### List snippet (ordered — "how to", "steps")
- Use a clear H2 with "how to …" or "steps to …"
- Sub-headings or `<ol>` with concise 8–15-word steps
- Number the steps explicitly in the heading ("Step 1: …")

### List snippet (unordered — "types of", "best", "examples")
- H2 phrased as the question
- `<ul>` of concise items, each on its own line
- Avoid overly long bullets (cut off in the snippet)

### Table snippet (comparisons, specs, prices)
- Real `<table>` (not divs styled as tables)
- First row = `<th>` headers
- Works best for "X vs Y", "prices", "dimensions", "compatibility"

### Video snippet
- Clear chapters / timestamps in the YouTube description
- Transcript on-page
- Question-format title

---

## People Also Ask (PAA) capture

PAA boxes expand — getting cited in the expansion is high-impact because it drives impressions and brand exposure even without a click.

**Discovery:**
- SerpApi / ScraperAPI `google_search` with `include_paa=true`
- `https://alsoasked.com/` (limited free tier) for question graphs
- Manual Google search in incognito, note the PAA questions and their expansions

**Capture tactic:**
1. Collect 20–40 PAA questions for your main target query
2. Cluster by intent (informational / comparative / commercial)
3. For each page on your site, add a dedicated **FAQ section** covering 5–8 of the most relevant PAA questions
4. Mark up with `FAQPage` schema — still rendered by Google for non-YMYL / non-government topics
5. Answer concisely (40–60 words per answer) — longer answers rarely win the snippet

```json
{
  "@context": "https://schema.org",
  "@type": "FAQPage",
  "mainEntity": [{
    "@type": "Question",
    "name": "What is a Service Level Agreement?",
    "acceptedAnswer": {
      "@type": "Answer",
      "text": "A Service Level Agreement (SLA) is a contract…"
    }
  }]
}
```

---

## Schema (the AEO lever)

Schema doesn't rank your page — it enables rich result formats.

| Schema type | Use for |
|---|---|
| `FAQPage` | Q&A sections — declining coverage since 2023 but still rendered for many queries |
| `HowTo` | Step-by-step guides (recipe, fix, install). Restricted to some verticals. |
| `Article` / `BlogPosting` / `NewsArticle` | Editorial content. `author` with `sameAs` LinkedIn is now meaningful. |
| `Product` + `Offer` + `AggregateRating` | E-commerce PDPs (review stars in SERP) |
| `Review` | Standalone reviews |
| `Recipe` | Full recipe rendering in Google |
| `Event` | Dates, venues, tickets |
| `VideoObject` | Video chapter markup |
| `Organization` + `WebSite` + `sameAs` | Knowledge-panel and entity-graph signal |
| `BreadcrumbList` | Better SERP display + entity signalling |

**Validation:** `https://validator.schema.org/` and Search Console → Rich Results Test. Pages with schema errors get silently dropped from rich results.

---

## Voice query optimisation

Voice queries are conversational and question-based. Typing "best restaurants LA" vs. asking "what are the best restaurants in Los Angeles open now on a Tuesday night".

- Long-tail, full-sentence H2s
- Natural-language answer blocks (no "see below" / "as detailed above")
- Mobile speed (voice is usually mobile-initiated)
- Local schema (`LocalBusiness`, `address`, `openingHoursSpecification`) for voice + local answers
- The featured-snippet answer is what voice assistants usually read — if you win the snippet, you win the voice query

---

## Zero-click strategy

Zero-click impressions still have value when:
- The user sees your brand name in the snippet or PAA answer
- The answer requires context they'll follow up on (the snippet whets the appetite)
- Your entity appears in a knowledge panel — strongest brand moat long-term

Track zero-click impressions in Search Console as `impressions − clicks` on branded + high-intent queries. Rising impressions with stable clicks = winning brand surface area.

---

## AEO audit checklist

```
AEO AUDIT — {DOMAIN}
====================
Audit date: {UTC date}

SNIPPET-READY STRUCTURE (sample: {N} target queries)
- Direct-answer blocks in first 60 words: {N/N pages}
- Question-format H2s: {N/N}
- Numbered steps for "how to" queries: {N/N}
- Comparison tables where intent is comparative: {N/N}

SCHEMA DEPLOYED
- FAQ: {pages with + validator pass/fail}
- HowTo: {pages with + validator pass/fail}
- Article/author with sameAs: {pages with}
- Product+Review (if e-comm): {pages with}
- Organization + sameAs on homepage: {yes/no}

PAA CAPTURE
- PAA questions discovered for top 10 target queries: {total count}
- PAA questions currently answered on-site: {count}
- Gap list: {questions to cover}

VOICE READINESS
- Mobile CWV passing: {yes/no}
- Conversational H2s on target pages: {N/N}
- LocalBusiness schema (if local-relevant): {yes/no}

ZERO-CLICK TRACKING
- Branded impression trend (90d): {up/flat/down}
- High-intent non-branded impression-minus-click trend: {up/flat/down}

PRIORITISED FIXES
-----------------
1. [impact: HIGH, effort: S] Add FAQ schema + block on {top 5 pages} — metric: rich result impressions
2. [impact: HIGH, effort: M] Restructure {pillar page} with direct-answer block for primary query — metric: featured snippet ownership
3. ...
```

---

## Current trends (cite source + date when you invoke these)

- **FAQ rich results scope narrowed** (Google 2023-08) — now shown mainly for authoritative government + health sites. Still mark up; rendering is Google's call.
- **AI Overviews** (launched May 2024, expanded 2025) overlap heavily with featured snippets. Pages optimised for AEO tend to be cited in Overviews too — one of the stronger AEO→GEO bridges.
- **INP replaced FID** (March 2024) — voice/mobile queries are especially sensitive to interaction latency.

Always caveat: "Google changes this frequently — verify before acting on a high-stakes rollout."
