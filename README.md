# Instantly Campaign Report Skill

A [Claude Agent Skill](https://docs.claude.com/en/docs/agents-and-tools/agent-skills/overview) that turns pasted [Instantly](https://instantly.ai/) cold email campaign data into polished, narrative performance reports — weekly or monthly — complete with a formatted Word doc and a Slack send-off message, without manual write-up.

Paste a campaign data table into a conversation with Claude, ask for a report, and it generates: totals, top performers, notable flags, and a wrap-up, following a consistent format every time. Ask for a `.docx` and it builds one with the same table/color styling every time. Ask for the Slack message that goes with it and it drops in the same three-line template with the right date range auto-filled.

## What it does

- **Weekly reports** — campaign-by-campaign narrative covering top performers by replies, positive responses, and volume, with related campaigns grouped together, plus an overall summary.
- **Monthly reports** — a short, high-level roll-up across months (totals, reply rate, best month, volume trend).
- **Consistent formatting** — bolded campaign names and metrics, calculated reply/positive response rates, and rules for which campaigns get their own paragraph vs. get grouped or skipped.
- **Markdown by default** — output is plain text/markdown; a `.docx` file is only generated if you explicitly ask for one afterward.
- **Branded Word doc format** — navy header row, green TOTAL row, left-aligned campaign column, 10pt table text, and green highlight shading on whichever campaign leads the period, all generated automatically.
- **Slack send-off message** — a short "Hi, Happy [Day]! Sending over the ... report ..." message you can paste straight into Slack, with the date range and report type pulled from whatever report you just generated.

## Installation

Pick whichever fits how you use Claude:

### Option 1 — Claude Code plugin marketplace (recommended for Claude Code users)

```
/plugin marketplace add jeffreythekiln/instantly-report-skill
/plugin install instantly-campaign-report@instantly-report-skill
```

Restart Claude Code when prompted. Update later with `/plugin marketplace update`.

### Option 2 — Agent Skills CLI (`npx`, works across Claude Code, Cursor, and other agents)

```
npx skills add jeffreythekiln/instantly-report-skill
```

This is a community installer ([skills.sh](https://skills.sh)), not an official Anthropic tool, but it reads `SKILL.md` straight from the repo and works across several agents. If it can't find the skill at the repo root, point it at the nested path instead:

```
npx skills add jeffreythekiln/instantly-report-skill/skills/instantly-campaign-report
```

### Option 3 — Manual upload to claude.ai or the Claude API

1. Download [`instantly-campaign-report.skill`](instantly-campaign-report.skill) from this repo (it's a ZIP with the skill folder as its root).
2. In claude.ai: **Settings > Capabilities > Skills** (or **Customize > Skills** on some plans) > upload the file.
3. Via the API: use the Skills API to upload the same ZIP to your workspace.

See Anthropic's [Agent Skills documentation](https://docs.claude.com/en/docs/agents-and-tools/agent-skills/overview) for details on managing skills for your specific setup — this can change, so check there if something above doesn't match what you see.

## Usage

Just paste your campaign data table into the chat and ask for a report. For example:

**Weekly:**
```
Here's this week's campaign data (June 29 – July 5):

Campaign            Status   Sent  Replies  Positive  OOO
...

Can you write up a report?
```

**Monthly:**
```
Monthly totals for April through June 2026:

Month   Email Sent   Replies
...

Give me a monthly summary.
```

If the data doesn't make it obvious whether you want a weekly or monthly report, Claude will ask.

Once you're happy with the text output:
- Ask Claude to **"make this a doc"** or **"save this as a Word file"** and it generates a matching `.docx` with the navy/green table styling and highlight shading.
- Ask Claude for **"the Slack message for this"** and it generates the send-off message template with the same date range and report type as the report — as chat text, never a file.

## Expected input format

| Report type | Required columns | Date context |
|---|---|---|
| Weekly | Campaign, Status, Sent, Replies, Positive, OOO | A date range (e.g. "June 29 – July 5") |
| Monthly | Month, Email Sent, Replies | A month range and year (e.g. "April through June 2026") |

## Example output

See [`skills/instantly-campaign-report/references/sample-weekly.md`](skills/instantly-campaign-report/references/sample-weekly.md) and [`skills/instantly-campaign-report/references/sample-monthly.md`](skills/instantly-campaign-report/references/sample-monthly.md) for full worked examples of the expected tone and structure.

## Repo structure

```
instantly-report-skill/
├── .claude-plugin/
│   ├── plugin.json                 # Claude Code plugin manifest
│   └── marketplace.json            # Marketplace catalog (this repo is its own marketplace)
├── skills/
│   └── instantly-campaign-report/
│       ├── SKILL.md                 # Skill definition and instructions (report + docx + Slack message)
│       └── references/
│           ├── sample-weekly.md    # Example weekly report output
│           └── sample-monthly.md   # Example monthly report output
├── instantly-campaign-report.skill # Pre-zipped skill folder for manual upload (Option 3)
└── README.md
```

## Customizing

All report logic (parsing rules, grouping rules, paragraph structure, docx table/color format, Slack message template) lives in `skills/instantly-campaign-report/SKILL.md`. Edit it directly to change tone, thresholds (e.g. what counts as "notable"), report structure, or the docx color scheme.

If you're using the Claude Code plugin install (Option 1), edits to `SKILL.md` in this repo are picked up next time you run `/plugin marketplace update`. If you're on the manual-upload path (Option 3), re-zip and re-upload after editing.

## License

Add a license of your choice (e.g. MIT) if you plan to share this publicly.
