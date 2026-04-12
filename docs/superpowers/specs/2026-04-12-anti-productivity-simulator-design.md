# Anti-Productivity Simulator — Design Spec
**Date:** 2026-04-12  
**Status:** Approved  
**Output:** `/home/mauricionunez/Desktop/anti-productivity-simulator/index.html`

---

## Overview

A single-file browser game disguised as a corporate SaaS productivity tool. 1-3 minute humorous mental break. Visual aesthetic: SNES-era Zelda — pixel fonts, limited palette, 8-bit sound, RPG HUD. No frameworks. No build step. Open index.html in any modern browser.

---

## Architecture

### Game State Machine
```
SCREEN_CORPORATE -> SCREEN_LOADING -> SCREEN_GAME -> (loop)
                                           |               \
                                    SCREEN_ACHIEVEMENT   SCREEN_GAME_OVER
                                        (overlay)
```

### Core State Object
```js
const GameState = {
  screen: "corporate",       // "corporate"|"loading"|"game"|"gameover"
  focusPoints: 100,
  timeWasted: 0,             // seconds since game start
  interactionCount: 0,       // increments on completeInteraction()
  achievements: new Set(),
  currentInteraction: null,  // active interaction ID string
  lastInteraction: null,     // previous interaction ID (dedup window of 1)
  passiveDrainPaused: false, // true only during INT-05
  cleanupFn: null,           // set by each interaction, called on teardown
  firstInteractionTime: null // timestamp when first interaction completes (for SPEEDRUNNER)
}
```

### Passive Focus Drain
- -1 every 3 seconds during SCREEN_GAME
- Paused when passiveDrainPaused === true
- Drain sound plays only when passive drain fires (not on interaction penalties)
- If focusPoints reaches 0: teardownCurrentInteraction() then show SCREEN_GAME_OVER

### Interaction Teardown Contract
Every interaction sets GameState.cleanupFn before doing anything else.
teardownCurrentInteraction() calls GameState.cleanupFn() then sets it to null.
Called on: normal completion, game over trigger, loading next interaction.
Each cleanup must: clear all setInterval/setTimeout, remove all event listeners added, clear the interaction-area container DOM.

### Interaction Deduplication
loadRandomInteraction() picks randomly from 8, excluding GameState.lastInteraction only.

### Game Over Screen
- Black bg, "GAME OVER" red pixel text
- "YOU DID ACTUAL WORK"
- Stats: TIME WASTED MM:SS, INTERACTIONS SURVIVED (GameState.interactionCount), ACHIEVEMENTS N/6
- [PLAY AGAIN] resets GameState to defaults, calls showCorporateScreen()

---

## Visual Design

### Color Palette
| Token | Hex | Use |
|-------|-----|-----|
| --green-dark | #1a3a1a | Game background |
| --gold | #c8a200 | HUD accents, borders |
| --beige | #f0e68c | Primary text |
| --red-dark | #8b0000 | Danger states |
| --black | #0a0a0a | Loading screen |
| --corporate-gray | #f5f5f5 | Corporate bg |
| --corporate-blue | #2563eb | Corporate button |

### Typography
- Game UI: Press Start 2P via Google Fonts CDN
- Corporate screen: system sans-serif

### Visual Effects (CSS only)
- Scanlines: ::after pseudo-element, repeating-linear-gradient 2px, 30% opacity
- Pixel borders: box-shadow offsets (no border-radius in game UI)
- Glitch: @keyframes clip-path slices + translateX (corporate->loading transition)
- CRT flicker: opacity 0.97-1.0, 4s cycle

### Sound (Web Audio API, no files)
- "click": 880Hz square, 80ms — played on user click in interactions
- "achievement": C4-E4-G4-C5 arpeggio, square wave — played by showAchievement()
- "drain": 220Hz thud, 150ms — played only when passive drain fires (-1 per 3s tick)
- "complete": descending two-tone 200ms — played by completeInteraction() before penalty
- "meeting": dissonant chord 300ms — played at start of INT-03

---

## Screens

