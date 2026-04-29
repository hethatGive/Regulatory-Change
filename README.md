# Regulatory-Change

Weekly regulatory change tracker for U.S. banks and credit unions.

The routine runs every Friday at 00:00 and produces:

- `briefings/<run-date>.md` — a markdown briefing of every material regulatory change in the prior 7 days, sorted by impact priority (1–10).
- `regulatory-changes.xlsx` — a cumulative master spreadsheet, one row per item, with a `Week-Of` column for filtering across weeks.
- `memory.json` — index of items already reported, used to de-duplicate next week's run.

Sources swept each week:

- **Federal**: Fed, OCC, FDIC, NCUA, CFPB, FFIEC, FinCEN, SEC, NACHA, HUD, FHA, FHFA, Fannie Mae, Freddie Mac, Sallie Mae, Federal Register.
- **State**: all 50 states + DC + 5 territories (PR, USVI, Guam, American Samoa, CNMI).
- **Cross-cutting**: CSBS, NASCUS.

See [PROMPT.md](PROMPT.md) for the full routine the agent executes.
