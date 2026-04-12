# Anti-Productivity Simulator — Design Spec
**Date:** 2026-04-12
**Status:** Approved
**Output:** `/home/mauricionunez/Desktop/anti-productivity-simulator/index.html`

---

## Overview

A single-file browser game disguised as a corporate SaaS productivity tool. The experience lasts 1–3 minutes, designed as a humorous mental break. Visual aesthetic: SNES-era Zelda — pixel fonts, limited color palette, 8-bit sound, RPG HUD.

No frameworks. No build step. No server required. Open `index.html` in any modern browser.

---

## Architecture

### File Structure
```
anti-productivity-simulator/
└── index.html    (all HTML + CSS + JS in one file)
```

### Game State Machine
```
SCREEN_CORPORATE → SCREEN_LOADING → SCREEN_GAME → (loop)
                                         ↕
                                  SCREEN_ACHIEVEMENT (overlay)
```

### Core State Object
```js
GameState = {
  screen: "corporate" | "loading" | "game",
  focusPoints: 100,         // decrements passively (-1 per 3s) and on interaction complete
  timeWasted: 0,            // seconds elapsed since game start
  achievements: Set,        // unlocked achievement IDs
  currentInteraction: null, // ID of active mini-game
  interactionHistory: []    // prevents same interaction twice in a row
}
```

### Main Loop
1. `focusPoints` drains passively at -1 per 3 seconds
2. Interaction completes → `-5 to -15` focus points deducted
3. Ironic message displayed via typewriter effect (bottom bar)
4. 2-second pause → random new interaction loads (no immediate repeat)
5. Achievements trigger on conditions, shown as Zelda chest overlay

---

## Visual Design

### Color Palette (SNES Zelda)
| Token | Hex | Use |
|-------|-----|-----|
| `--green-dark` | `#1a3a1a` | Game background |
| `--gold` | `#c8a200` | HUD accents, borders |
| `--beige` | `#f0e68c` | Primary text |
| `--red-dark` | `#8b0000` | Danger states, low focus |
| `--black` | `#0a0a0a` | Loading screen, shadows |
| `--corporate-gray` | `#f5f5f5` | Corporate landing bg |
| `--corporate-blue` | `#2563eb` | Corporate button |

### Typography
- **Game UI:** `Press Start 2P` (Google Fonts CDN) — all pixel text
- **Corporate screen:** `Inter` or system sans-serif — clean, boring

### Visual Effects (CSS only)
- **Scanlines:** `::after` pseudo-element with repeating-linear-gradient, 2px lines, 30% opacity over full screen
- **Pixel borders:** `box-shadow` with multiple offsets creating blocky border (no border-radius anywhere in game UI)
- **Glitch effect:** CSS `@keyframes` with `clip-path` slices and `translateX` for the corporate→loading transition
- **CRT flicker:** Subtle `opacity` animation on the game container (0.97–1.0, 4s cycle)

### Sound (Web Audio API — no external files)
- **Click/confirm:** Short 880Hz square wave, 80ms
- **Achievement:** Ascending arpeggio C4-E4-G4-C5, square wave
- **Focus drain:** Low 220Hz thud, 150ms
- **Interaction complete:** Descending two-tone, 200ms
- **Meeting encounter:** Dissonant chord, 300ms

---

## Screens

### 1. Corporate Landing (`SCREEN_CORPORATE`)
- White background, no pixel fonts
- Logo: `ProFlow™` in gray sans-serif
- Headline: `"Optimize your productivity"`
- Subtext: `"Start your focused work session"`
- CTA button: `"Start Working"` (blue, rounded, corporate)
- On click: 500ms CSS glitch animation → cut to loading screen

### 2. RPG Loading Screen (`SCREEN_LOADING`)
- Black background, `Press Start 2P` font
- Sequence (timed):
  1. `LOADING SAVE FILE...` (800ms)
  2. Progress bar fills unevenly over 2000ms (random speed chunks)
  3. Bar reaches 100% → resets → `FILE CORRUPT...` (600ms)
  4. `LOADING ANYWAY` (400ms)
  5. Fade to game screen