### 1. SCREEN_CORPORATE
White bg, ProFlow logo, "Optimize your productivity" headline, "Start Working" blue button.
On click: 500ms CSS glitch animation -> showLoadingScreen()

### 2. SCREEN_LOADING
Black bg, pixel font:
1. "LOADING SAVE FILE..." (800ms)
2. Progress bar fills unevenly via setInterval with random increments (2000ms total)
3. Resets -> "FILE CORRUPT..." (600ms)
4. "LOADING ANYWAY" (400ms)
5. Fade to showGameScreen()

### 3. SCREEN_GAME
HUD top bar: title | brain icons | timer
Interaction area: Zelda dialog box, gold border on dark green, id="interaction-area"
Message bar bottom: typewriter messages, displayed by typewriterMessage() after each completeInteraction() call

### 4. SCREEN_GAME_OVER (see Architecture)

---

## Message Bar Trigger Spec
typewriterMessage(text) is called by completeInteraction(penalty, msg) immediately after teardown.
It also displays a random ironic message from the pool after a 500ms delay following the interaction-specific end message.
Not displayed at any other time (not on passive drain, not randomly).

---

## The 8 Interactions

### INT-01: CLICK QUEST
Title: UNLOCK MOTIVATION
Mechanic: Central button, counter "0 / 37 CLICKS". Click to increment. At 37: 1s pixel confetti (CSS animation on small colored divs), then auto-complete.
Focus penalty: -8
End message: "MOTIVATION NOT FOUND"
Achievement trigger: CLICKED_WITHOUT_PURPOSE
Cleanup: remove click listener, clear DOM

### INT-02: ELUSIVE BUTTON
Title: SUBMIT REPORT
Mechanic: "SUBMIT" button inside interaction area. On mouseover, teleports to random position (min 100px away, clamped to container bounds). Teleport counter shown: "ATTEMPTS REMAINING: 10". After 15 seconds elapsed OR 10 teleports, button stops moving. Text changes to "OK FINE, CLICK ME". Player must click the frozen button to complete. completeInteraction() is called on that click.
Focus penalty: -10
End message: "REPORT SUCCESSFULLY AVOIDED"
Cleanup: remove mouseover listener, remove click listener, clear timeout, clear DOM

### INT-03: MEETING ESCAPE
Title: ENEMY ENCOUNTERED
Sound: playSound("meeting") on start
Enemy rendering: 32x32px CSS div, background #c8a200, dark 4x4 "eyes" via box-shadow, label "MEETING" text below. No images.
Movement: requestAnimationFrame, enemy moves toward cursor position at 1.5px/frame (measured as Euclidean distance each frame, normalized direction vector * speed).
Contact definition: distance between cursor (x,y) and enemy center < 40px.
Mechanic: Survive timer counts up. Contact resets timer. Goal: 10 cumulative seconds without contact. After 3 contacts, speed becomes 2.5px/frame. Win: 10s accumulated. Progress bar shows 0-10s visually.
Focus penalty: -12
End message: "YOU SURVIVED: MEETING THAT COULD HAVE BEEN AN EMAIL"
Achievement: MEETING_SURVIVOR
Cleanup: cancelAnimationFrame (store rAF ID in closure), remove mousemove listener, clear DOM

