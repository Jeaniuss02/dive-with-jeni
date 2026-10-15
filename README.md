# Dive with Jeni

**Open:** double-click `index.html` (works offline over file://, no install, no network).

**Controls**
- Mouse: move the pointer, Jeni swims toward it. Touch: drag anywhere on the screen.
- Keyboard: arrow keys or WASD.
- Catch only the fish in the task panel (gold ring marks them) for EXP. Avoid spiky trigger fish (-1 heart).
- 90 seconds, 3 hearts; the run ends on whichever comes first.

**Tuning:** edit the `const CONFIG = {...}` block at the top of the `<script>` in `index.html`.
Every field is commented in Chinese + English: run time, hearts, invulnerability, diver speed,
spawn rates, hazard counts, difficulty ramp, and the species table (size, speed, EXP, rarity weights, task counts).

**Test hook:** `window.DIVE_DEBUG` exposes state, exp, hearts, timeLeft, taskCount, entityCount, fps,
forceCatch(id), forceHit(), skipToEnd().
