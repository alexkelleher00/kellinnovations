# Property Empire — batch of updates (working doc)

Status: **items 1-5 and 7-10 shipped and verified** (131 backend tests passing, live
browser smoke test of every feature). Item 6 is still a follow-up, not done — see bottom.

## 1. Bankruptcy payout change (3+ player games) — done
- [x] Cash still goes to whoever the debt was owed to (unchanged).
- [x] Properties transferred to a player-creditor now come over **mortgaged** (and any
      houses/hotels on them return to level 0) instead of transferring fully improved —
      no more inheriting a built-up monopoly for free.
- [x] Bankrupt-to-the-bank path unchanged: properties return unowned so anyone can
      buy/auction them again.
- [x] Bankrupt player still becomes a spectator of the rest of the game (confirmed,
      no regression).
- [x] Backend tests added.

## 2. Bigger board squares — done
- [x] Reweighted the 11x11 grid's column/row tracks (1.7fr outer / 0.9fr inner)
      so the outer ring (corners + edge properties) gets bigger at the expense of
      the purely-decorative center area.
- [x] Fixed `applyBoardZoom()`'s follow-mode math (`trackCenterFraction()`), which
      assumed uniform 1/11 track sizes — verified the "zoom to my token" feature
      still centers correctly with the new weighted tracks.
- [x] Nudged corner/cell font-size clamp ceilings up to use the extra room.
- [x] Verified visually — corners ~3x bigger area, edge cells ~40% bigger area.

## 3. Chat — done
- [x] Backend: text-only chat action, validated/200-char-capped, stored in
      `self.chat`, included in state.
- [x] Frontend: chat button + input panel near the reaction button, text only.
- [x] Sent messages flash on screen like reactions.
- [x] Full history viewable in the drawer, in its own "Chat" section next to Activity.

## 4. More reaction emojis — done
- [x] Added: 🤙 (finger guns), 🤏 (pinching hand), 🍆 🍑 🍒 🍌 to the existing set
      (now 14 total).

## 5. Roulette wonky rule — done (v1, later superseded by item 8 below)
- [x] New lobby toggle (host-only, like Easy Monopolies): "🎰 Roulette".
- [x] Not turn-gated — bet red/black (even money) or a number 0-36 (35:1), any
      amount up to current cash ("ALL IN" shortcut = all of it).
- [x] Standard European-wheel numbers/colors (0 green, real red/black split).
- [x] Spin result + win/loss broadcasts to everyone via the existing log/toast.
- [x] 🎰 button opens a betting modal — bet type tabs, amount + presets, spin
      button with a brief animation, colored result reveal.
- [x] Backend tests: payouts (1:1 and 35:1), can't bet more than cash, can't go
      negative, works regardless of whose turn it is.

## 6. (Stretch / optional — "feel free to") Standalone casino game + homepage card
Not started. Explicitly framed as optional in the original ask ("feel free to"),
and items 1-5 above were already a full session's worth of work. Flagged to the
user as a genuine follow-up rather than rushed in at the end.

## 7. Rent/debt payout capping — done
Fixes the "creditor gets overpaid" bug: previously `_transfer` always credited the
payee the *full* amount owed even when the payer couldn't cover it (letting cash go
negative), and if the payer then went bankrupt the creditor got the full amount
*plus* whatever assets remained — a double benefit. Concrete example fixed: a
player with $1,000 cash landing on a $2,000 rent now pays out at most $1,000 up
front, never $2,000.
- [x] `_transfer` caps the immediate payment at `min(amount, cash_on_hand)`. Any
      shortfall becomes a tracked debt: `self.debts[pid] = {"creditor": id|"bank",
      "remaining": amount}` (previously just a bare creditor-id string).
- [x] `_maybe_clear_debt` now actually routes recovered cash to the creditor
      incrementally (mortgaging, selling a building, an incoming trade, or someone
      else paying the debtor rent) instead of just clearing a flag once cash
      happened to reach 0 — the creditor gets paid as the debtor recovers, capped
      at what was actually owed, never more.
- [x] `declare_bankruptcy` payout to the creditor is capped at the remaining
      tracked debt (defensive — normally 0 more is owed by the time bankruptcy is
      reached, since recovered cash is already swept as it comes in).
- [x] Bot debt resolution (`_bot_resolve_debt`), `respond_trade` (sweeps debt for
      both sides after a cash exchange), and all existing debt/bankruptcy tests
      updated for the new dict-shaped debt entries.
