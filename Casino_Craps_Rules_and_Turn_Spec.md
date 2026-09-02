# Casino Craps — Detailed Rules & Turn/Roll Specification

## 1. Overview

Craps is a dice game played with two standard six-sided dice. Players bet on the outcome of rolls made by a designated shooter.

A craps **round** is divided into:

1. A **come-out phase**
2. A **point phase**, if the come-out roll establishes a point
3. A sequence of additional rolls
4. A **seven-out**, point resolution, or other event that ends the shooter's hand
5. A new come-out roll when appropriate

The most important concept for implementing craps is that the table has a persistent state called the **Point**.

---

# 2. The Dice

Every roll consists of two six-sided dice.

Possible totals:

| Total | Possible combinations |
|---:|---:|
| 2 | 1+1 |
| 3 | 1+2 |
| 4 | 1+3, 2+2, 3+1 |
| 5 | 1+4, 2+3, 3+2, 4+1 |
| 6 | 1+5, 2+4, 3+3, 4+2, 5+1 |
| 7 | 6 combinations |
| 8 | 5 combinations |
| 9 | 4 combinations |
| 10 | 3 combinations |
| 11 | 2 combinations |
| 12 | 1 combination |

The total 7 is the most common result.

---

# 3. The Shooter

The player currently rolling the dice is the **shooter**.

The shooter must normally make a **Pass Line** bet before rolling.

The shooter continues rolling until the hand ends with a **seven-out**.

A seven-out occurs when:

- A point has been established, and
- The shooter rolls a 7 before making the point.

After a seven-out, the dice are passed to another shooter.

Important distinction:

> The shooter's hand can contain many rolls. A "roll" is one throw of the two dice; a "hand" or "turn" is the entire sequence controlled by one shooter.

---

# 4. Table State

A craps game can be modeled with these primary states:

```text
COME_OUT
POINT_ESTABLISHED
SEVEN_OUT
NEW_SHOOTER
```

The game begins in:

```text
COME_OUT
```

There is no point.

Example:

```text
point = None
phase = COME_OUT
```

---

# 5. Come-Out Roll

The first roll made by a shooter is called the **come-out roll**.

During the come-out roll, the Pass Line has the following rules.

## Natural

If the shooter rolls:

- 7
- 11

The Pass Line wins immediately.

The shooter continues and another come-out roll occurs.

Example:

```text
Roll 1: 7
Pass Line: WIN

Roll 2: 11
Pass Line: WIN

Roll 3: 5
Point becomes 5
```

The 7 or 11 does **not** end the shooter's hand.

---

## Craps

If the shooter rolls:

- 2
- 3
- 12

The Pass Line loses.

The point is still `None`.

The shooter rolls another come-out roll.

Example:

```text
Roll 1: 2
Pass Line: LOSE

Point: None

Roll 2: 8
Point = 8
```

A 2, 3, or 12 on the come-out roll does not end the shooter's hand.

---

## Point Established

If the come-out roll is:

- 4
- 5
- 6
- 8
- 9
- 10

That number becomes the **Point**.

Example:

```text
Come-out roll = 6

point = 6
phase = POINT_ESTABLISHED
```

The shooter continues rolling.

---

# 6. Point Phase

Once a point has been established, the rules change.

Suppose:

```text
point = 6
```

The shooter continues rolling until either:

1. The shooter rolls another 6 → point is made
2. The shooter rolls a 7 → seven-out
3. Something else → continue rolling

Example:

```text
Come-out: 6
Point = 6

Roll: 4
Nothing happens

Roll: 10
Nothing happens

Roll: 8
Nothing happens

Roll: 6
POINT MADE
```

After the point is made, the point resets:

```text
point = None
phase = COME_OUT
```

The same shooter gets another come-out roll.

---

# 7. Seven-Out

A seven-out can only occur after a point has been established.

Example:

