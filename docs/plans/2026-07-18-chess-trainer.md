# Chess Improvement Program + Personal Trainer App (500 → 1000+)

**Date:** 2026-07-18
**Status:** Plan (no code yet)
**Goal:** Take a ~500-rated chess.com player to 1000+ rapid, and design a small app that automates the training loop.

---

## Part 0 — Brainstorm summary

### The core insight that shapes everything below

At 500, games are not decided by strategy, openings, or endgame technique. They are decided by
**free pieces**: one player leaves a piece where it can be taken for nothing (or misses that the
opponent did). Analysis of low-rated games consistently shows 3–6 outright blunders per game per
side. At 1000, that number is roughly 1 per game.

Therefore the entire 500→1000 journey compresses to two trainable skills:

1. **Stop giving pieces away** (blunder-check discipline before every move).
2. **Take pieces that are given away** (board vision — spotting undefended/hanging pieces).

Plus a thin layer of: basic tactics (forks, pins, mate-in-1/2), sane opening habits, and two
checkmating techniques. Everything else (opening theory, positional play, deep endgames) is
*wasted effort at this level* and is deliberately excluded.

### Assumptions made (adjust the plan if wrong)

| Assumption | Value chosen | Why |
|---|---|---|
| Time budget | ~45–60 min/day | Enough for the loop below; less than this stretches the timeline, it doesn't break the method |
| Time control | Rapid 15\|10 | Blitz/bullet at 500 trains guessing, not thinking. Rating goal is rapid |
| Timeline | 10–16 weeks | Realistic for +500 with daily practice; faster is possible but not promised |
| App platform | Web (client-only, free hosting) | Zero backend cost, works on phone + desktop, Stockfish runs in-browser via WASM |
| Data source | chess.com public API | It's where the games and rating live; API is free, no auth |

### Success criteria

- **Primary:** chess.com rapid rating ≥ 1000, sustained over 20+ games.
- **Leading indicators (the app measures these):**
  - Blunders per game (moves losing ≥ 2 pawns of eval): from ~4 → below 1.
  - Hanging-piece capture rate: % of opponent's free pieces actually taken → above 80%.
  - Puzzle accuracy at 400–1000 puzzle rating → above 80% *without time pressure*.

---

## Part 1 — The training program (works with or without the app)

### The daily loop (~50 min)

1. **Puzzles — 15 min.** Easy tactical puzzles (rated 400–1000). Accuracy over speed: the goal is
   ≥80% correct. Themes in priority order: hanging piece, mate in 1, mate in 2, fork, pin.
   Getting one wrong = replay it until the pattern is obvious.
2. **Play — 2 games of 15|10 rapid, ~30 min.** Non-negotiable rules while playing:
   - **Before every move, run the 3-second check:** "If I make this move, what can my opponent
     *take*, and what can they *check*?" Physically look at every opponent piece that attacks
     the square you're moving to.
   - **After every opponent move, ask:** "What changed? What does that move attack or threaten?"
   - Use the clock. Finishing a 15|10 game with 12 minutes left means you weren't checking.
   - Hard tilt rule: after 2 losses in a row, stop playing for the day (puzzles are still fine).
3. **Review — 5 min per loss.** Not a deep analysis. One question only: *"Find the first move
   where a piece was lost for free (mine or theirs)."* Look at the position, cover the moves,
   and find what should have been played. That single habit is the highest-ROI 5 minutes in chess.

### The weekly additions

- **One 30-minute game** (or 30|0) per week — longer thinking time cements the blunder-check habit.
- **Endgame drill, 10 min/week:** checkmate with K+Q vs K, then K+R vs K, against the computer
  until it's automatic (this alone converts many "winning but drew/lost" games).

### Opening policy (deliberately minimal)

No memorized lines. One setup per color, chosen for clarity of plans:

- **White:** Italian Game (1.e4 e5 2.Nf3 Nc6 3.Bc4). Develop knights before bishops, castle by
  move 6–8, don't move the same piece twice, don't bring the queen out early.