- [x] Frontend debt banner now shows the specific remaining amount owed and who
      it's owed to, instead of a generic "you owe money" message.
- [x] New backend tests: exact $1k/$2k scenario from the ask, incremental
      creditor payout via mortgaging, bankruptcy payout capping (including a
      defensive case with stale/mismatched state). 113 tests passing total.

## 8. Real roulette wheel + full bet menu — done (supersedes item 5's simple version)
- [x] Backend: full real-table bet menu, any combination staked in a single spin —
      straight (35:1), split (17:1), street (11:1), corner (8:1), six-line (5:1),
      column (2:1), dozen (2:1), plus outside bets red/black/odd/even/low/high
      (1:1). Adjacency validated server-side (split/street/corner/six-line must be
      real neighbors on the table, matching physical roulette layout rules). 0 is
      bettable straight-up only (basket/trio bets touching 0 are out of scope —
      they vary by American vs. European table convention).
- [x] Betting popup: click-to-select number grid (0 + 1-36, matches real table
      layout) infers straight/split/street/corner/six-line from selection count/
      shape; separate buttons for the outside bets, 3 columns, and 3 dozens. Bets
      are staged into a list (removable) before spinning — any mix at once.
- [x] Every spin is broadcast to all clients via `self.roulette_spins` (same
      flash-detection pattern as chat/reactions), so every player at the table
      watches the same wheel animate together, not just whoever pressed spin.
- [x] Real spinning wheel: large centered overlay (up to ~80vw/68vh), a static
      conic-gradient disc built from the actual European wheel's physical pocket
      order (not table order), with a ball that spins several full rotations and
      decelerates to land on the winning number, followed by a result caption
      (number, color, net win/loss) shown to everyone.
- [x] Backend tests for every bet type's validation + payout, invalid
      adjacency/targets rejected, multi-bet-combo-in-one-spin net calculation,
      spin broadcast to `roulette_spins`. 
- [x] Live 2-browser Playwright smoke test: staged a 5-bet combo (outside +
      column + dozen + straight + split), spun, confirmed the wheel opens and
      animates identically for both the spinner and an observer via polling, and
      closes cleanly after one animation cycle (caught and fixed a real
      double-queue bug during this test — the spinner's own render() call was
      independently re-detecting its own spin as a "new" broadcast).

## 9. Roulette-only debt rule: mortgage-then-gamble, pay off manually — done
New request: when the roulette rule is on, raising cash (mortgaging, selling a
building, a trade, being paid rent) should NOT auto-pay down a tracked debt —
the player can deliberately gamble that cash on the wheel first, and only settle
up via a manual "Pay Debt" button, for exactly as long as they choose to risk it.
Normal (non-roulette) games keep the auto-sweep-as-you-mortgage behavior from
item 7 completely unchanged.
- [x] `_maybe_clear_debt` (the auto-sweep hook called from mortgage/sell/trade/
      incoming-rent) is now a no-op whenever `self.roulette_enabled` is true.
      Extracted the actual settle logic into `_settle_debt`, shared with the new
      manual action.
- [x] New `pay_debt` action/method: sweeps whatever cash the player currently has
      toward their tracked debt on demand. Works in both modes, but it's the
      *only* way to pay down debt when roulette is on. Rejects if nothing's owed
      or there's no cash to pay with.
- [x] Bots never gamble — `_bot_resolve_debt` explicitly calls `_settle_debt`
      after every mortgage/sale regardless of the roulette flag, so bot behavior
      is identical in both modes.
- [x] Frontend debt banner: shows a "Pay Debt" button (only when there's cash on
      hand to pay with) plus mode-specific copy explaining whether cash auto-pays
      or has to be paid manually.
- [x] Fixed a pre-existing visual bug flagged during this pass: the reaction
      picker (now 14 emoji, 4 rows) was anchored low enough that its top rows sat
      behind the chat/roulette FAB buttons, which paint on top since they're
      later in the DOM — silently hiding 🍑 (and partly 🍒/🍌) behind the
      roulette button whenever the picker was open. Repositioned the picker
      above the whole FAB stack (same anchor the chat panel already uses).
- [x] Backend tests: auto-sweep disabled when roulette is on (mortgage, incoming
      rent), manual pay_debt pays correctly and rejects invalid cases, normal
      games unaffected, bots still self-settle under roulette. 121 tests passing.
- [x] Live browser test: forced a debt scenario via a debug-only endpoint (dev
      server only, not shipped), confirmed the Pay Debt button appears/disappears
      based on cash on hand, actually pays down the debt when clicked, and
      confirmed via bounding-box math that the peach emoji no longer overlaps
      the roulette/chat buttons.

