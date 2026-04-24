# Skill: Marketing Analytics

GA4, Search Console, attribution, funnel diagnostics, cohort retention. Mission: turn noise into a confident decision. Always triangulate. Never celebrate a leading metric without a line to a lagging one.

---

## The two types of questions

1. **Monitoring** — "how are we doing?" → dashboards, trends, anomaly detection
2. **Diagnosis** — "why did X happen?" → hypothesis, segmentation, triangulation, test

Most of the value is in type 2. Dashboards tell you something changed. Diagnosis tells you why and what to do.

---

## Source hierarchy (trust order)

1. **Revenue / CRM / Stripe / billing** — money is ground truth. If a funnel looks great but revenue is flat, the funnel is wrong.
2. **Search Console** — impressions and clicks from Google. Unaffected by consent banners (server-side).
3. **Server logs** — unaffected by client-side blocking. Rarely used but invaluable when ad blockers / consent loss distort GA4.
4. **GA4** — user-level behavioural data. Consent-banner-affected; cookieless in EU commonly. Useful for relative trends; absolute numbers usually undercount by 10–40%.
5. **Ad platforms (Meta / Google Ads / LinkedIn)** — self-reported, conflict-of-interest biased. Use for spend + reach; always validate conversions against CRM.

Triangulate any finding across ≥ 2 sources. A finding that shows in only one source is a measurement artefact until proven otherwise.

---

## GA4 essentials

### Event model
GA4 is event-first, not session-first. Every interaction is an event with parameters.

Default events worth trusting:
- `page_view` — page views (automatic)
- `session_start` — session boundary
- `scroll` — 90% scroll (if enhanced measurement on)
- `click` (outbound), `file_download`, `video_*` — enhanced measurement
- `user_engagement` — time in focus

**Custom events** to define for almost every site:
- `form_submit` (with parameter: `form_id`)
- `cta_click` (with parameter: `cta_location`, `cta_label`)
- `signup`, `trial_start`, `purchase` — mark as conversions
- `content_group` (set on every page_view for editorial sites)

Mark true business outcomes as **Key Events** (formerly Conversions, renamed 2024). Attribution, funnel, and ROAS reports use these.

### Custom dimensions
Free tier: 50 event-scoped + 25 user-scoped custom dimensions. Spend them:
- `content_group` — blog / product / pricing / docs / …
- `user_tier` — free / trial / paid
- `signup_source` — organic / paid / referral
- `experiment_variant` — A/B variant
- `geo_city` (if needed — careful with PII)

### UTM hygiene
Every paid / social / email link MUST have UTM:
- `utm_source` (google, meta, linkedin, newsletter)
- `utm_medium` (cpc, social, email, referral)
- `utm_campaign` (dated slug: `2026-q2-launch`)
- `utm_content` (creative variant)
- `utm_term` (keyword, paid search only)

Standardise a taxonomy doc. UTM chaos is the #1 reason attribution reports are unusable.

### Attribution model comparison
GA4 ships:
- **Data-driven** (default since 2023) — ML-based, credit based on observed path contribution
- **Last-click**
- **First-click**
- **Linear**, **Position-based**, **Time-decay**

Compare data-driven vs last-click in the **Model Comparison** report — the delta tells you how much top-funnel you'd be undercrediting under last-click.

Caveat: GA4 data-driven is a black box. For spend > €100k/mo, consider incrementality tests (geo holdouts / MMM) instead.

### GA4 via API
```bash
# GA4 Data API — property-level reports
curl -fsS --max-time 30 \
  -H "Authorization: Bearer ${GA4_ACCESS_TOKEN}" \
  -H "Content-Type: application/json" \
  "https://analyticsdata.googleapis.com/v1beta/properties/${GA4_PROPERTY_ID}:runReport" \
  -d '{
    "dateRanges":[{"startDate":"30daysAgo","endDate":"yesterday"}],
    "dimensions":[{"name":"sessionSourceMedium"},{"name":"pagePath"}],
    "metrics":[{"name":"sessions"},{"name":"conversions"}]
  }'
```

Prefer service-account auth (`GA4_SERVICE_ACCOUNT_JSON`) for automation.

---

## Search Console essentials

GSC is free, authoritative, and under-used. It's the only place you see real Google-side data.

### Reports worth checking weekly
- **Performance → Search results** — query × page × country × device
- **Indexing → Pages** — index coverage, excluded pages, reasons
- **Experience → Core Web Vitals** — field-data CWV by page group
- **Links** — internal + external links Google has indexed

### Performance report hygiene
- Filter by **page path** to isolate sections
- Compare 28-day vs previous 28-day — avoids day-of-week noise
- Always check **impressions** before clicks — a click drop with flat impressions is a CTR problem (title/meta), a click drop with impression drop is a ranking problem
- Filter out brand queries when measuring growth — branded always looks green