### 3. Game Screen (`SCREEN_GAME`)

**HUD (top bar):**
```
[ANTI-PRODUCTIVITY SIMULATOR]  [🧠🧠🧠🧠🧠 FOCUS]  [⏱ 00:00 WASTED]
```
- 5 brain icons representing 20 focus points each
- Brains decay visually: full 🧠 → grey 🩶 → skull 💀
- Timer counts up in MM:SS format

**Interaction Area (center):**
- Zelda-style dialog box with pixel border (gold on dark green)
- Title of interaction at top
- Interactive content in center
- Interaction-specific UI rendered here

**Message Bar (bottom):**
- Typewriter effect, letter by letter
- Pool of ironic messages displayed after each interaction

---

## The 8 Interactions

### INT-01: CLICK QUEST
**Title:** `UNLOCK MOTIVATION`
**Mechanic:** Counter displays "0 / 37 CLICKS". Each click increments. At 37: brief celebration pixel animation.
**Focus penalty:** -8
**End message:** `MOTIVATION NOT FOUND`
**Achievement trigger:** `CLICKED_WITHOUT_PURPOSE`

### INT-02: ELUSIVE BUTTON
**Title:** `SUBMIT REPORT`
**Mechanic:** One button labeled "SUBMIT". On `mouseover`, it teleports to a random position within the container (minimum 100px away). After 15 seconds OR 10 escape attempts, button freezes and becomes clickable.
**Focus penalty:** -10
**End message:** `REPORT SUCCESSFULLY AVOIDED`

### INT-03: MEETING ESCAPE
**Title:** `⚠ ENEMY ENCOUNTERED`
**Mechanic:** A pixel-art enemy sprite labeled `"MEETING (COULD HAVE BEEN EMAIL)"` slowly tracks the mouse cursor. User must keep cursor away from it for 10 seconds. If touched, HP bar loses a tick and chase speed increases. After 10 seconds survived: WIN.
**Focus penalty:** -12
**End message:** `YOU SURVIVED: MEETING THAT COULD HAVE BEEN AN EMAIL`
**Achievement trigger:** `MEETING_SURVIVOR`

### INT-04: RUPIA RAIN
**Title:** `COLLECT RUPIAS`
**Mechanic:** Rupias fall from top (CSS animation). Green=bad (-2 focus on catch), Gold=good. Click to catch. Need 10 gold rupias. Green ones reset progress if caught accidentally. Timer: 20 seconds.
**Focus penalty:** -7
**End message:** `WALLET FULL, PRODUCTIVITY EMPTY`

### INT-05: ETERNAL STANDUP
**Title:** `DAILY STANDUP`
**Mechanic:** Countdown from 5:00. At 0:00 → `"RESCHEDULED TO TOMORROW"` → resets to 5:00. After 3 resets, a small "LEAVE MEETING" button appears in the corner. Click it to escape.
**Focus penalty:** -6
**End message:** `FREEDOM ACHIEVED (TEMPORARILY)`

### INT-06: PROGRESS BAR
**Title:** `UPLOADING TPS REPORT`
**Mechanic:** Progress bar fills to 99% then resets. Resets 3 times. On 4th attempt: bar only reaches 100% if mouse is completely still for 3 seconds (mousemove resets progress). Tooltip: `"DO NOT MOVE"`.
**Focus penalty:** -9
**End message:** `SCHRÖDINGER'S UPLOAD: COMPLETE`

### INT-07: INBOX ZERO QUEST
**Title:** `INBOX: ∞`
**Mechanic:** Email icons fall. User clicks to "delete" them. Each deleted email spawns 3 more. After deleting 15 total emails, game declares `"INBOX ZERO"` sarcastically.
**Focus penalty:** -11
**End message:** `INBOX: ∞`
**Achievement trigger:** `INBOX_HERO` (after 20 deleted)

### INT-08: TASK LABYRINTH
**Title:** `NAVIGATE REQUIREMENTS`
**Mechanic:** 5×5 pixel grid maze. Player moves with WASD or arrow keys. Exit is reachable. On reaching exit: `"WRONG EXIT — SEE ATTACHED: requirements_FINAL_v3.docx"`. Maze resets with new layout. Escape: find the *same* exit twice in a row.
**Focus penalty:** -13
**End message:** `REQUIREMENTS UPDATED. START OVER.`