## 10. CPU bot fixes + rejoin-a-game — done
New request: "the CPU seems to freeze for ~20s then jump ahead, I missed an
auction", "it bids on auctions it can't afford", "make the CPU better in
general", plus a standalone ask for a way to rejoin a game after losing your
session (refresh, cleared storage, new device).

**Found and fixed the actual root cause of the freeze/missed-auction report** —
a real, serious bug, not a misunderstanding:
- When the bot's *own* turn triggers an auction (it declines to buy), the
  auction's first bidder is the human — not the bot — per `_rotated_order_from`
  (whoever passed bids last). But `turn_order[current_idx]` still points at the
  bot until the auction resolves. `_bot_step()` didn't have an explicit "an
  auction is active and it's not my turn in it" check, so execution fell through
  to the bot's normal turn logic (build → end_turn) *while the auction was still
  waiting on the human*. `end_turn()` raises `GameError("An auction is in
  progress")` in that case — uncaught, which crashed `tick()`, which runs at the
  top of **every single poll and action from every connected browser**. Every
  request 500'd until the human's 20-second auction timer expired and
  auto-passed them, at which point the auction resolved and ticks stopped
  crashing — explaining exactly what was reported: total freeze, then a sudden
  jump once the (unseen) auction had already resolved.
- Fixed: `_bot_step()` now explicitly returns (does nothing) whenever there's an
  active auction that isn't the bot's turn to act in, instead of falling through.
- Verified the bug was real by reverting just this fix and re-running the new
  regression test — it reproduced the exact `GameError` and traceback described
  above. Reinstated the fix, test now passes.
- Found and fixed a second, unrelated live bug while smoke-testing this: the
  "Add CPU Player" button in the lobby also 500'd (`add_bot()` returns a live
  `Player` object, which isn't JSON-serializable) — the bot still got added
  server-side, but the click always errored and only looked like it worked once
  the next poll silently picked up the change. Fixed by converting to a plain
  dict at the HTTP boundary, without changing what the engine method itself
  hands back to in-process callers (test suite still uses the live object).
- Auction affordability: the existing cash-reserve check (`BOT_CASH_RESERVE`)
  was actually already correct on inspection — a cash-poor bot already passes
  even on a $1 bid. Added explicit tests locking this in (a poor bot passes, a
  healthy one bids) since the crash bug above was very likely what actually made
  bidding look erratic/unaffordable in practice.
- General bot improvement: bots now proactively propose a trade (fair cash
  premium, capped by what they can spare) whenever they're exactly one property
  short of a monopoly and the last piece is a plain, unbuilt, unmortgaged
  property held by a single human opponent — previously bots only ever
  responded to trades offered *to* them, never initiated one, so a bot holding
  the last piece of someone else's color group would sit on it forever.
- Roulette's full real bet menu (item 8) was already complete from a prior
  pass — verified intact, no regressions. Added payout odds (e.g. "17:1") next
  to every bet button/hint in the UI so "all the correct odds" is visibly
  confirmable, not just functionally correct under the hood.
- New `POST /rooms/<code>/rejoin` endpoint: given a room code + the exact name
  you were already using (case-insensitive, names are already unique per room),
  issues a fresh session token for that player. Works in any game phase, and
  bots can't be rejoined into (nobody "was" the CPU). New "Rejoin as that
  Player" button on the landing screen's Join panel.
- Backend tests: the exact auction-freeze regression (reproduced against the
  unfixed code first, to confirm it's real), the auction still resolves on
  timeout, add_bot serialization isn't retested at the unit level (no existing
  HTTP-level test suite — see note below), bot trade-proposal behavior (proposes
  correctly, skips duplicates/built/mortgaged/unaffordable/unowned cases), and
  auction affordability. 131 tests passing (up from 121).
- Live browser verification: 100+ cumulative seconds of a bot playing a full
  game with zero server errors (previously would reliably 500 within the first
  auction); full rejoin flow including a cleared-session reload and a
  wrong-name rejection.
- **Note for later**: there's still no HTTP/route-level test suite for this
  project — only the `Game` engine class itself is unit-tested. The add_bot bug
  above is exactly the kind of thing that only shows up at the JSON-serialization
  boundary, which engine-level tests can't catch. Worth a dedicated follow-up to
  add a small Flask-test-client suite (with the room-persistence directory
  properly redirected to a temp dir for test isolation) rather than folding it
  into this fix.