```text
Come-out: 8
Point = 8

Roll: 5
Continue

Roll: 6
Continue

Roll: 7
SEVEN-OUT
```

The shooter's hand ends.

The current shooter stops rolling.

The dice are passed to a new shooter.

The new shooter begins with a come-out roll.

---

# 8. Complete Turn Example

Here is a complete shooter hand.

```text
Shooter A begins

Come-out:
Roll 1 = 4
Point = 4

Point phase:
Roll 2 = 8
Nothing happens

Roll 3 = 5
Nothing happens

Roll 4 = 4
Point made

Point resets.

Come-out:
Roll 5 = 11
Pass Line wins
No point

Come-out:
Roll 6 = 6
Point = 6

Point phase:
Roll 7 = 3
Nothing happens

Roll 8 = 9
Nothing happens

Roll 9 = 7
SEVEN-OUT

Shooter A's hand ends.

Dice → Shooter B
```

Notice that a single shooter can have many come-out rolls during one hand.

---

# 9. Pass Line Bet

The Pass Line is the primary craps bet.

A player places the Pass Line bet before the come-out roll.

## Come-out results

| Roll | Pass Line |
|---:|---|
| 2 | Lose |
| 3 | Lose |
| 4 | Point established |
| 5 | Point established |
| 6 | Point established |
| 7 | Win |
| 8 | Point established |
| 9 | Point established |
| 10 | Point established |
| 11 | Win |
| 12 | Lose |

Once a point is established:

| Roll | Pass Line |
|---:|---|
| Point | Win |
| 7 | Lose |
| Anything else | Continue |

A Pass Line bet generally cannot be increased after the point is established.

---

# 10. Don't Pass

The Don't Pass is essentially the opposite side of the Pass Line, with an important exception for 12.

On the come-out roll:

| Roll | Don't Pass |
|---:|---|
| 2 | Win |
| 3 | Win |
| 4 | Point established |
| 5 | Point established |
| 6 | Point established |
| 7 | Lose |
| 8 | Point established |
| 9 | Point established |
| 10 | Point established |
| 11 | Push |
| 12 | Push |

After a point is established:

- 7 → Don't Pass wins
- Point → Don't Pass loses
- Other numbers → continue

The 12 is traditionally a push/tie rather than a win or loss.

---

# 11. Odds Bets

After a point is established, a Pass Line player may usually place **odds** behind the Pass Line bet.

Odds pay true mathematical odds.

Typical payouts:

| Point | Odds payout |
|---:|---:|
| 4 | 2:1 |
| 5 | 3:2 |
| 6 | 6:5 |
| 8 | 6:5 |
| 9 | 3:2 |
| 10 | 2:1 |

Example:

```text
Pass Line = $10
Point = 6

Player places $10 odds.

If 6 is rolled:
Pass Line wins $10
Odds win $12

Total profit = $22
```

The exact maximum odds allowed depends on the casino/table.

Common limits include:

```text
1x odds
2x odds
3x odds
5x odds
10x odds
20x odds
```

Some tables advertise special odds such as 3-4-5x.

---

# 12. Come Bet

A Come bet works similarly to a Pass Line bet, but it begins **after a point already exists**.

Example:

```text
Main point = 6

Player makes $10 Come bet.

Next roll = 8

Come bet moves to 8.
```

The Come bet now has its own number.

The table can therefore have:

```text
Main Point = 6
Come Bet Point = 8
```

The next rolls can affect both.

Example:

```text
Roll = 8

Main point 6 → nothing
Come point 8 → WIN
```

A Come bet can also have odds placed behind it after its number is established.

---

# 13. Don't Come

A Don't Come bet is the opposite-side equivalent of a Come bet.

After placing it:

- 2 or 3 → wins
- 12 → push
- 7 or 11 → loses
- 4, 5, 6, 8, 9, 10 → establishes a Don't Come number

Once established:

