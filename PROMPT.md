# Weekly Regulatory Change Briefing — U.S. Banks & Credit Unions

You are running the weekly regulatory change briefing for U.S. financial services. This file (PROMPT.md) describes the full routine — follow each step exactly. The wrapper routine has already configured the git remote with a PAT before you read this file; if it hasn't, stop and surface the failure.

## Goal

Produce a weekly briefing covering every material regulatory change that occurred in the **prior 7 days** and would affect U.S. banks or credit unions. Each run:

1. Confirms today's date and computes the prior-week window.
2. Loads memory of items already reported in past weeks (de-duplication).
3. Sweeps every source listed below — federal, GSE/housing, payments, SEC, plus all 50 states + DC + 5 territories.
4. Captures: new proposed regulations, finalized regulations, near-final laws, news, announcements, updates to manuals/exam materials, enforcement actions, and any other material change banks or credit unions should track.
5. Scores each item 1–10 for impact (10 = very high, 1 = inconsequential).
6. Writes a markdown briefing sorted highest-impact first.
7. Appends a row per item to a cumulative master Excel file (`regulatory-changes.xlsx`).
8. Updates memory.json so next week's run skips items already reported.
9. Commits and pushes everything.

---

## Step 0 — Confirm context

Run:
```bash
date +%Y-%m-%d
git status
git pull --rebase origin main
```

Capture today's date as `RUN_DATE` (YYYY-MM-DD). The routine is scheduled for Friday at 00:00 local. The week being covered is the 7 days ending **the day before** RUN_DATE.

Compute the window:
```bash
WINDOW_END=$(date -d "yesterday" +%Y-%m-%d)
WINDOW_START=$(date -d "7 days ago" +%Y-%m-%d)
```
Use these as `WINDOW_START` and `WINDOW_END` for all filtering.

---

## Step 1 — Load past-week memory (de-duplication)

Read `memory.json`. Schema:
```json
{
  "reported_items": [
    {"id": "url-or-hash", "first_seen_week": "YYYY-MM-DD", "title": "..."}
  ]
}
```
If the file is missing or malformed, treat as empty (`{"reported_items": []}`). Do not include any item whose URL or content-hash matches an existing entry. Build a Python `set` of ids upfront so each candidate item is checked in O(1).

---

## Step 2 — Sweep federal sources

Search each source's news / press release / proposed rules pages for items dated within `[WINDOW_START, WINDOW_END]`. For ambiguous dates, prefer the publication or release date over the effective date.

### Banking prudential regulators
- **Federal Reserve (Fed)** — https://www.federalreserve.gov/newsevents/pressreleases.htm
- **OCC** — https://www.occ.gov/news-issuances/news-releases/index.html
- **FDIC** — https://www.fdic.gov/news/press-releases/index.html
- **NCUA** — https://www.ncua.gov/newsroom
- **CFPB** — https://www.consumerfinance.gov/about-us/newsroom/
- **FFIEC** — https://www.ffiec.gov/press.htm

### AML, securities, payments
- **FinCEN** — https://www.fincen.gov/news/news-releases
- **SEC** — https://www.sec.gov/news/pressreleases
- **NACHA** — https://www.nacha.org/news

### Housing & GSEs
- **HUD** — https://www.hud.gov/press
- **FHA (single-family policy)** — https://www.hud.gov/program_offices/housing/sfh/SFH_POLICY_PUBS
- **Fannie Mae** — https://www.fanniemae.com/newsroom
- **Freddie Mac** — https://www.freddiemac.com/about/news
- **Sallie Mae** — https://news.salliemae.com/press-releases
- **FHFA** (regulator of Fannie / Freddie) — https://www.fhfa.gov/news

### Cross-cutting
- **Federal Register — agency rules & proposed rules** — https://www.federalregister.gov/agencies (filter by the agencies above)
- **CSBS** (multi-state coverage) — https://www.csbs.org/newsroom
- **NASCUS** (state credit-union news) — https://www.nascus.org/news

---

## Step 3 — Sweep state regulators (all 50 + DC + territories)

For every jurisdiction below, search the state banking department / financial regulator for material items in the window. If a jurisdiction has nothing material, note it under "Quiet jurisdictions" — do not pad.

**States (50)**: AL, AK, AZ, AR, CA, CO, CT, DE, FL, GA, HI, ID, IL, IN, IA, KS, KY, LA, ME, MD, MA, MI, MN, MS, MO, MT, NE, NV, NH, NJ, NM, NY, NC, ND, OH, OK, OR, PA, RI, SC, SD, TN, TX, UT, VT, VA, WA, WV, WI, WY.

**District**: DC.

**Territories**: Puerto Rico, U.S. Virgin Islands, Guam, American Samoa, Northern Mariana Islands.

For each jurisdiction, run a focused web search such as:
- `"<state> department of banking" OR "<state> financial regulation" news <WINDOW_START>..<WINDOW_END>`
- `"<state> credit union" regulation OR rule <WINDOW_START>..<WINDOW_END>`

Prioritize the regulator's own newsroom over secondary coverage. Cross-reference CSBS and NASCUS for state items aggregated nationally.