- **Black vs 1.e4:** 1...e5 and mirror the same development principles. Learn to *refute* the
  Scholar's Mate attempt (Qh5/Qf3 + Bc4 aimed at f7) — at 500 this is worth ~50 rating by itself.
- **Black vs 1.d4:** 1...d5, develop, castle.

### Phase gates (how you know it's working)

| Gate | Signal | Expected rating |
|---|---|---|
| Gate 1 (weeks 1–3) | Blunder rate under 3/game; you spot Scholar's Mate every time | 600–650 |
| Gate 2 (weeks 4–7) | Blunder rate under 2/game; you take >60% of hanging pieces offered | 700–800 |
| Gate 3 (weeks 8–12) | Blunder rate ~1/game; puzzle accuracy ≥80%; K+Q and K+R mates automatic | 900–1000 |
| Gate 4 (weeks 12–16) | Sustained 1000+ over 20 games | 1000+ ✅ |

---

## Part 2 — The app: **BlunderGym** (a personal trainer built from your own games)

### Why build an app at all when chess.com/Lichess exist?

Generic trainers show you *random* puzzles. The fastest improvement at this level comes from
drilling **your own mistakes**: positions where *you* actually blundered last week, resurfaced
as puzzles on a spaced-repetition schedule until you stop making that class of mistake. No
mainstream tool does this well for free — and it's a perfect one-person app: no backend, no
accounts, no cost.

### Core features (mapped to the skills that move rating)

1. **Game importer** — pulls all your games from the chess.com public API by username.
2. **Blunder Finder** — Stockfish (WASM, in-browser) walks every game and flags each move where
   your evaluation dropped ≥ 200 centipawns (or a win became a draw/loss). Each blunder becomes
   a card: the position, the move you played, and "find the better move."
3. **Mistake Repetition Queue** — your blunder cards on a spaced-repetition schedule
   (Leitner boxes: again → 1d → 3d → 7d → 21d). Solve it right, it moves back; wrong, it resets.
4. **Hanging-Piece Radar (board vision drill)** — shows a random position (from your games or
   the puzzle DB), you have 20 seconds to tap every undefended/underdefended piece. Trains the
   scan that prevents *and* exploits free pieces.
5. **Tactics packs** — Lichess open puzzle database (5M puzzles, CC0-licensed CSV, each tagged
   with theme + rating) filtered to 400–1200 and the five priority themes.
6. **Endgame gym** — K+Q vs K and K+R vs K versus Stockfish, with a move counter (par: mate in
   under 15 moves from any position).
7. **Progress dashboard** — rating over time (chess.com API), blunders/game trend line, puzzle
   accuracy, hanging-piece capture rate. The Gate 1–4 table above, rendered live.

### Tech stack (all free, all client-side)

| Concern | Choice | Notes |
|---|---|---|
| Framework | Vite + React + TypeScript | Simple SPA; no server |
| Board UI | `react-chessboard` | Drag/drop, arrows, mobile-friendly |
| Rules/PGN/FEN | `chess.js` | Legal moves, PGN parsing of imported games |
| Engine | `stockfish` npm (WASM) in a Web Worker | UCI protocol; depth 12–15 is plenty for blunder detection |
| Puzzles | Lichess `lichess_db_puzzle.csv` | Pre-filter offline to ~20k easy puzzles, ship as a static JSON chunk |
| Games/rating | chess.com Published-Data API | `GET api.chess.com/pub/player/{user}/games/{YYYY}/{MM}` — no auth, CORS-enabled |
| Storage | IndexedDB via `dexie` | Blunder cards, repetition schedule, stats — all local, private |
| Hosting | Vercel or GitHub Pages | Static deploy, $0 |

### Architecture notes

- **Everything runs in the browser.** The chess.com API is public and CORS-friendly; Stockfish
  WASM analyzes ~1 game in a few seconds at depth 12 in a worker without blocking the UI.
  There is deliberately no login, no database, no server to maintain.