---

## Gamification

### Focus Points Display
```
100–81: 🧠🧠🧠🧠🧠  (all healthy, green tint)
80–61:  🧠🧠🧠🧠💀  (one skull, no tint change)
60–41:  🧠🧠🧠💀💀  (two skulls, slight red tint on HUD)
40–21:  🧠🧠💀💀💀  (three skulls, red tint)
20–1:   🧠💀💀💀💀  (HUD flickers)
0:      💀💀💀💀💀  (game over screen — "GAME OVER: You did actual work")
```

### Achievements (Zelda chest overlay)
| ID | Title | Trigger |
|----|-------|---------|
| `CLICKED_WITHOUT_PURPOSE` | `★ CLICKED WITHOUT PURPOSE` | Complete INT-01 |
| `MEETING_SURVIVOR` | `★ MEETING SURVIVOR` | Complete INT-03 |
| `PROFESSIONAL_PROCRASTINATOR` | `★ PROFESSIONAL PROCRASTINATOR` | Complete 3 interactions |
| `THE_FLOOR_IS_DEADLINES` | `★ THE FLOOR IS DEADLINES` | Focus drops below 50 |
| `INBOX_HERO` | `★ INBOX HERO` | Delete 20 emails in INT-07 |
| `SPEEDRUNNER` | `★ SPEEDRUNNER` | Complete 5 interactions in <3 minutes |

**Achievement display:** Zelda treasure chest CSS animation (chest opens, star rises), sound arpeggio, text `"YOU GOT: [ACHIEVEMENT NAME]"`, auto-dismiss after 3 seconds.

---

## Ironic Message Pool (bottom typewriter bar)
```
PRODUCTIVITY DECREASED SUCCESSFULLY
GREAT JOB AVOIDING WORK
YOU ARE NOW 3% LESS EFFICIENT
BOSS IMPRESSION: MAXIMUM
THIS SESSION HAS BEEN LOGGED
HR HAS BEEN NOTIFIED (JUST KIDDING)
CONGRATULATIONS ON YOUR PROGRESS (THERE IS NONE)
YOUR CONTRIBUTION HAS BEEN NOTED AND IGNORED
SYNERGY LEVELS: CRITICAL
PLEASE UPDATE YOUR STATUS IN JIRA
```

---

## JS Module Structure

```
// === STATE ===
const GameState = { ... }

// === SCREENS ===
function showCorporateScreen() { ... }
function showLoadingScreen() { ... }
function showGameScreen() { ... }

// === HUD ===
function updateHUD() { ... }
function updateFocusDisplay() { ... }
function startTimerWasted() { ... }

// === INTERACTION ENGINE ===
function loadRandomInteraction() { ... }
function completeInteraction(focusPenalty, message) { ... }

// === INTERACTIONS (one function each) ===
function interactionClickQuest() { ... }
function interactionElusiveButton() { ... }
function interactionMeetingEscape() { ... }
function interactionRupiaRain() { ... }
function interactionEternalStandup() { ... }
function interactionProgressBar() { ... }
function interactionInboxZero() { ... }
function interactionTaskLabyrinth() { ... }

// === ACHIEVEMENTS ===
function checkAchievements() { ... }
function showAchievement(id) { ... }

// === SOUND ===
function playSound(type) { ... }  // uses Web Audio API

// === MESSAGES ===
function typewriterMessage(text) { ... }
```

---

## Adding New Interactions

To add a new interaction in the future:
1. Add function `interactionMyNew()` following the pattern of existing ones
2. Call `completeInteraction(penalty, message)` when done
3. Add entry to `INTERACTIONS` array: `{ id: 'my_new', fn: interactionMyNew }`
4. Optionally add an achievement trigger inside the function

---

## Out of Scope (MVP)
- Mobile touch support (desktop priority)
- Saving progress between sessions
- Multiplayer / leaderboard
- User-customizable interactions
- External asset files (sprites, audio files)