- 7 → wins
- Established number → loses
- Other numbers → continues

---

# 14. Place Bets

Place bets bet that a specific number will roll before a 7.

Common place numbers:

```text
4
5
6
8
9
10
```

A player can place several numbers simultaneously.

Example:

```text
Place 6
Place 8
Place 9
```

The bets remain active while the shooter rolls.

If the selected number rolls:

```text
WIN
```

If 7 rolls:

```text
LOSE
```

The player can normally remove or change a Place bet between rolls, subject to table rules and dealer procedure.

---

# 15. Typical Place Bet Payouts

Standard payouts are:

| Number | Typical payout |
|---:|---:|
| 4 | 9:5 |
| 5 | 7:5 |
| 6 | 7:6 |
| 8 | 7:6 |
| 9 | 7:5 |
| 10 | 9:5 |

The minimum wager may be adjusted by the dealer so the bet can pay the correct amount.

For example, a $10 place 5 generally pays:

```text
$10 × 7/5 = $14 profit
```

---

# 16. Field Bet

The Field is a one-roll bet.

It wins on:

```text
2
3
4
9
10
11
12
```

It loses on:

```text
5
6
7
8
```

Typical payout:

```text
2 → 2:1
12 → 2:1
3,4,9,10,11 → 1:1
```

Some casinos use a higher payout for either 2 or 12.

Therefore a game implementation should store the table's specific Field payout rules.

---

# 17. Big 6 and Big 8

Some traditional tables offer:

```text
Big 6
Big 8
```

These bets win if the selected number appears before 7.

For example:

```text
Big 6
```

wins if:

```text
6 occurs before 7
```

They generally pay even money.

Many modern craps tables do not offer these bets.

---

# 18. Proposition Bets

Proposition bets are generally one-roll or short-event bets placed in the center of the table.

Common examples:

```text
Any Craps
Craps 2
Craps 3
Craps 12
Any 7
Hard 4
Hard 6
Hard 8
Hard 10
Yo (11)
```

These usually have significantly higher house edges than Pass Line, Don't Pass, or odds.

---

# 19. Hardways

A Hardway means rolling a number as a pair of matching dice.

Examples:

```text
Hard 4 = 2 + 2
Hard 6 = 3 + 3
Hard 8 = 4 + 4
Hard 10 = 5 + 5
```

A hardway bet generally wins when the hard number occurs before:

- 7
- The corresponding easy version of the number

Example:

```text
Hard 8

4 + 4 → WIN

6 + 2 → EASY 8 → LOSE
5 + 3 → EASY 8 → LOSE
7 → LOSE
```

---

# 20. Hardway Example

Suppose:

```text
Hard 6 = $5
```

Rolls:

```text
5
8
4
10
3
7
```

The Hard 6 loses when the 7 appears.

If instead the rolls were:

```text
5
8
3
3
```

The final `3 + 3 = 6` is a Hard 6, so the bet wins.

---

# 21. One-Roll Bets vs Persistent Bets

For implementation, it is useful to divide bets into two categories.

## One-roll bets

These resolve immediately after the next roll.

Examples:

```text
Field
Any 7
Any Craps
Craps 2
Craps 3
Craps 12
Yo
```

They should automatically disappear after the roll.

---

## Persistent bets

These remain active until their condition resolves or the player removes them.

Examples:

```text
Pass Line
Don't Pass
Come
Don't Come
Place bets
Hardways
```

---

# 22. Roll Processing Order

A software implementation should process every dice roll in a predictable order.

Recommended sequence:

```text
1. Generate dice
2. Calculate total
3. Record dice values
4. Determine current game phase
5. Resolve one-roll bets
6. Resolve Pass/Don't Pass
7. Resolve Come/Don't Come
8. Resolve Place bets
9. Resolve Hardways
10. Resolve other persistent bets
11. Update point
12. Determine whether shooter continues
13. Record payouts
14. Prepare next roll
```

