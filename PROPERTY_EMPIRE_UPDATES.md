# Property Empire — batch of updates (working doc)

Status: **items 1-5 shipped and verified** (95 backend tests passing, live browser
smoke test of every feature). Item 6 is a follow-up, not done — see bottom.

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

## 5. Roulette wonky rule — done
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
