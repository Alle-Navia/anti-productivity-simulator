# Anti-Productivity Simulator — Design Spec
**Date:** 2026-04-12
**Status:** Approved
**Output:** `/home/mauricionunez/Desktop/anti-productivity-simulator/index.html`

---

## Overview

A single-file browser game disguised as a corporate SaaS productivity tool. The experience lasts 1-3 minutes, designed as a humorous mental break. Visual aesthetic: SNES-era Zelda - pixel fonts, limited color palette, 8-bit sound, RPG HUD.

No frameworks. No build step. No server required. Open `index.html` in any modern browser.

---

## Architecture

### Game State Machine
```
SCREEN_CORPORATE -> SCREEN_LOADING -> SCREEN_GAME -> (loop)
                                           |               \
                                    SCREEN_ACHIEVEMENT   SCREEN_GAME_OVER
                                        (overlay)
```

**State transitions:**
- SCREEN_CORPORATE -> SCREEN_LOADING: user clicks "Start Working"
- SCREEN_LOADING -> SCREEN_GAME: loading sequence completes (~4s)
- SCREEN_GAME -> SCREEN_ACHIEVEMENT: achievement unlocked (overlay, game continues)
- SCREEN_GAME -> SCREEN_GAME_OVER: focusPoints reaches 0

### Core State Object
```js
GameState = {
  screen: "corporate" | "loading" | "game" | "gameover",
  focusPoints: 100,
  timeWasted: 0,
  achievements: Set,
  currentInteraction: null,
  lastInteraction: null,      // avoid immediate repeat
  passiveDrainPaused: false,  // paused during INT-05
  cleanupFn: null             // teardown for active interaction
}
```

### Passive Focus Drain Rules
- Drain: -1 per 3 seconds during SCREEN_GAME
- Paused when passiveDrainPaused === true (INT-05 sets this)
- If focusPoints reaches 0 at ANY time (mid-interaction or between), call teardownCurrentInteraction() then show SCREEN_GAME_OVER

### Interaction Deduplication
- lastInteraction stores the ID of the most recently completed interaction
- loadRandomInteraction() picks randomly from all 8, excluding lastInteraction only (window of 1)

### Interaction Teardown Contract
Every interaction registers GameState.cleanupFn before setting up timers or listeners.
teardownCurrentInteraction() calls cleanupFn if present, then nulls it.
Called on: normal completion, game over, loading next interaction.
Each interaction clears its own DOM by emptying the interaction-area container.

### Game Over Screen
- Full black screen, pixel font
- "GAME OVER" in red, large
- Subtitle: "YOU DID ACTUAL WORK"
- Stats: time wasted MM:SS, interactions survived N, achievements N/6
- [PLAY AGAIN] button resets GameState and returns to SCREEN_CORPORATE

---

## Visual Design

### Color Palette
| Token | Hex | Use |
|-------|-----|-----|
| --green-dark | #1a3a1a | Game background |
| --gold | #c8a200 | HUD accents, borders |
| --beige | #f0e68c | Primary text |
| --red-dark | #8b0000 | Danger states, low focus |
| --black | #0a0a0a | Loading screen, shadows |
| --corporate-gray | #f5f5f5 | Corporate landing bg |
| --corporate-blue | #2563eb | Corporate button |

### Typography
- Game UI: `Press Start 2P` via Google Fonts CDN
- Corporate screen: system sans-serif

### Visual Effects (CSS only)
- Scanlines: ::after pseudo-element, repeating-linear-gradient, 2px lines, 30% opacity
- Pixel borders: box-shadow with multiple offsets (no border-radius in game UI)
- Glitch effect: CSS @keyframes with clip-path slices + translateX (corporate->loading)
- CRT flicker: opacity 0.97-1.0 animation, 4s cycle

### Sound (Web Audio API, no files)
- click: 880Hz square wave, 80ms
- achievement: C4-E4-G4-C5 arpeggio, square wave
- drain: 220Hz thud, 150ms
- complete: descending two-tone, 200ms
- meeting: dissonant chord, 300ms

---

## Screens