The exact internal ordering can vary as long as all bets receive the correct result based on the state **before** and **after** the roll.

---

# 23. Recommended Game State

A basic craps engine could maintain:

```python
game = {
    "phase": "COME_OUT",
    "point": None,
    "shooter": 1,
    "roll_number": 0,
    "hand_number": 1,
    "dice": [0, 0],
    "total": None
}
```

Example after a point is established:

```python
game = {
    "phase": "POINT",
    "point": 6,
    "shooter": 1,
    "roll_number": 4,
    "hand_number": 1,
    "dice": [3, 3],
    "total": 6
}
```

---

# 24. Basic Game Algorithm

Pseudocode:

```text
START GAME

point = NONE
shooter = first player

LOOP:

    shooter makes COME-OUT roll

    IF total == 7 OR total == 11:
        resolve come-out wins
        point remains NONE
        continue

    IF total == 2 OR total == 3 OR total == 12:
        resolve come-out losses/pushes
        point remains NONE
        continue

    IF total is 4, 5, 6, 8, 9, or 10:
        point = total
        resolve bets
        enter POINT phase


    WHILE point exists:

        shooter rolls

        resolve all bets

        IF total == point:
            point is made
            resolve Pass Line
            point = NONE
            return to COME_OUT

        ELSE IF total == 7:
            seven-out
            resolve Pass Line
            end shooter hand
            shooter = next shooter
            point = NONE
            return to COME_OUT

        ELSE:
            continue rolling
```

---

# 25. Example With Multiple Bets

Suppose:

```text
Pass Line = $10
Place 6 = $12
Place 8 = $12
```

Come-out roll:

```text
8
```

Pass Line establishes:

```text
Point = 8
```

The Place 6 and Place 8 remain active.

Next roll:

```text
6
```

Results:

```text
Pass Line → nothing
Place 6 → wins
Place 8 → nothing
```

Next roll:

```text
8
```

Results:

```text
Pass Line → wins
Place 8 → wins
```

The point resets.

The same shooter gets a new come-out roll.

---

# 26. Multiple Simultaneous Numbers

One of the reasons craps is complicated is that several bets can be tracking different numbers simultaneously.

Example:

```text
Main Point = 6

Come bet → 8

Place 5
Place 9

Hard 6
```

A roll of 8 could therefore:

```text
Main Point 6 → nothing
Come 8 → wins
Place 5 → nothing
Place 9 → nothing
Hard 6 → remains active
```

A roll of 6 could:

```text
Main Point 6 → point made
Come 8 → nothing
Place 5 → nothing
Place 9 → nothing
Hard 6 → depends on dice:
              3+3 = Hard 6 → WIN
              4+2 = Easy 6 → LOSE
```

This is why the dice values, not just the total, must be stored.

---

# 27. Dice Values Matter

Never store only:

```python
total = 6
```

You should store:

```python
die1 = 3
die2 = 3
total = 6
```

because:

```text
3 + 3 = Hard 6
4 + 2 = Easy 6
5 + 1 = Easy 6
```

All have the same total but different betting consequences.

---

# 28. Shooter Hand vs Individual Roll

This distinction is extremely important.

### Roll

One throw of the dice.

Example:

```text
Roll #17
Dice = 4 + 2
Total = 6
```

### Shooter hand

Everything from when the shooter receives the dice until seven-out.

Example:

```text
Hand #12

Roll 1 → 7
Roll 2 → 4
Roll 3 → 9
Roll 4 → 6
Roll 5 → 4
Roll 6 → 8
Roll 7 → 7

Hand ends.
```

The first 7 above is **not** a seven-out because there was no point.

The final 7 **is** a seven-out because point 4 was active.

---

# 29. New Shooter

After seven-out:

```text
old shooter → stops rolling

point = NONE

next eligible shooter → receives dice
```

The next shooter begins with a new come-out roll.

