# Dive with Jeni

**Open:** double-click `index.html` (works offline over file://, no install, no network).
Everything (HTML, CSS, JS, Canvas 2D, WebAudio music and sound effects) lives in that one file.

**Controls**
- Mouse: move the pointer, Jeni swims toward it. Touch: drag anywhere on the screen.
- Keyboard: arrow keys or WASD.
- Catch only the fish in the task panel (gold ring marks them) for EXP. Avoid every hazard (-1 heart).
- 90 seconds, 3 hearts; the run ends on whichever comes first.

## Species and EXP

Task fish (only these can be task targets):

| id | 中文 | English | EXP | Notes | Music mood |
|----|------|---------|-----|-------|-----------|
| `nemo` | 小丑鱼 | Nemo | 10 | small, fast, common | cute |
| `dory` | 蓝刀鱼 | Dory | 25 | small | cute |
| `butterfly` | 蝴蝶鱼 | Butterflyfish | 35 | small, yellow/black, fast | cute |
| `puffer` | 河豚 | Pufferfish | 50 | slow | - |
| `angelfish` | 神仙鱼 | Angelfish | 55 | medium, streaked fins | - |
| `seahorse` | 海马 | Seahorse | 70 | small, upright, drifts and bobs | cute |
| `turtle` | 海龟 | Sea turtle | 80 | large, slow | upbeat |
| `manta` | 魔鬼鱼 | Manta ray | 120 | large | upbeat |
| `dolphin` | 海豚 | Dolphin | 150 | large, FAST, swims in pairs, leaps in an arc with a bubble trail | upbeat |
| `whale` | 鲸鲨 | Whale shark | 300 | huge, rare | upbeat |

Big species are rarer and unlock later (`taskUnlock` = number of cleared tasks needed before the
species can be asked for): turtle 2, manta 4, dolphin 5, whale 7.

## Hazards

Every species flagged `hazard: true` in `CONFIG.species` costs **1 heart** on contact, plays the damage
sound, vibrates, shakes the screen, flashes red and grants 1.5 s of invulnerability. They all go through
one code path (`applyHit(sourceId)`) and are never chosen as task targets.

| id | 中文 | English | Behaviour |
|----|------|---------|-----------|
| `trigger` | 触发鱼 | Trigger fish | spiky, patrols across the screen (from the start) |
| `shark` | 鲨鱼 | Shark | large, grey, dorsal fin + teeth, patrols faster than other big fish (from ~30% of the run) |
| `jellyfish` | 水母 | Jellyfish | translucent glowing dome, floats up and down slowly (from ~10%) |
| `urchin` | 海胆 | Sea urchin | spiky dark-purple ball, sits on the seabed for ~16 s (from ~15%) |

To add a hazard, add a species entry with `hazard: true`, `hazWeight` and `hazStart`, plus a draw function.

## Music (all generated in code with WebAudio, no audio files)

Three moods, all in `CONFIG.music.moods` (tempo, chords, scale, melody, gains, filter cutoff, reverb wet):

| mood | style | tempo |
|------|-------|-------|
| `dreamy` | default; magical underwater mermaid waltz in F major (I-vi-IV-V): harp/glockenspiel arpeggios, warm pad, low root bass, 3/4 | ~69 BPM |
| `upbeat` | celebratory and driving: bouncy bass, off-beat chord stabs, bright lead, shaker on the off-beats, 4/4 | ~124 BPM |
| `cute` | playful, slightly swung: marimba/celesta plucks, a melody hopping between high notes, light hi-hats | ~112 BPM |

- The start screen is silent. **Start** begins `dreamy`; it is also the resting mood between bursts.
- Each species has a `mood` field. Catching a big fish (turtle / manta / dolphin / whale) triggers `upbeat`;
  catching a small cute fish (nemo / dory / butterfly / seahorse) triggers `cute`; other catches change nothing.
- A mood burst lasts `CONFIG.music.moodSeconds` (10 s) and then eases back to `dreamy`. A new trigger during
  a burst replaces it immediately (0.7 s crossfade).
- Game over stops the music and plays a short descending motif. **Play again** restarts `dreamy`.
- Taking damage briefly ducks the music (about -45% volume and a darker filter for ~0.35 s) so the hit sound cuts through.
- Engine: lookahead scheduler (`setInterval` 25 ms, notes scheduled ~120 ms ahead on the AudioContext clock),
  one master gain, a light feedback delay as underwater reverb, a sweepable lowpass, a cap of ~24 oscillators.
  The AudioContext is created on the first user gesture. If WebAudio is unavailable the game plays silently.

## Sound effects

Normal catch = soft pop + sparkle; big catch (EXP >= `CONFIG.sfx.bigExp`) = rising flourish; task cleared =
rising chime; damage = low thud + buzzy noise; game over = descending motif. Volume: `CONFIG.sfx.volume`
(and `CONFIG.audio.volume` for the master level).

## Sound / vibration toggle

The 🔊 / 🔇 button (HUD top-right, and on the start screen) is **one switch for sound AND vibration**.
The choice is saved in `localStorage` (`diveWithJeniSound`). When muted: music stops, sound effects are
skipped and vibrations are skipped. The game loop is unaffected.

## Vibration

Every damaging hit calls `navigator.vibrate([45, 55, 45])` (guarded by a feature check). **iOS Safari does not
implement `navigator.vibrate`, so nothing vibrates there - that is expected.** The screen shake and the red
flash always fire, so damage feedback is never missing. Android Chrome vibrates once you have tapped the page.

**Tuning:** edit the `const CONFIG = {...}` block at the top of the `<script>` in `index.html`.
Every field is commented in Chinese + English: run time, hearts, invulnerability, diver speed,
spawn rates, hazard counts, difficulty ramp, music moods, sfx, and the species table
(size, speed, EXP, rarity weights, task counts, mood).

## Test hook

`window.DIVE_DEBUG` (read-only):

- `state`, `exp`, `hearts`, `timeLeft`, `taskCount`, `entityCount`, `fps`
- `forceCatch(id)`, `forceHit([hazardId])` (default `trigger`), `skipToEnd()`
- `audio` = `{ mood, level, ctxState, voices, musicOn, lastSfx }`
  - `mood`: current mood id (`dreamy` / `upbeat` / `cute`, or `none` on the start and game-over screens)
  - `level`: current output level 0..1, peak measured through an AnalyserNode on the master gain (0 while muted)
  - `ctxState`: AudioContext state (`none` if it has not been created yet)
  - `voices`: live oscillators; `musicOn`: music is actually playing; `lastSfx`: `catch` / `bigCatch` / `clear` / `hurt` / `gameover`
- `setMood(id)`: force a mood (stays until the next game event)
- `toggleSound()`: same as clicking the 🔊 button; `soundOn`: current state
- `vibrateLog`: array of patterns passed to `navigator.vibrate` (cleared on each new run)
- `damageLog`: array of `{ source: speciesId, heartsAfter }` for each damaging hit (cleared on each new run)