### 1. SCREEN_CORPORATE
- White bg, ProFlow logo, "Optimize your productivity" headline
- "Start Working" blue button
- On click: 500ms glitch animation -> SCREEN_LOADING

### 2. SCREEN_LOADING
- Black bg, pixel font sequence:
  1. "LOADING SAVE FILE..." (800ms)
  2. Progress bar fills unevenly (2000ms)
  3. Resets -> "FILE CORRUPT..." (600ms)
  4. "LOADING ANYWAY" (400ms)
  5. Fade to SCREEN_GAME

### 3. SCREEN_GAME

HUD top bar:
```
[ANTI-PRODUCTIVITY SIMULATOR]  [brain brain brain brain brain FOCUS]  [timer WASTED]
```
- 5 brain span elements, each = 20 focus points
- Brain states: healthy (brain emoji) -> dead (skull emoji) based on thresholds
- Timer counts up MM:SS

Interaction Area (center): Zelda dialog box, gold border on dark green
Message Bar (bottom): typewriter effect, ironic messages pool

### 4. SCREEN_GAME_OVER
- Game over stats, play again button (see Architecture section)

---

## The 8 Interactions

### INT-01: CLICK QUEST
Title: UNLOCK MOTIVATION
Mechanic: Button in center, counter "0 / 37 CLICKS". Click 37 times. At 37: pixel confetti animation (1s).
Focus penalty: -8
End message: "MOTIVATION NOT FOUND"
Achievement: CLICKED_WITHOUT_PURPOSE
Cleanup: remove click listener, clear DOM

### INT-02: ELUSIVE BUTTON
Title: SUBMIT REPORT
Mechanic: "SUBMIT" button teleports on mouseover (min 100px away, stays in bounds). After 15 seconds OR 10 teleports (whichever first), button freezes, becomes clickable.
Focus penalty: -10
End message: "REPORT SUCCESSFULLY AVOIDED"
Cleanup: remove mouseover listener, clear timeout, clear DOM

### INT-03: MEETING ESCAPE
Title: ENEMY ENCOUNTERED
Enemy rendering: 32x32px CSS div, background #c8a200, pixel eyes via box-shadow, label "MEETING" below in pixel font. No image files.
Mechanic: Enemy tracks cursor via requestAnimationFrame at 1.5px/frame. Keep cursor 40px+ away for 10 cumulative seconds. "Survive" progress bar fills over 10s, resets on contact. After 3 contacts, speed increases to 2.5px/frame. Win: 10s accumulated.
Focus penalty: -12
End message: "YOU SURVIVED: MEETING THAT COULD HAVE BEEN AN EMAIL"
Achievement: MEETING_SURVIVOR
Cleanup: cancelAnimationFrame, remove mousemove listener, clear DOM