Example:

```text
Shooter A
Point = 5
Roll = 7

SEVEN-OUT

Shooter B receives dice

Come-out roll
```

Players' bets are not necessarily all removed merely because the shooter changes. Persistent bets have their own rules.

---

# 30. Don't Automatically Clear the Entire Table

A common programming mistake is:

```python
if seven_out:
    bets.clear()
```

That is incorrect.

Instead, each bet should have its own state and lifecycle.

For example:

```text
Pass Line → resolves
Place bets → generally lose
Come bets → generally lose on 7
Hardways → lose
Player's chips → remain
New shooter → begins
```

Some bets can be taken down or turned off according to the specific bet and table rules.

---

# 31. Bet Object Model

For a game implementation, a bet can be represented as:

```python
bet = {
    "player_id": 3,
    "type": "PLACE",
    "amount": 12,
    "number": 6,
    "active": True
}
```

A Pass Line bet could be:

```python
bet = {
    "player_id": 3,
    "type": "PASS_LINE",
    "amount": 10,
    "number": None,
    "active": True
}
```

A Come bet after moving to 8:

```python
bet = {
    "player_id": 3,
    "type": "COME",
    "amount": 10,
    "number": 8,
    "active": True
}
```

---

# 32. Important Event Types

A digital craps game should ideally record events.

Examples:

```text
ROLL_STARTED
DICE_ROLLED
COME_OUT_NATURAL
COME_OUT_CRAPS
POINT_ESTABLISHED
POINT_HIT
SEVEN_OUT
BET_PLACED
BET_WON
BET_LOST
BET_PUSHED
BET_MOVED
SHOOTER_CHANGED
HAND_STARTED
HAND_ENDED
```

Example event:

```json
{
    "event": "POINT_ESTABLISHED",
    "roll": 17,
    "total": 8,
    "point": 8,
    "shooter": 2
}
```

This makes a game much easier to debug and replay.

---

# 33. Roll-by-Roll Example

Complete simplified example:

```text
HAND 1 — SHOOTER A

Roll 1
Dice: 3 + 4
Total: 7
State: COME_OUT

Pass Line → WIN
Point → NONE

--------------------------------

Roll 2
Dice: 2 + 3
Total: 5
State: COME_OUT

Point → 5

--------------------------------

Roll 3
Dice: 4 + 4
Total: 8
State: POINT

Point = 5
No point hit
Continue

--------------------------------

Roll 4
Dice: 2 + 4
Total: 6

Continue

--------------------------------

Roll 5
Dice: 3 + 2
Total: 5

POINT MADE

Pass Line → WIN
Point → NONE

--------------------------------

Roll 6
Dice: 5 + 6
Total: 11

COME-OUT NATURAL

Pass Line → WIN

--------------------------------

Roll 7
Dice: 4 + 4
Total: 8

Point → 8

--------------------------------

Roll 8
Dice: 3 + 5
Total: 8

POINT MADE

Point → NONE

--------------------------------

Roll 9
Dice: 6 + 1
Total: 7

COME-OUT NATURAL

Pass Line → WIN

--------------------------------

Roll 10
Dice: 2 + 5
Total: 7

COME-OUT NATURAL

Pass Line → WIN

--------------------------------

Roll 11
Dice: 3 + 2
Total: 5

Point → 5

--------------------------------

Roll 12
Dice: 6 + 1
Total: 7

SEVEN-OUT

HAND ENDS

Dice → Shooter B
```

---

# 34. Simplified State Machine

The entire game can be visualized as:

