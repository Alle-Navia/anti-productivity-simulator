# Anti-Productivity Simulator

A browser game that pretends to be a corporate productivity tool. Zelda SNES aesthetic.

## Play

Open index.html in any modern browser. No server needed.

## Add a new interaction

1. Write interactionMyNew() — set GameState.cleanupFn first
2. Call completeInteraction(penalty, message) when done
3. Push { id: 'my_id', fn: interactionMyNew } to the INTERACTIONS array