### INT-04: RUPIA RAIN
Title: COLLECT RUPIAS
Rendering: 20x20px divs rotated 45deg. Gold (#c8a200) and green (#00a020).
Click detection: pointer-events: auto on each rupia div. Click fires on the div itself (standard DOM click). No proximity calculation needed — CSS transform does not break pointer-events on the element bounds.
Mechanic: New rupia spawns every 800ms at random X (0 to container width - 20px), falls via CSS transition translateY to container bottom over 3-5s (random per rupia). On click: gold increments goldCaught, green resets goldCaught to 0. Win: goldCaught === 10. Timer: 20s.
Timer expiry: completeInteraction(-7, "DROPPED EVERYTHING")
Focus penalty: -7
Win end message: "WALLET FULL, PRODUCTIVITY EMPTY"
Cleanup: clear spawn interval, remove all rupia divs, clear DOM

### INT-05: ETERNAL STANDUP
Title: DAILY STANDUP
Passive drain: passiveDrainPaused = true on start, false on complete
Mechanic: Countdown timer from 0:30. At 0:00: flash "RESCHEDULED TO TOMORROW" (1s), reset to 0:30. Repeat 3 times. After 3rd reset: [LEAVE MEETING] button appears. Click it: completeInteraction().
Focus penalty: -6
End message: "FREEDOM ACHIEVED (TEMPORARILY)"
Cleanup: clear countdown interval, set passiveDrainPaused = false, clear DOM

### INT-06: PROGRESS BAR
Title: UPLOADING TPS REPORT
Mechanic:
- Attempts 1-3: bar fills at 25% per second (4s to reach 100%), but is hard-capped at 99% and resets to 0 with "CONNECTION LOST" text.
- Attempt 4: tooltip "DO NOT MOVE THE MOUSE" appears. Bar advances at 10% per second ONLY while mouse has not moved for 500ms+ (measured via mousemove listener resetting a stillness timer). Any mousemove: bar resets to 0 and stillness timer restarts. Reach 100%: completeInteraction().
Focus penalty: -9
End message: "SCHRODINGER'S UPLOAD: COMPLETE"
Cleanup: clear fill interval, clear stillness timeout, remove mousemove listener, clear DOM

### INT-07: INBOX ZERO QUEST
Title: INBOX
Spawn cap: max 8 email divs visible at once. Spawn queue holds pending emails.
Logic: Start with 5 emails. On click-delete: remove that div, add 3 to spawn queue, increment totalDeleted. Dequeue from spawn queue one-at-a-time every 300ms, only if visible count < 8.
Achievement: INBOX_HERO triggers when totalDeleted === 20 (before force-complete).
Force-complete: at 25 seconds total, completeInteraction() fires regardless of count.
Focus penalty: -11
End message: "INBOX: INFINITY"
Cleanup: clear spawn interval, clear dequeue timeout, remove all email divs, clear DOM

### INT-08: TASK LABYRINTH
Title: NAVIGATE REQUIREMENTS
Win condition: The force-complete at 4 exit-reaches IS the real completion path. The matching-exit flavor mechanic is secondary. Specifically: on each exit-reach, show the flavor message ("WRONG EXIT..."), regenerate the maze, and track exitCount. At exitCount === 4: completeInteraction(). The matching-exit mechanic is NOT a win condition - it is removed from the spec for clarity.
Mechanic: 5x5 CSS grid. Walls defined as a hardcoded array of 4 maze layouts (pre-defined, not random) cycled in order. Player = green 100% sized div in its cell. WASD/arrow keys move player one cell at a time (preventDefault on arrow keys). One exit cell per maze (different position per layout). Player cannot move through wall cells. On reaching exit cell: show "WRONG EXIT - SEE ATTACHED: requirements_FINAL_v3.docx" for 1s, load next maze layout, increment exitCount, reset player to top-left. At exitCount === 4: completeInteraction().
Focus penalty: -13
End message: "REQUIREMENTS UPDATED. START OVER."
Cleanup: remove keydown listener, clear timeout, clear DOM

---

## Focus Points Display
```
100-81: 5 brains (HUD green tint)
80-61:  4 brains + 1 skull
60-41:  3 brains + 2 skulls (slight red tint on HUD border)
40-21:  2 brains + 3 skulls (red tint)
20-1:   1 brain  + 4 skulls (HUD flickers via CSS animation)
0:      5 skulls -> GAME OVER immediately
```
Each icon is a span. Brain emoji = alive, skull emoji = dead. Threshold brackets computed from focusPoints value.

---

## Achievements
| ID | Title | Trigger |
|----|-------|---------|
| CLICKED_WITHOUT_PURPOSE | CLICKED WITHOUT PURPOSE | completeInteraction from INT-01 |
| MEETING_SURVIVOR | MEETING SURVIVOR | completeInteraction from INT-03 |
| PROFESSIONAL_PROCRASTINATOR | PROFESSIONAL PROCRASTINATOR | interactionCount === 3 |
| THE_FLOOR_IS_DEADLINES | THE FLOOR IS DEADLINES | focusPoints drops below 50 |
| INBOX_HERO | INBOX HERO | totalDeleted === 20 in INT-07 |
| SPEEDRUNNER | SPEEDRUNNER | interactionCount === 5 AND timeWasted < 180 |

checkAchievements() is called inside completeInteraction() and inside applyFocusDelta().
Each achievement fires only once (checked against GameState.achievements Set).

Achievement display: Two CSS divs (lid + body). Lid rotates -60deg via CSS transform on trigger. Star character "★" rises via @keyframes translateY(-40px), then auto-dismiss after 3s (setTimeout removes overlay). Sound: playSound("achievement").

---

## Ironic Message Pool
Displayed by completeInteraction() -> typewriterMessage() after each interaction ends.
One random message selected from:
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
STATE: GameState object (as defined above)

SCREENS:
  showCorporateScreen()
  showLoadingScreen()
  showGameScreen()      // starts passive drain, starts timer, loads first interaction
  showGameOverScreen()  // stops timer, stops drain, renders stats

HUD:
  updateHUD()           // calls updateFocusDisplay() + updates timer display
  updateFocusDisplay()  // updates brain/skull spans based on focusPoints thresholds
  startTimerWasted()    // setInterval every 1s, increments GameState.timeWasted, calls updateHUD

DRAIN:
  startPassiveDrain()   // setInterval every 3s, checks passiveDrainPaused, calls applyFocusDelta(-1), playSound("drain")
  applyFocusDelta(n)    // GameState.focusPoints += n, clamp to 0, call updateFocusDisplay(), checkAchievements(), if 0 then showGameOverScreen()

ENGINE:
  loadRandomInteraction()             // filter lastInteraction, pick random, call its fn
  completeInteraction(penalty, msg)   // playSound("complete"), teardown, applyFocusDelta(penalty), typewriterMessage(msg), checkAchievements(), GameState.interactionCount++, GameState.lastInteraction = id, setTimeout 2000ms then loadRandomInteraction()
  teardownCurrentInteraction()        // call GameState.cleanupFn() if exists, set to null

INTERACTIONS (each sets GameState.cleanupFn):
  interactionClickQuest()
  interactionElusiveButton()
  interactionMeetingEscape()
  interactionRupiaRain()
  interactionEternalStandup()
  interactionProgressBar()
  interactionInboxZero()
  interactionTaskLabyrinth()

REGISTRY:
  const INTERACTIONS = [
    { id: 'click_quest', fn: interactionClickQuest },
    { id: 'elusive_button', fn: interactionElusiveButton },
    { id: 'meeting_escape', fn: interactionMeetingEscape },
    { id: 'rupia_rain', fn: interactionRupiaRain },
    { id: 'eternal_standup', fn: interactionEternalStandup },
    { id: 'progress_bar', fn: interactionProgressBar },
    { id: 'inbox_zero', fn: interactionInboxZero },
    { id: 'task_labyrinth', fn: interactionTaskLabyrinth },
  ]

ACHIEVEMENTS:
  checkAchievements()     // check all conditions against GameState, skip already-unlocked
  showAchievement(id)     // render chest overlay, play sound, auto-dismiss 3s

SOUND:
  playSound(type)         // type: "click"|"achievement"|"drain"|"complete"|"meeting"
                          // Web Audio API: create OscillatorNode, set frequency/type, connect to destination, start/stop

MESSAGES:
  typewriterMessage(text) // clears bottom bar, writes one character every 50ms
```

---

## Adding New Interactions
1. Write interactionMyNew() - first line must set GameState.cleanupFn
2. Call completeInteraction(penalty, message) when done
3. Add { id: 'my_new', fn: interactionMyNew } to INTERACTIONS array
4. Optionally call showAchievement('ID') inside the function

---

## Out of Scope (MVP)
- Mobile touch support
- Session persistence
- Multiplayer / leaderboard
- External asset files
