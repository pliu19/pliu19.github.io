---
name: update-conferences
description: Refresh conferences.html with current CFP deadlines, conference dates, and locations for the ML/IR/data-mining venues tracked on the site. Use when asked to update, refresh, or re-verify the conference deadlines page, add a venue to it, or check whether any listed deadline has changed.
---

# Update the conference deadlines page

`conferences.html` is a hand-maintained calendar of submission deadlines and
conference dates. It goes stale on its own, so this skill re-verifies it against
primary sources and rewrites the two tables.

## Ground rules

1. **Primary sources only.** Verify every date against the venue's own site —
   `kdd2027.kdd.org`, `iclr.cc/Conferences/2027/Dates`, `acmweb2027.org`,
   `wsdm-conference.org/2027/`, `aclrollingreview.org/dates`, and so on.
   Aggregators (aideadlin.es, mldeadlines.com, conferencedeadlines.com,
   getpaperpilot.com, myhuiban.com) are useful for *finding* a venue's current
   site, and nothing else. They routinely serve cached or predicted dates
   dressed up as confirmed ones, and they cross-contaminate venues — one pass
   had AISTATS 2027 listed at AAAI-27's dates and location.
2. **Never silently promote an estimate to a fact.** Every date on the page is
   either confirmed against an official CFP or wrapped in `<span class="est">`.
   If the official page says "dates coming soon", it stays estimated no matter
   what an aggregator claims.
3. **Estimates come from the same venue's previous cycle**, not from a general
   sense of when things are due. If KDD 2027 Cycle 2 is unannounced, look up
   KDD 2026 Cycle 2 and shift by a year. Say so in the Notes section.
4. **Regions are not locations.** ICML 2027 ("South America") and NeurIPS 2027
   ("Europe") have announced regions with no city. Write the region and mark
   the city TBA rather than inventing one.

## Procedure

1. Read `conferences.html` to see what is currently listed and what the previous
   verification date was.
2. Drop rows whose deadline **and** conference date have both passed. A venue
   whose deadline passed but that has not yet met stays in the calendar table.
3. For each remaining venue, fetch its official site and confirm: abstract
   deadline, full paper deadline, conference dates, location. Batch the fetches —
   run several `WebFetch`/`WebSearch` calls in one message rather than serially.
4. Roll the horizon forward so the deadline table always covers roughly the next
   twelve months. When a cycle closes, add the following year's edition as an
   estimate (e.g. once ICLR 2027's deadline passes, add ICLR 2028 estimated from
   ICLR 2027's dates).
5. Rewrite both tables. Keep them **sorted**: the first table by
   `data-deadline`, the second by conference start date.
6. Refresh the **ARR cycle table** from `aclrollingreview.org/dates`, which is
   authoritative for every ACL-family venue. Do not try to derive an ACL, EMNLP,
   NAACL, EACL, or COLING deadline from the conference's own site — those
   pages restate a cycle date and sometimes disagree with ARR by a few days on
   commitment. The ARR page also tells you which venues each cycle feeds, which
   is how you discover pairings like NAACL 2027 and COLING 2027 sharing the
   October 2026 cycle.
7. Update the Notes section — it explains two-cycle venues (KDD), ARR-mediated
   venues, rolling-round venues (ICWSM), venues whose host city is announced only
   at the preceding conference (IJCAI), and region-announced-only venues.
   Correct anything that has moved.
8. Update the trailing "Verified against official calls for papers on
   &lt;date&gt;" line to today's date.

## Page mechanics worth preserving

- Every row in `#deadline-table` carries `data-deadline="YYYY-MM-DD"` holding the
  **operative** deadline — the abstract deadline where one binds the paper slot,
  otherwise the paper deadline. For an estimate, still supply a concrete ISO date
  so the row sorts and highlights correctly, and mark the visible text as an
  estimate.
- The inline script at the bottom greys out past rows (`is-past`) and highlights
  rows due within 30 days (`is-soon`, plus a day-count badge). It reads
  `data-deadline`, so **the page ages correctly on its own between updates** —
  do not replace this with hardcoded styling.
- Estimated text uses `<span class="est">`, which renders italic grey with a
  leading `~`. Do not write a literal `~`; the CSS supplies it.
- The page follows the site template: `w3-include-html` for navbar and footer,
  Bootstrap 3, `css/main.css`, and the `$("li#conferences a").addClass("active")`
  nav-highlight block. Match `research.html` if anything structural is unclear.

## Scope

The page tracks venues relevant to five focus areas: **retrieval, web search,
recommender systems, NLP, and agentic systems**. Use that as the test when
deciding whether a newly announced venue belongs.

Do **not** filter by CCF or CORE tier. It was considered and rejected — every
strict reading cuts CIKM, which is CCF-B and CORE-A despite being a venue that
matters here. Breadth within the focus areas beats ranking purity.

Venues currently tracked:

- **IR / recsys / web**: SIGIR, SIGIR-AP, CIKM, WSDM, WWW, RecSys
- **Data mining**: KDD, ICDM
- **General ML / AI**: NeurIPS, ICML, ICLR, AISTATS, AAAI, IJCAI
- **NLP**: ACL, EMNLP, NAACL, EACL, COLING, COLM
- **Agentic systems**: AAMAS (Generative and Agentic AI track), COLM
- **Social / web science**: ICWSM

Add a venue when asked; do not expand the list unprompted. Deliberately **out of
scope** — do not re-add these when refreshing, even though they sit adjacent to
the tracked venues and aggregators will surface them: ECIR, CHIIR, AACL-IJCNLP,
and the CV conferences (CVPR/ICCV/ECCV).

## Finishing

Validate that the HTML is well-formed and that `data-deadline` values are in
ascending order before committing. A quick check:

```bash
python3 - <<'EOF'
import re
s = open('conferences.html').read()
rows = re.findall(r'data-deadline="([\d-]+)"', s)
print(len(rows), "rows; sorted:", rows == sorted(rows))
EOF
```

Then commit and push to `master` — the site deploys from that branch.
