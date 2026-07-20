---
name: instantly-campaign-report
description: >
  Generate Instantly campaign performance reports — weekly or monthly — from pasted campaign data tables.
  Use this skill whenever the user pastes campaign data (Sent, Replies, Positive, OOO columns) and asks
  for a report, summary, or write-up. Also triggers when the user says "create a report", "weekly report",
  "monthly report", "campaign performance", "performance summary", or "Instantly report". Always use this
  skill when campaign table data is present and a narrative or document output is requested, even if the
  user doesn't say "skill" or "report" explicitly.
---

# Instantly Campaign Performance Report Skill

Generates narrative performance reports from pasted Instantly campaign data. Supports two report types:
- **Weekly** — campaign-by-campaign narrative with totals summary
- **Monthly** — high-level summary across months with totals

Output is always **text/markdown first**. Only create a `.docx` file if the user explicitly asks for it after seeing the text output. If the user asks for a Slack notification message (or similar send-off message) to accompany the report, use the template in Step 6 — this is a short chat message, never a file.

---

## Step 1 — Identify Report Type

Determine from context:
- **Weekly**: user provides a table with columns Campaign / Status / Sent / Replies / Positive / OOO, and a date range (e.g. "June 29 – July 5")
- **Monthly**: user provides a table with columns Month / Email Sent / Replies, and a date range (e.g. "April through June 2026")

If unclear, ask: "Is this a weekly campaign report or a monthly summary?"

---

## Step 2 — Parse the Data

Extract all rows from the pasted table. Identify:
- Total Sent, Replies, Positive, OOO (from TOTAL row or sum)
- Top campaign by replies
- Top campaign by positive responses
- Top campaign by send volume
- Campaigns with 0 replies
- Completed vs Active campaigns
- Any campaigns worth flagging (high active %, near completion, notable positive responses)
- Progress/active % per campaign (from a Progress % column if present) — needed for the Status column in the doc table and for callouts like "nearly done (94.8%)"

Calculate:
- Reply rate = Replies / Sent × 100 (format as 0.XX%)
- Positive response rate = Positive / Sent × 100 (format as 0.XX%)

---

## Step 3 — Generate the Report

### Weekly Report Format

Follow this structure exactly:

```
Here's the campaign performance summary for [DATE RANGE].

Across all campaigns, we sent [X] emails, received [X] replies, and generated [X] positive responses. This resulted in an overall [X]% reply rate and a [X]% positive response rate.

[MAIN TAKEAWAY SENTENCE — 2-3 sentences summarizing the period. Mention which campaign had the most traction. Note if most campaigns are early in sequence.]

[CAMPAIGN PARAGRAPHS — see rules below]

Overall, [2-3 sentence wrap-up. What to watch. What's nearing completion. What's early-stage.]
```

#### Campaign Paragraph Rules

