# OB + Volume Strength

A TradingView (Pine Script v6) indicator that marks **order blocks** from structure breaks and measures **how much volume actually went into them** — so you can tell a block worth respecting from a block that's just a box on your chart.

Built and tuned for **XAUUSD intraday** (1m / 3m / 5m), but the logic works on any symbol that reports volume.

---

## Table of contents

- [What it does](#what-it-does)
- [Quick start](#quick-start)
- [Reading the OB label](#reading-the-ob-label)
- [The baseline — the key to every number](#the-baseline--the-key-to-every-number)
- [Strength ladder](#strength-ladder)
- [The panel, row by row](#the-panel-row-by-row)
- [Retest volume (R) explained](#retest-volume-r-explained)
- [Sessions matter](#sessions-matter)
- [MTF bias table](#mtf-bias-table)
- [What makes a block fail](#what-makes-a-block-fail)
- [Settings reference](#settings-reference)
- [Known limitations](#known-limitations)
- [Journalling](#journalling)

---

## What it does

1. **Finds structure breaks** (BOS / CHoCH) using a pivot swing lookback
2. **Walks back** from the break to find the last opposite-coloured candle — that's the order block
3. **Draws the zone** and measures the volume that went into forming it
4. **Scores that volume** against the recent average, so you know if it's strong or noise
5. **Tracks retest volume separately** every time price comes back into the zone
6. **Adjusts expectations by session**, because London volume and Asian volume are not the same thing

The volume scoring is the part most order block indicators don't do. Marking boxes is easy — knowing which ones matter is the actual problem.

---

## Quick start

1. Add the script to your chart
2. Confirm your symbol reports volume (on gold, use **OANDA:XAUUSD** — some feeds return nothing)
3. Set **Session timezone** to match yours (default `GMT+8` / PHT)
4. Leave everything else alone for the first week. Just watch.

> **Don't tune on day one.** The default thresholds are a starting point, not a truth. Observe first, adjust after you have data.

---

## Reading the OB label

Each block carries a label like:

```
13.609k  (8%)   R:0.42K
```

| Part | Meaning |
|---|---|
| `13.609k` | **Formation volume** — total volume from the order block candle **through the candle that broke structure** |
| `(8%)` | That volume as a share of the last 40 candles' total volume |
| `R:0.42K` | **Retest volume** — accumulates each time price re-enters the zone |

> ⚠️ **Common misread:** formation volume is *not* just the one OB candle. If the block took 3 bars to form before structure broke, all 3 are in that number.
>
> A **1-bar block at 8%** is much stronger than a **3-bar block at 8%** — the second one is spreading the same volume across three candles. Glance at how wide the block is before trusting the multiple.

**Grey boxes** are mitigated — price already broke through. Kept on chart for reference, no longer tradeable.

---

## The baseline — the key to every number

The percentage is a **share of the lookback window**, not a share of anything intuitive. So the neutral point isn't 50%:

```
1 average candle  =  100 ÷ volLB  =  100 ÷ 40  =  2.5%
```

**2.5% is 1x. That's average. That's nothing.**

Everything else is a multiple of that:

```
8% ÷ 2.5% = 3.2x
```

> This block holds about **3.2 average candles' worth** of volume.

This is why an "8%" block is actually significant even though 8 sounds small. It's one zone eating 8% of forty candles' worth of activity.

**If you change `volLB`, the baseline changes with it.** At 60 bars, 1x becomes 1.67%. The panel recalculates automatically — but your mental shortcuts won't, so be aware.

---

## Strength ladder

| % (at volLB=40) | Multiple | Read |
|---|---|---|
| 2.5% | 1x | Noise — average candle, ignore |
| 5% | 2x | Minimum worth marking |
| 7.5% | 3x | Solid — real participation |
| 10% | 4x | Strong |
| 15% | 6x | Very strong |
| 25%+ | 10x+ | Extreme — displacement or news |

**1x is not "stronger than average." 1x *is* average.** Strength starts above it.

---

## The panel, row by row

### SESSION
Shows the active session and what it requires right now.

```
SESSION    LONDON    need 7.5% (3.0x)
```

### LAST OB
The most recently created block — its percentage, its multiple, its label, and direction (▲ bullish / ▼ bearish).

```
LAST OB ▲   8.0%   3.2x  SOLID
```

### VERDICT
The panel doing the session math for you.

- **`PASSES session filter`** (green) — strong enough for the session we're in
- **`BELOW session filter`** (red) — the block exists, but isn't carrying enough volume to respect right now

> **The same block can pass in Asian and fail in London.** A 2.5x block is fine at 10am PHT and unremarkable at 4pm PHT. This row saves you doing that comparison in your head while price is moving.

> ⚠️ **PASSES is not an entry signal.** It only means the block cleared the volume bar. Direction, structure, and HTF bias are still your call.

### WARNING

```
⚠ Window warming 23/40 — % inflated
✓ Window settled
```

London just opened, and the 40-candle lookback is still mostly filled with quiet Asian candles. Every percentage reads inflated because it's being measured against a sleepy baseline.

The counter shows progress — at `23/40`, there are 17 bars to go. During warming, treat high percentages with suspicion. A block showing 15% at London open might be completely ordinary once the window catches up.

### LEGEND
The strength ladder, with the band your current OB falls into **highlighted**.

### RULE
Static footer reminding you of both session thresholds:

```
Asian 2.0x  |  Ldn/NY 3.0x
```

Pulls from your settings, so it updates if you change them.

---

## Retest volume (R) explained

### First, the catch

**Formation volume and retest volume are not measured the same way.**

- **Formation** = fixed window (OB candle → break candle, usually 1–4 bars)
- **Retest** = *running total* that keeps adding every bar price sits inside the zone

So R grows the longer price hangs around. A big R can mean two completely opposite things:

- A violent high-volume rejection in 2 bars ✅
- Price loitering in the zone for 30 quiet bars ❌

**The number alone can't tell them apart. You have to look at how many bars price spent inside.** That's the reading skill — not the ratio.

### Volume > R  (formation bigger than retest)

Price came back into the zone **quietly**. The original move had more force than the return.

> Whoever loaded the block isn't being seriously challenged. Nobody's fighting at this level.

Generally the healthier setup — a clean, low-volume retest into a strong block is the textbook continuation scenario. Price is coming back to collect orders, not to battle.

*But check the bar count.* Volume > R because price touched once and left = good. Volume > R because price has barely reached the zone = you're reading an incomplete number.

### Volume < R  (retest bigger than formation)

More volume traded on the return than during formation. Three very different readings:

| Pattern | Read |
|---|---|
| **Fast (2–5 bars) + price rejected** | Strong defence. Heavy participation and the zone held — arguably the best confirmation available. |
| **Slow (15+ bars) + price still sitting** | Warning. The zone is being worn down. Blocks that get camped on usually break. |
| **R climbing + price grinding through** | Absorption against you. The block is failing. Get out of the way. |

### Cheat sheet

```
R stays small                      → intact, untested, still has orders
R spikes fast + price rejects      → defended. strongest confirmation.
R grows slowly + price lingers     → being eaten. weakening.
R grows + price pushes through     → failed. don't fight it.
```

**The question isn't "is R bigger or smaller."**
**It's "did R arrive fast or slow, and did price leave or stay?"**

### What R can't tell you

R is **directionless**. It counts total volume in the zone but can't tell you whether buyers or sellers produced it.

So: **R tells you how much fighting happened. Price action tells you who won.** You need both.

---

## Sessions matter

The lookback window is *rolling*, so the baseline itself shifts through the day.

| Session | Minimum | Strong | Note |
|---|---|---|---|
| **Asian** | 2x (5%) | 4x+ | Quiet baseline — 2x here is respectable |
| **London / NY** | 3x (7.5%) | 5x+ | Baseline already elevated, noise is louder, demand more |

**The trap:** the first candles after London open print huge percentages simply because the preceding 40 bars were Asian. Those aren't automatically good blocks — the window hasn't caught up. Give it ~40 bars post-open before trusting the number. The **WARNING** row tracks this for you.

---

## MTF bias table

Top-right. EMA-based bias across 15m / 1H / 4H / 1D, plus a **NET** verdict.

Use it as a filter: if NET says **BULLISH**, favour the bullish blocks. A high-volume block pointing against higher-timeframe direction is a **target**, not support.

This is the check the volume number can't give you.

---

## What makes a block fail

Volume alone won't tell you. Watch for:

- ❌ **High % from a news spike** — that's panic, not positioning. Reverts hard.
- ❌ **High volume on a doji or long-wick candle** — two sides fighting, nobody loading
- ❌ **Already retested 2+ times** — each touch drains it
- ❌ **Against the HTF bias** — no matter how big the number

That last one is the main killer. A 10x block against your higher-timeframe bias is a magnet, not a floor.

### What tends to hold

- ✅ High multiple **and** the candle did real work — displacement, not a wick-heavy indecision bar
- ✅ Formed *into* a structure break, not randomly mid-range
- ✅ Aligned with HTF bias
- ✅ Narrow formation (1–2 bars) carrying the volume

---

## Settings reference

### Structure
| Input | Default | Notes |
|---|---|---|
| Swing length (pivot) | 3 | Lower = more BOS detected, more noise. Higher = fewer, cleaner breaks. |
| Require displacement move | on | Filters weak breaks |
| Displacement = ATR × | 1.0 | Break candle body must exceed this × ATR |
| ATR length | 14 | |
| Show BOS / CHoCH labels | on | |

### Order Blocks
| Input | Default | Notes |
|---|---|---|
| Bars to search back for OB | 12 | How far back to hunt the opposite candle |
| Use full wick | on | Off = body only, tighter zones |
| Max live OBs per side | 6 | Chart clutter control |
| Extend box (bars) | 30 | |
| Show 50% equilibrium band | on | |
| Mitigation rule | Break of structure | `Wick touch` is stricter |
| Delete OB when mitigated | off | Off keeps greyed boxes for reference |

### Volume
| Input | Default | Notes |
|---|---|---|
| **Lookback for total volume** | **40** | **Sets the baseline. Changing this changes every percentage.** |
| Show % of total volume | on | |
| Show retest volume | on | |
| Hide OBs below % strength | 0 | Raise to declutter — 5 ≈ 2x filter |
| Label size | small | |

### Strength Panel
| Input | Default | Notes |
|---|---|---|
| Show volume strength panel | on | |
| Show strength legend | on | |
| Show session warnings | on | |
| Panel position | Bottom Right | Keep separate from the MTF table |
| **Session timezone** | **GMT+8** | **Set to your local zone** |
| Asian session | 0800-1600 | |
| London session | 1500-0000 | |
| New York session | 2030-0500 | |
| Asian: min multiple | 2.0 | |
| London/NY: min multiple | 3.0 | |

### MTF Bias Table
| Input | Default |
|---|---|
| Bias EMA length | 50 |
| TF 1–4 | 15 / 60 / 240 / D |

---

## Known limitations

**No buy/sell split.** Spot gold feeds report *tick volume* — the count of price updates, not actual contracts with a buyer/seller tag. You cannot split what was never recorded. Any indicator claiming to show "buying vs selling volume" on spot XAUUSD is inferring it from candle direction, which is not the same thing.

**Formation volume spans multiple bars.** The sum runs from the OB candle through the break candle, so multi-bar formations naturally score higher without being stronger *per candle*. The 1x = 2.5% baseline is a single-candle reference. Always note the bar count.

**Rolling window means session bleed.** Covered above — the WARNING row handles the London-open case, but the same effect exists at every session transition, just less dramatically.

**Tick volume ≠ real volume.** It correlates well enough for relative comparison (which is all this indicator does), but don't treat the raw numbers as institutional order flow.

---

## Journalling

Volume alone won't give you a win rate. Log these per trade and you'll have your own thresholds within ~30 trades — far more useful than any defaults:

| Field | Why |
|---|---|
| OB % at formation | The raw reading |
| Multiple (x) | The interpreted reading |
| **Bars in formation** | Distinguishes real strength from spread-out volume |
| Session | Context for the threshold |
| Candle type | Displacement / wick-heavy / doji |
| HTF aligned? | Y/N |
| R at entry | Retest pressure |
| **Bars price spent in zone** | Fast rejection vs slow grind |
| Outcome | W/L, R multiple |

After enough samples, the thresholds in this README should be replaced with **your** numbers.

---

## Credits

Order block logic follows the standard BOS/CHoCH → last opposite candle → zone recipe common to most OB indicators. The volume strength scoring, session-aware thresholds, and window-warming warning are the additions that make it usable as a filter rather than just a drawing tool.