---

## Step 4 — Classify and score impact

For each candidate item, capture:
- **Title** — concise, plain-English (no marketing language, no all-caps).
- **Summary** — 2–4 sentences: what changed, who's affected, effective / comment-close date.
- **Type** — Proposed Rule / Final Rule / Law / Guidance / Manual Update / Exam Update / Enforcement / News-Announcement.
- **Jurisdiction** — `Federal` or specific state / territory name.
- **Regulator** — e.g. OCC, NY DFS, NCUA.
- **Link** — canonical source URL (the regulator's page, not a press aggregator).
- **Priority (1–10)** using this rubric:
  - **10** — Final rule with broad applicability and a near-term compliance date affecting most banks or credit unions.
  - **8–9** — Proposed rule from a prudential regulator with broad scope; major exam-manual or call-report change.
  - **6–7** — Targeted proposed rule, significant guidance, GSE selling/servicing-guide bulletin with operational impact.
  - **4–5** — Narrow guidance, single-state law of moderate scope, FAQs, technical updates.
  - **2–3** — Minor announcements, speeches, routine reminders.
  - **1** — Inconsequential / informational.

Items scoring **below 2** should be omitted from the briefing entirely (but you may keep them out of memory.json so a later, higher-impact follow-up isn't suppressed).

Be conservative with priority 10. Reserve it for items most banks or credit unions would actually need to act on within a few months.

---

## Step 5 — De-duplicate against memory.json

Drop any item whose URL is already in `reported_items`. For items where URL alone is ambiguous (e.g. a "news roundup" hub), compute the id as `sha256(regulator + "|" + title)`.

---

## Step 6 — Write the briefing

Write to `briefings/<RUN_DATE>.md`. Sort all items **descending by priority**, ties broken by jurisdiction (Federal first, then alphabetical state). Format:

```markdown
# Weekly Regulatory Change Briefing — Week of <WINDOW_START> to <WINDOW_END>

_Run date: <RUN_DATE>. <N> items captured across <X> federal sources and <Y> states / territories._

## Top of the week
- Three bullets covering the highest-priority items, each with a link.

## Items (priority order)

### 1. [Priority 10] <Title> — <Regulator>, <Jurisdiction>
- **Type**: Final Rule
- **Date**: YYYY-MM-DD
- **Summary**: ...
- **Link**: <url>

### 2. [Priority 9] ...

...

## Quiet jurisdictions
One line listing every state / territory with no material items this week.

## Sources swept
Compact table: source → items found.

## Sources unreachable
Any source you couldn't load (URL + reason). If empty, omit this section.
```

---

## Step 7 — Append to the cumulative Excel

The master file is `regulatory-changes.xlsx` in the repo root. One sheet named `Changes`.

Columns (in order):

| Week-Of | Title | Summary | Link | Jurisdiction | Regulator | Priority |

- `Week-Of` = `WINDOW_END` (the day-before date used as the week-ending Thursday).
- Append rows already sorted by priority descending so the sheet is readable as-is.

Use Python with openpyxl. If openpyxl isn't installed, run `pip install --quiet openpyxl` first.

```python
import os
from openpyxl import Workbook, load_workbook

PATH = "regulatory-changes.xlsx"
HEADERS = ["Week-Of", "Title", "Summary", "Link", "Jurisdiction", "Regulator", "Priority"]

if os.path.exists(PATH):
    wb = load_workbook(PATH)
    ws = wb["Changes"] if "Changes" in wb.sheetnames else wb.create_sheet("Changes")
    if ws.max_row == 1 and ws.cell(1, 1).value is None:
        ws.append(HEADERS)
else:
    wb = Workbook()
    ws = wb.active
    ws.title = "Changes"
    ws.append(HEADERS)

# `items` = priority-desc-sorted list of dicts produced in Step 4
for it in items:
    ws.append([
        WINDOW_END,
        it["title"],
        it["summary"],
        it["link"],
        it["jurisdiction"],
        it["regulator"],
        it["priority"],
    ])

wb.save(PATH)
```

---

## Step 8 — Update memory.json

Append every new item's identifier to `reported_items`:
```json
{"id": "<url-or-hash>", "first_seen_week": "<RUN_DATE>", "title": "<title>"}
```
Then trim entries older than 26 weeks to bound the file size — six months of de-dup memory is plenty.

---

## Step 9 — Commit and push

```bash
git add briefings/<RUN_DATE>.md regulatory-changes.xlsx memory.json
git commit -m "Weekly briefing <WINDOW_START> to <WINDOW_END>"
git push origin main
```

If the push fails for any reason, surface it loudly at the end of your output. Next week's run depends on this commit landing — without it, de-duplication breaks.

---

## Quality bar

- Every item must have a working link to the primary source. No paraphrased summaries without a citation.
- Prefer primary regulator pages over news aggregators.
- A speech or blog post is not a regulation; mark those Type=News-Announcement and score accordingly (usually 2–3).
- If a source is unreachable, list it under "Sources unreachable" rather than silently skipping it.
- Don't pad. A quiet week is fine — say so.