Do NOT write a paragraph for every campaign. Write paragraphs only for:
1. **Top reply campaign** — always gets a paragraph
2. **Top positive response campaign** (if different from #1) — gets a paragraph
3. **Top send volume campaign** (if different from above) — gets a paragraph only if it has notable context (e.g. near completion, 0 replies despite high volume)
4. **Campaign groups** — group McDonald's / MCD variants into one paragraph. Group BK variants into one paragraph.
5. **Other notable active campaigns** — only if they have replies, positive responses, or are significantly progressed (>25% active status)
6. **Completed campaigns** — group all completed into one paragraph at the end, only if any have notable positive responses or warrant follow-up

Each paragraph starts with the **campaign name in bold**, followed by narrative. Example:

> **BK $50 Flat Fee (new) - Mark Klindera** led in both replies and positive responses this period, generating 3 replies and 1 positive response from 343 sends, along with 1 out-of-office reply. The campaign is now at 72.6% active status, meaning it is well into sequence and approaching completion.

**Do NOT write paragraphs for campaigns with 0 replies, 0 positive, and low active status — unless they are part of a group summary.**

---

### Monthly Report Format

```
Here's the campaign performance summary for [MONTH RANGE] [YEAR].

Over the past [X] months, [X] emails were sent and [X] replies were received, resulting in a [X]% reply rate. [BEST MONTH] had the highest reply rate at [X]%, while [HIGHEST VOLUME MONTH] drove the most volume with [X] sends. [1 sentence on trend — e.g. reply rate vs volume observation.]
```

Keep monthly reports short — 2-3 sentences after the opener. No campaign-level paragraphs needed.

---

## Step 4 — Formatting Rules

- Use **bold** for campaign names at the start of each paragraph
- Use **bold** for all key metrics inline (send counts, reply counts, percentages)
- Keep tone professional but direct — no fluff
- Never write paragraphs for every campaign — only notable ones
- Always end weekly reports with an "Overall" paragraph
- Always start with "Here's the campaign performance summary for [DATE RANGE]."
- For OOO: only mention if > 0
- For completed campaigns with 0 of everything: skip unless grouped

---

## Step 5 — Doc File Format

Default output is still text/markdown in chat first. If the user asks to create a file ("make me a word doc", "create a file", "save this"), build a `.docx` using the format below. Read `/mnt/skills/public/docx/SKILL.md` before writing any file code (docx-js gotchas, page size, verification steps).

### Layout

- No title heading and no date line at the top of the document. The doc starts directly with the opening paragraph: "Here's the campaign performance summary for [DATE RANGE]." (or the monthly equivalent).
- Body paragraphs (opening, takeaway, campaign paragraphs, overall) are plain paragraphs, default font size, with `spacing: { before: 240, after: 240 }`.
- Inline metrics use **bold** (`TextRun({ bold: true })`), same rule as the text-only version.

### Table

One table per reporting period, immediately after the takeaway paragraph (before the campaign paragraphs). Columns match the pasted data: `Campaign | Status | Sent | Replies | Positive | OOO` for weekly, `Month | Email Sent | Replies` for monthly. If the source data has no OOO column, drop it from the table entirely rather than showing zeros.

- **Status column format:** append the campaign's progress/active % directly in the Status cell, e.g. `Active 74.5%`, `Completed 100%`, `Draft 0%` — not just the bare status word.
- Column widths (weekly, DXA, sum 9345): `[3105, 2160, 900, 960, 1110, 1110]` when OOO is present, or `[3735, 2160, 1150, 1150, 1150]` (sum 9345) for the 5-column version without OOO.
- All table text is **10pt** (`size: 20` in docx-js half-point units).
- **Campaign column (and Month column) is left-aligned.** Every other column is center-aligned. This applies to both the header row and data rows.
- Header row: navy fill `#1F3864`, white bold text, centered except the first column (left-aligned).
- Data rows: no shading, thin gray borders (`#CCCCCC`, size 4).
- TOTAL row: light green fill `#D9EAD3`, bold text, same alignment rules as data rows.
- Borders on every cell: thin single, color `#CCCCCC`, size 4.

### Highlight colors for campaign paragraphs

Apply a background shading (`ShadingType.CLEAR`) to the **campaign name plus its immediate lead-in phrase** (e.g. "led in replies this period", "drove the highest send volume") — not the whole paragraph — using this hierarchy:

- **`#93C47D`** (darkest green): a single campaign explicitly leads in *both* replies and positive responses this period ("led in both X and Y").
- **`#B6D7A8`** (medium green): a single campaign is the top performer on one specific metric called out as the headline (e.g. top reply campaign when it doesn't also lead positive).
- **`#D9EAD3`** (light green): reserved for the TOTAL row, and also used for a standalone top-send-volume campaign callout (when that campaign gets its own paragraph rather than being folded into a group paragraph).

Only apply a highlight when a campaign gets its own dedicated paragraph as the headline performer. Group paragraphs (e.g. "McDonald's variants combined for...") and secondary paragraphs (completed campaigns, other notable actives) stay unhighlighted — bold name only, no shading.

If a top-volume or top-positive campaign is folded into a group paragraph instead of getting its own paragraph, do not apply highlight shading there.

### Verify

After generating, convert to PDF and render a page image to confirm: no title/date, table columns aligned as specified, 10pt table text, header navy with white text, TOTAL row green, and highlight shading only on the intended headline campaign(s).

---

## Step 6 — Slack Notification Message

If the user asks for a Slack message, notification, or "send-off" to accompany the report, produce this as **plain chat text, never a `.docx` or `.md` file.** It's a short message meant to be pasted straight into Slack.

Follow this structure exactly:

```
Hi, Happy [DAY OF WEEK]!
Sending over the MFT's [weekly/monthly] campaign report ([DATE RANGE]), Monday Board is also updated

Here's the link for the document report:
[LINK]
```

Rules:
- `[DAY OF WEEK]` — use the actual day the message is being sent (e.g. "Happy Monday!", "Happy Friday!"). If not specified or not inferable, default to "Happy Monday!" since these reports typically go out at the start of the week.
- `[weekly/monthly]` — match the report type just generated.
- `[DATE RANGE]` — same date range used in the report itself (e.g. "June 22-28, 2026").
- "Monday Board is also updated" is a fixed line — always include it, since a Monday.com board update accompanies every report send-off. Don't ask about it or make it conditional unless the user says the board wasn't updated this time.
- `[LINK]` — if the user hasn't provided a document link, leave a placeholder like `[insert doc link]` and ask for it, or leave it blank for the user to paste in themselves. Never invent a URL.
- Keep the tone exactly as casual and brief as the template — no added pleasantries, no sign-off, no extra sentences.

---



## Reference: Sample Weekly Output

See the conversation history or `/references/sample-weekly.md` for a full worked example.
Key patterns:
- Opening line always starts with "Here's the campaign performance summary for"
- Totals paragraph immediately follows
- Main takeaway is 2-3 sentences
- Campaign paragraphs cover groups and top performers only
- "Overall" closing paragraph wraps up with what to watch

## Reference: Sample Monthly Output

Opening: "Here's the campaign performance summary for [Month] through [Month] [Year]."
Body: 2-3 sentences max covering total sends, reply rate, best-performing month, and trend observation.