### GSC via API
```bash
# Search Analytics query — top queries for last 28d
curl -fsS --max-time 30 \
  -H "Authorization: Bearer ${GSC_ACCESS_TOKEN}" \
  -H "Content-Type: application/json" \
  "https://www.googleapis.com/webmasters/v3/sites/${SITE_URL_ENCODED}/searchAnalytics/query" \
  -d '{
    "startDate":"2026-03-01",
    "endDate":"2026-03-28",
    "dimensions":["query","page"],
    "rowLimit":500
  }'
```

---

## Funnel diagnostics

Standard SaaS funnel:
```
Visit → Signup → Activation → Trial → Paid → Expansion → Retention
```

For each step, know:
- The event that marks it
- The conversion rate to the next step (both gross and by segment)
- The median time-to-next-step
- The leading indicator that predicts drop-off

### Diagnosis pattern: "conversion rate dropped"
1. **Segment** — drop in one segment or all? (source, country, device, cohort week)
2. **Timing** — gradual or cliff? Cliff → deploy / tracking / algo event
3. **Upstream** — did the traffic mix change? (cheaper traffic often converts worse)
4. **Page-level** — which specific step dropped?
5. **Technical** — did the event fire? Check `DebugView` in GA4.
6. **External** — algo update, competitor launch, seasonality

Jump to "fix" only after step 6. Premature fixes waste engineering and usually target the wrong layer.

---

## Cohort retention (SaaS)

Retention curves expose product truth that acquisition metrics hide.

- Define "retained" by active usage, not login — log-in-as-retained flatters inactive users
- Weekly cohorts for early-stage (< 500 signups/mo), monthly for later
- Plot by week-N retention: week 1, week 4, week 12, week 24
- Compare cohorts by signup source — paid vs organic retention is usually different

Red flag: flat-bottom retention curve with > 30% month-12 retention is rare and usually a measurement error.

---

## Anomaly investigation template

```
ANOMALY INVESTIGATION
=====================
Observation: {metric} {direction} {magnitude} {window}
  e.g. Organic clicks −38% week-over-week

STEP 1 — SEGMENT
- By page: {top losers}
- By country: {top losers}
- By device: {top losers}
- By query (GSC): {top losers}

STEP 2 — TIMING
- Gradual over {N} days / cliff on {DATE}
- Events on that date: {deploys, algo updates, competitor launches, ad campaigns paused, tracking changes}

STEP 3 — TRIANGULATE
- GSC impressions: {+/-}
- GSC clicks: {+/-}
- GA4 sessions (source=organic): {+/-}
- Server logs bot traffic: {normal / anomalous}
- CRM pipeline from organic: {trend}

STEP 4 — HYPOTHESIS
{1–3 ranked by likelihood}

STEP 5 — TEST PLAN
{cheap experiment or diagnostic query to confirm}

CONFIDENCE: {HIGH/MED/LOW}
BASIS: {data points that support the ranked hypothesis}
```

Never prescribe a fix before completing steps 1–4. Premature fixes cost more than the anomaly.

---

## Common pitfalls

- **Last-click attribution as truth** — undercredits top-funnel. Always compare models.
- **Consent-banner loss ignored** — post-GDPR / post-iOS14 tracking covers 50–80% of users at best. Don't compare pre-consent-banner periods to post-.
- **Ad-blocker loss ignored** — dev-heavy audiences lose 30–50% of GA4 events. Use server logs to estimate.
- **UTM chaos** — source `meta` vs `facebook` vs `Meta` — normalise in GA4 or use a server-side transform.
- **Currency / timezone mismatch** — GA4 property timezone ≠ CRM timezone produces off-by-one reports. Check before reconciling.
- **Bot traffic not filtered** — GA4's bot filtering is weak. Filter aggressively: known IPs, suspicious countries (if not your market), zero-scroll / zero-engagement sessions.
- **Sampling** — GA4 samples at high cardinality. Use the GA4 BigQuery export for anything quantitative at scale.

---

## Dashboards

Build with **Looker Studio** (free, connects to GA4 + GSC + BigQuery) or **Metabase** (open-source, connects to your DB + Stripe + HubSpot via connectors).

Standard set for an early-stage SaaS:
1. **Acquisition** — sessions / signups by source, weekly cohort
2. **Conversion funnel** — visit → signup → trial → paid by cohort week
3. **Retention** — week-N retention heatmap, by signup source
4. **Revenue** — MRR / ARR, expansion, churn, LTV by cohort
5. **Channel efficiency** — CAC / LTV / payback by paid channel
6. **SEO health** — GSC impressions, clicks, position trend for top 20 pages; CWV pass rate

Keep dashboards focused. If a team stops looking at one, delete it.

---

## What NOT to report

- Vanity metrics without a business-outcome line: followers, likes, time-on-page, bounce rate in isolation
- Percent changes on tiny bases ("+500% on 3 signups → 18 signups!")
- Weekly revenue for businesses with long sales cycles (noise > signal — use monthly or rolling 28-day)
- Last-click as the only view on multi-touch journeys
