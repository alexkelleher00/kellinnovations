# Property Empire — batch of updates (working doc)

Status: **items 1-5 and 7-8 shipped and verified** (113 backend tests passing, live
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