### INT-04: RUPIA RAIN
Title: COLLECT RUPIAS
Rendering: Rotated square CSS divs. Gold (#c8a200) and green (#00a020). No images.
Mechanic: Rupias fall (CSS animation, 3-5s, random X). Click to catch. Gold increments goldCaught. Green resets goldCaught to 0. Win: goldCaught === 10. Timer: 20s.
Timer expiry: interaction ends with same -7 penalty, message "DROPPED EVERYTHING"
Focus penalty: -7
End messages: win="WALLET FULL, PRODUCTIVITY EMPTY" / timeout="DROPPED EVERYTHING"
Cleanup: clear spawn interval, remove click listeners, clear DOM

### INT-05: ETERNAL STANDUP
Title: DAILY STANDUP
Passive drain: PAUSED for duration (passiveDrainPaused = true). Resumes on completeInteraction().
Mechanic: Countdown from 0:30. At 0:00 -> "RESCHEDULED TO TOMORROW" -> resets. After 3 resets, [LEAVE MEETING] button appears. Click to complete.
Focus penalty: -6
End message: "FREEDOM ACHIEVED (TEMPORARILY)"
Cleanup: clear countdown interval, set passiveDrainPaused = false, clear DOM

### INT-06: PROGRESS BAR
Title: UPLOADING TPS REPORT
Mechanic: Bar fills to 99% in 4s, resets with "CONNECTION LOST". Resets 3 times. On 4th attempt, tooltip "DO NOT MOVE THE MOUSE" appears. Bar advances only when mouse is still for 500ms+; any movement resets bar to 0. Reach 100% to complete.
Focus penalty: -9
End message: "SCHRODINGER'S UPLOAD: COMPLETE"
Cleanup: clear fill interval, remove mousemove listener, clear DOM

### INT-07: INBOX ZERO QUEST
Title: INBOX
Mechanic: Email icon divs appear (max 8 visible). Click to delete. Each deleted spawns 3 more (300ms stagger). Tracks totalDeleted. Achievement INBOX_HERO triggers at totalDeleted === 20. Interaction force-completes at 25 seconds total.
Focus penalty: -11
End message: "INBOX: INFINITY"
Achievement: INBOX_HERO at 20 deleted
Cleanup: clear spawn timeouts, remove click listeners, clear DOM

### INT-08: TASK LABYRINTH
Title: NAVIGATE REQUIREMENTS
Mechanic: 5x5 CSS grid maze with border walls. Player = green square, WASD/arrow keys. One exit cell. On reaching exit: "WRONG EXIT - SEE ATTACHED: requirements_FINAL_v3.docx". Maze regenerates with new random layout and new exit position. Win condition: new exit position matches previous exit position (roughly 1-in-4 chance per attempt). Force-complete after 4 exit-reaches.
Focus penalty: -13
End message: "REQUIREMENTS UPDATED. START OVER."
Cleanup: remove keydown listener, clear DOM

---

## Focus Points Display
```
100-81: 5 brains (HUD green tint)
80-61:  4 brains + 1 skull
60-41:  3 brains + 2 skulls (slight red tint on HUD border)
40-21:  2 brains + 3 skulls (red tint)
20-1:   1 brain  + 4 skulls (HUD flickers)
0:      5 skulls -> GAME OVER immediately
```
Each icon is a span element. Emoji swapped by threshold. No grey intermediate state.

---

## Achievements
| ID | Title | Trigger |
|----|-------|---------|
| CLICKED_WITHOUT_PURPOSE | CLICKED WITHOUT PURPOSE | Complete INT-01 |
| MEETING_SURVIVOR | MEETING SURVIVOR | Complete INT-03 |
| PROFESSIONAL_PROCRASTINATOR | PROFESSIONAL PROCRASTINATOR | Complete 3 interactions |
| THE_FLOOR_IS_DEADLINES | THE FLOOR IS DEADLINES | Focus drops below 50 |
| INBOX_HERO | INBOX HERO | Delete 20 emails in INT-07 |
| SPEEDRUNNER | SPEEDRUNNER | Complete 5 interactions in <3 minutes |

Achievement chest: Two CSS divs (lid + body), brown bg, gold border. Lid rotates via CSS transform on trigger. Star character rises via @keyframes translateY. Auto-dismiss after 3s. Sound: arpeggio.

---

## Ironic Message Pool
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
STATE: GameState object
SCREENS: showCorporateScreen, showLoadingScreen, showGameScreen, showGameOverScreen
HUD: updateHUD, updateFocusDisplay, startTimerWasted
DRAIN: startPassiveDrain, applyFocusDelta(n)
ENGINE: loadRandomInteraction, completeInteraction(penalty, msg), teardownCurrentInteraction
INTERACTIONS: interactionClickQuest, interactionElusiveButton, interactionMeetingEscape,
              interactionRupiaRain, interactionEternalStandup, interactionProgressBar,
              interactionInboxZero, interactionTaskLabyrinth
REGISTRY: INTERACTIONS array of {id, fn} objects
ACHIEVEMENTS: checkAchievements, showAchievement(id)
SOUND: playSound(type) via Web Audio API
MESSAGES: typewriterMessage(text)
```

---

## Adding New Interactions
1. Write interactionMyNew() - must set GameState.cleanupFn
2. Call completeInteraction(penalty, message) when done
3. Add { id: 'my_new', fn: interactionMyNew } to INTERACTIONS array
4. Optionally call showAchievement() inside the function

---

## Out of Scope (MVP)
- Mobile touch support
- Session persistence
- Multiplayer / leaderboard
- External asset files