- **Analysis is incremental and cached:** each imported game is analyzed once, results stored in
  IndexedDB keyed by game URL. Re-opening the app only analyzes new games.
- **Blunder threshold is tunable** but defaults to: eval swing ≥ 200cp, or win-probability drop
  ≥ 20% (win-prob smooths out silly ±800cp swings in already-lost positions).

---

## Part 3 — Implementation plan (phased, each phase independently shippable)

### Phase 1 — MVP: "Show me my blunders" (build first, ~1–2 weekends)

1. Scaffold Vite + React + TS app; board page rendering a FEN with `react-chessboard` + `chess.js`.
   - ✅ *Verify: can display any position and make legal moves.*
2. Importer: input a chess.com username → fetch monthly archives → parse PGNs (last 50 games).
   - ✅ *Verify: your real account loads; game list shows result, color, opponent, date.*
3. Stockfish worker: UCI wrapper (`position fen … / go depth 12`), eval per position.
   - ✅ *Verify: startpos evals ≈ +0.2; a queen-odds position evals ≈ ±9.*
4. Blunder detection pass over one game → list of blunder cards (FEN, played move, best move).
   - ✅ *Verify: hand-check 3 games — every "piece hung for free" move is flagged.*
5. Review UI: show blunder position, ask for the better move, reveal engine line on failure.
   - ✅ *Verify: full loop works on your own games. This alone is already a useful app — ship it.*

### Phase 2 — Retention: spaced repetition + tactics packs

6. Leitner scheduler in Dexie (5 boxes: 0d/1d/3d/7d/21d); daily queue = due cards, oldest first.
7. Filter Lichess puzzle CSV offline (rating 400–1200; themes: hangingPiece, mateIn1, mateIn2,
   fork, pin) → static JSON; add "Tactics" tab consuming the same card UI.
8. Daily session composer: 10 blunder-review cards + 10 tactics = the app's "today" screen.
   - ✅ *Verify: card answered wrong today reappears tomorrow; right → next box interval.*

### Phase 3 — Board vision + endgames

9. Hanging-Piece Radar: static analysis (attackers vs defenders per square via chess.js) marks
   ground truth; tap-to-answer with 20s timer and streak scoring.
10. Endgame gym: K+Q/K+R vs K from random legal starts, play vs Stockfish (limited strength),
    move-count par scoring.
    - ✅ *Verify: stalemate is detected and scored as failure (the classic K+Q trap).*

### Phase 4 — Dashboard + coach

11. Stats: rating pulled from chess.com stats endpoint; blunders/game and capture-rate trends
    computed from analyzed games; Gate 1–4 table rendered with live status.
12. "Coach card" on the today screen: one sentence generated from the data, e.g. *"Your last 10
    games: 2.1 blunders/game (down from 3.4). Most common: leaving knights undefended after
    ...Nf6–g4. 6 review cards due."*
13. Deploy to Vercel; add PWA manifest so it installs on the phone.

### Explicitly out of scope (resist the temptation)

- Opening repertoire trainer, accounts/auth/backend, multiplayer, mobile-native builds,
  engine strength above what blunder-detection needs. None of these move a 500 to 1000.

---

## Part 4 — Risks & mitigations

| Risk | Mitigation |
|---|---|
| Building the app becomes procrastination from playing | The daily loop (Part 1) starts **today**, app or no app. Phase 1 is capped at 2 weekends |
| Stockfish WASM performance on phone | Depth 12 + single thread is fine; analysis runs on desktop, review works anywhere (cards are precomputed) |
| Lichess puzzle file is huge (~900MB raw) | One-time offline filter script → ~5MB JSON shipped with the app |
| Rating plateau ~800 despite fewer blunders | Expected: add the endgame gym + mate-in-2 packs (Phase 3) — converting won positions is usually the 800→1000 gap |
| chess.com API rate limits | Serial requests + IndexedDB cache; archives are fetched once per month of history |