```text
              ┌─────────────────────┐
              │      COME-OUT       │
              └──────────┬──────────┘
                         │
             ┌───────────┼────────────┐
             │           │            │
           7/11        2/3/12       4/5/6/8/9/10
             │           │            │
             ▼           ▼            ▼
          Pass Win     Pass Lose   SET POINT
             │           │            │
             └───────┬───┘            ▼
                     │          ┌───────────────┐
                     └─────────►│ POINT PHASE   │
                                └───────┬───────┘
                                        │
                              ┌─────────┼─────────┐
                              │                   │
                         Point rolled          7 rolled
                              │                   │
                              ▼                   ▼
                         POINT MADE          SEVEN-OUT
                              │                   │
                              ▼                   ▼
                         COME-OUT          NEW SHOOTER
```

---

# 35. Key Rules for a Digital Implementation

If building a craps game, these rules should be treated as fundamental:

### Rule 1
A 7 on the come-out roll is **not** a seven-out.

### Rule 2
An 11 on the come-out roll is a Pass Line win.

### Rule 3
A 2, 3, or 12 on the come-out roll is Craps.

### Rule 4
A 4, 5, 6, 8, 9, or 10 establishes the point.

### Rule 5
Once a point exists, that number must be rolled before a 7 for the Pass Line to win.

### Rule 6
A 7 before the point is made is a seven-out.

### Rule 7
After a point is made, the point resets and the same shooter begins another come-out cycle.

### Rule 8
After a seven-out, the shooter changes.

### Rule 9
Dice values must be preserved, not just the total.

### Rule 10
Every bet needs its own lifecycle.

---

# 36. Recommended Digital Turn Structure

For a game with multiple human or AI players:

```text
GAME
│
├── Table
│   ├── Current shooter
│   ├── Current point
│   ├── Current phase
│   └── Roll history
│
├── Players
│   ├── Chips
│   ├── Active bets
│   └── Betting permissions
│
└── Roll
    ├── Die 1
    ├── Die 2
    ├── Total
    └── Result events
```

A single roll should then follow:

```text
WAIT FOR BETS
       ↓
LOCK BETTING
       ↓
ROLL DICE
       ↓
CALCULATE TOTAL
       ↓
PROCESS BETS
       ↓
UPDATE POINT
       ↓
CHECK SEVEN-OUT / POINT MADE
       ↓
PAY WINNERS
       ↓
DISPLAY RESULT
       ↓
OPEN NEXT BETTING WINDOW
```

---

# 37. Casino-Style Timing

A digital game can emulate a real casino by dividing the experience into phases:

```text
BETTING OPEN
      ↓
BETTING CLOSED
      ↓
DICE ANIMATION
      ↓
RESULT
      ↓
PAYOUTS
      ↓
POINT/SHOOTER UPDATE
      ↓
BETTING OPEN
```

During:

```text
DICE ANIMATION
```

players should not be allowed to change bets.

After the result:

```text
PAYOUTS
```

are processed.

Then the game transitions to the next roll.

---

# 38. Important Variations

Real casinos can have different rules concerning:

- Maximum odds
- Place bet minimums
- Proposition payouts
- Field payouts
- Whether Big 6/8 are offered
- Buy/Lay bet commissions
- Don't Pass odds
- Don't Come odds
- Fire bets
- Bonus bets
- Crapless craps
- Other specialty bets

Therefore, a digital game should ideally store table configuration separately from core game logic.

Example:

```python
table_rules = {
    "max_odds": 3,
    "field_2_payout": 2,
    "field_12_payout": 2,
    "big_6_8_available": False
}
```

This allows different casino tables without rewriting the game engine.

---

# 39. Core Craps Engine in One Paragraph

The simplest correct mental model is:

> A shooter starts with a come-out roll. A 7 or 11 wins the Pass Line, while 2, 3, or 12 loses it. A 4, 5, 6, 8, 9, or 10 establishes the point. Once a point exists, the shooter keeps rolling until either the point is rolled, which wins the Pass Line and starts a new come-out cycle, or a 7 is rolled, which is a seven-out and ends the shooter's hand. Other totals simply allow the shooter to continue. The dice then pass to the next shooter.

That state machine is the foundation on which the rest of the craps table is built.
