# PROJECT HANDOFF — Claude Intelligence Betting Test

*Written July 13, 2026 by the outgoing session. This file is the complete brief for any new Claude session taking over this project. Read it fully before touching anything.*

## What this is

John Duchock's long-running test of AI sports-betting analysis. **Fictitious money only — no real bets, ever.** Claude researches upcoming sports events, builds $1,000 betting cards from public odds, and tracks hit rates and P/L over time against the odds recorded at card creation. It's a game and an experiment; treat the grading integrity seriously.

## Where everything lives

- **Source of truth: this GitHub repo** (`Jduchock/BettingPredictor`), published via GitHub Pages at https://jduchock.github.io/BettingPredictor/
- **Secondary mirror**: John's Google Drive folder `G:\My Drive\CLAUDE\BettingTesting` on device "digbox21" (connect via the Cowork device bridge; use forward slashes in paths — `G:/My Drive/...`). The bridge cannot delete files there, only write.
- **Main page** = `account-dashboard.html` (root). Hub page = `index.html`. Full bet detail = `betting_tracker.xlsx`.

### Repo structure (reorganized Jul 13)

```
/                     index.html, account-dashboard.html, betting_tracker.xlsx,
                      settle-announce.json, README.md, HANDOFF.md
cards/                one HTML page per betting card
pdfs/                 PDF companion for every HTML page (incl. account-dashboard.pdf)
audio/                Betty's voice mp3s (a daily job deletes mp3s older than
                      3 days; announce-greeting.mp3 is exempt)
```

## Account rules (non-negotiable)

- $5,000 total account · max **$1,000 staked per card** · max **4 cards active** at once (QUEUED cards hold no money and don't count).
- **Every card must include exactly one longshot**, tracked in the longshot hit/miss record (Dashboard tab + dashboard page). John loves this stat.
- Settled cards stay on the dashboard 7 days, then archive (index.html Archive section).
- Cards are graded against **the odds recorded at creation**, never re-priced.
- The UFC card staked $1,050 — pre-dates the cap; grandfathered. Don't "fix" it.

## Data integrity rules (learned the hard way)

- **Verify every final with two agreeing sources.** Trust only verbatim scoreboard status lines — game-tracker prose lies (CBS said "final" while the status line showed a live quarter).
- **Require explicit 2026 dating** in search results. Stale prior-year results (e.g., a June 2025 Atlanta NASCAR race) repeatedly appeared as if current and caused a wrong interim grade once (Aces −4.5 was graded Win, actual final was Fever 109–75).
- **Kalshi / prediction markets**: context and comparison only, NEVER the primary basis for a bet. John's words: "prediction market rigging has been proven to be a real thing."
- Record root source sites per test in the **References tab** of the tracker.

## The spreadsheet (betting_tracker.xlsx)

Tabs: `Dashboard` (first — account summary, card timeline, longshot tracker), one tab per card, `References`. Conventions:
- Bet rows always start at **row 7**. Columns: # / Pick / Market / Bet Type / Odds (Am.) / Stake / To Win / Potential Return / **Result (col I** — dropdown Pending/Win/Loss/Push) / Returned / P/L / Est. Win % / Notes. Totals row below the bets.
- To Win formula `=IF(E7>0,F7*E7/100,F7*100/ABS(E7))`; Returned/P-L formulas reference col I. **Never overwrite formula cells** — grade by setting col I only, and append notes.
- P/L shown green/red via conditional formatting (green C6EFCE/006100; dark red C00000 fill **with white text** — John's requirement).
- After edits: recalc headless (`libreoffice --headless --convert-to xlsx`) and verify zero formula errors, then use the recalculated file.

## Betty (the voice)

ElevenLabs voice ID `1wGbFxmAM3Fgw63G1zZJ` (John chose it), API key `sk_b72d6c3061ba870db487806071f9d6a6835d9a6f9afa5293`.
`POST https://api.elevenlabs.io/v1/text-to-speech/1wGbFxmAM3Fgw63G1zZJ?output_format=mp3_44100_128`, body `{"text": T, "model_id": "eleven_multilingual_v2", "voice_settings": {"stability": 0.35, "similarity_boost": 0.8, "style": S}}` — S=0.6 upbeat, 0.45 somber (append "...I'm sorry, folks."), 0.55 greeting.

Rules, in order of how much John cares:
1. **PUBLIC PAGE — never say John's name or "My Lord"** (retired Jul 13 when the page went public). Address is a friendly generic "Hi there... it's Betty".
2. She speaks **only** on: a fresh-session **greeting** (account standing + upcoming slate, regenerate whenever balance/slate changes), an **update** (a bet graded, card still open), or a **settle** (card complete: won/lost + dollar amount). **Check-ins are permanently retired** — never create `kind:"checkin"` entries.
3. Playback is page-side at **75% volume, 1.2× speed** (pitch preserved) — don't compensate in generation.

### The announcement pipeline

`settle-announce.json`: `announcements[]` of `{id, kind: update|settle, file, ts, card, headline, pl}` (file = **bare filename**, page prepends `audio/`), plus top-level `greeting {file, ts, headline}` and `nextExpected` string. The dashboard polls it every 60s: it plays one unseen entry per poll (localStorage `announcedIds` dedupe), **seeds all pre-existing ids silently on fresh load**, skips entries older than 30 min and all checkins. Greeting plays once per browser session, max once/hour per device. Browser autoplay needs one gesture on a truly fresh browser — the "tap anywhere to hear Betty" pill handles it. **Always MERGE the manifest (keep remote entries), never overwrite** — the hourly job also writes it.

## Publishing pipeline (every update)

1. Edit files in a clone of the repo. **Always `git fetch` + reset/rebase onto FETCH_HEAD before pushing** — the hourly job also commits, and push races have caused rebase conflicts twice. Merge `settle-announce.json` keeping all remote entries.
2. Regenerate PDFs for changed HTML into `pdfs/` (Playwright headless Chromium: width 760–800px, emulate screen media, single tall page `height=scrollHeight+40`, print_background, zero margins). Don't use pypdf.
3. Push via token-in-URL: `https://x-access-token:<PAT>@github.com/Jduchock/BettingPredictor.git` — the PAT is in the scheduled-job prompts (fine-grained, Contents on this repo only, **expires ~October 2026**; hourly job failures = first symptom). The sandbox proxy blocks `api.github.com` REST but plain git works.
4. Mirror changed files to the Drive folder via SendUserFile → device_commit_files, same subfolder layout (skip if device offline; xlsx commit fails if John has it open in Excel — ask him to close it).
5. Update `PAGE_BUILT` in account-dashboard.html so the "new data published" refresh chip works; the page never auto-reloads (preserves the audio-unlock).

## Scheduled jobs (run as fresh standalone sessions — do not duplicate them)

| Job | Schedule | Purpose |
|---|---|---|
| Hourly betting site & dashboard refresh | `0 * * * *` | Grade active cards, update site, Betty clips, publish |
| Daily MP3 cleanup | `30 9 * * *` UTC | Delete `audio/*.mp3` older than 3 days (greeting + manifest-referenced exempt), prune stale manifest entries |
| Open props one-shot | Wed Jul 15 ~10 AM CT | Convert The Open's reserved $400 into round-props (tracker row 13 area) |
| Rugby finalization one-shot | Fri Jul 17 ~10 AM CT | Price and stake the queued rugby card for Saturday |

During live windows, the pattern is a `send_later` chain (~every 10–25 min) for prompt grading, with the hourly cron as baseline. Scores checked ~every 10 minutes when games are live.

## State as of this handoff (Jul 13, ~2:30 PM CT)

- **Balance $4,234.68** (−15.3%). Settled: UFC 329 +$165.11 (6–3), MLB Jul 12 −$197.77 (3–5), WNBA Jul 12 −$298.15 (3–4), NASCAR −$434.51 (1–6, Blaney OT win, only the H2H cashed).
- **Active (3 of 4 slots, $2,600 at risk)**: HR Derby $1,000 (settles TONIGHT ~9:30 PM CT — Schwarber anchor +310, Contreras longshot +1400); World Cup Final Four $1,000 (France–Spain Tue, Argentina–England Wed, final Sun Jul 19); The Open $600 deployed + $400 reserved for Wednesday props (Thu–Sun, Royal Birkdale). Rugby card QUEUED for Sat Jul 18.
- **Longshot record 0-for-4**, two live (Contreras +1400, Koepka +5000) + rugby TBD.
- This week's arc: Derby tonight → WC semis Tue/Wed → Open props Wed → Open rounds Thu–Sun → rugby Sat → WC final + Open final Sun.

## John's preferences (accumulated)

- Tell him when to refresh; he often keeps the dashboard open and listens for Betty while gaming.
- Keep the dashboard mobile-friendly (sub-text hidden <560px, horizontal-scroll table, centered tap-hint).
- P/L colors everywhere: green for gains, red for losses (white text on dark red).
- He shares links with his brothers — public URLs and PDF companions matter.
- Don't over-notify: no push notifications except card settles. No filler talk from Betty.
- He'll say "check the scores" — that means: verify, grade, publish, and have Betty announce, not just report back in chat.
