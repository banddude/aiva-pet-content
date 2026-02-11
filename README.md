# AIVA Pet Content Repository

Remote content system for the AIVA desktop pet app. Everything in this repo is fetched at launch and injected into the running app. Edit a JSON file, push, and the pet updates on next launch. No app recompile needed.

## How It Works

The app fetches `manifest.json` from this repo on startup. The manifest points to all the content files. The app loads each one and injects it into the WebView engine.

```
manifest.json          -- entry point, lists everything
intro.json             -- startup animation
cartoon.json           -- "Play Cartoon" sequence
music.json             -- chiptune melody for cartoon
packs/                 -- content packs (seasonal, accessories, etc.)
  valentines.json
```

## manifest.json

The hub. Every content file is referenced here.

```json
{
  "version": 3,
  "intro": "intro.json",
  "cartoon": "cartoon.json",
  "music": "music.json",
  "packs": [
    {
      "id": "valentines",
      "name": "Valentine's Day",
      "version": 1,
      "file": "packs/valentines.json",
      "active_from": "02-01",
      "active_until": "02-28"
    }
  ]
}
```

### Pack date ranges

`active_from` and `active_until` use `MM-DD` format. Packs outside the date range are skipped. Wrapping around year end works (e.g. `"active_from": "12-15", "active_until": "01-05"`). Omit both to make a pack always active.

## intro.json

Controls what happens when the app launches.

```json
{
  "id": "default-intro",
  "version": 1,
  "name": "Skateboard Kickflip Intro",
  "greeting": "Hi there!",
  "greeting_expression": "happy",
  "greeting_duration": 1.0,
  "delay": 0.3,
  "js": "bunny.facingRight = true; if (!bunny.hasSkateboard) toggleSkateboard(); bunny.introMode = true; bunny.introWraps = 0; startKickflip();"
}
```

| Field | Description |
|---|---|
| `greeting` | Speech bubble text shown on launch |
| `greeting_expression` | Expression during greeting (happy, surprised, etc.) |
| `greeting_duration` | How long the greeting bubble stays (seconds) |
| `delay` | Delay before running the JS (seconds) |
| `js` | JavaScript to execute for the intro animation |

## cartoon.json

The "Play Cartoon" timeline. An array of timed actions.

```json
{
  "id": "meet-aiva",
  "version": 1,
  "name": "Meet AIVA",
  "script": [
    { "t": 0, "action": "position", "x": -20 },
    { "t": 0.1, "action": "walk", "target": 180 },
    { "t": 2.5, "action": "stop" },
    { "t": 3.0, "action": "say", "text": "Hi!", "duration": 2.0 },
    { "t": 60.0, "action": "end" }
  ]
}
```

### Cartoon actions

| Action | Fields | Description |
|---|---|---|
| `position` | `x` | Teleport bunny to x position |
| `walk` | `target` | Walk to x position |
| `stop` | | Stop walking |
| `say` | `text`, `duration` | Show speech bubble and speak with Flo voice |
| `wave` | `type` (left, right, both) | Wave animation |
| `jump` | | Jump |
| `kickflip` | | Kickflip on skateboard |
| `look` | `dir`, `duration` | look_left, look_right, look_up, look_down |
| `expression` | `name`, `duration` | happy, sad, surprised, thinking, sleeping, etc. |
| `face` | `value` (true/false) | Face right (true) or left (false) |
| `sunglasses` | `value` (true/false) | Toggle sunglasses |
| `skateboard` | `value` (true/false) | Toggle skateboard |
| `accessory` | `id`, `value` (true/false) | Toggle a content pack accessory |
| `daynight` | `value` (true/false) | Day (true) or night (false) |
| `blink` | | Quick blink |
| `end` | | End the cartoon |

## music.json

Chiptune melody played during the cartoon.

```json
{
  "id": "bouncy-chiptune",
  "version": 1,
  "name": "Cute Bouncy Chiptune",
  "bps": 3.2,
  "volume": 0.08,
  "waveform": "square",
  "melody": [
    ["C5", 1], ["E5", 1], ["G5", 1], ["R", 2]
  ],
  "frequencies": {
    "C5": 523.25, "D5": 587.33, "E5": 659.25,
    "F5": 698.46, "G5": 783.99, "A5": 880,
    "B5": 987.77, "C6": 1046.5, "R": 0
  }
}
```

| Field | Description |
|---|---|
| `bps` | Beats per second (tempo) |
| `volume` | 0.0 to 1.0 |
| `waveform` | Web Audio oscillator type: square, sine, triangle, sawtooth |
| `melody` | Array of [note, beats] pairs. "R" is a rest. |
| `frequencies` | Hz values for each note name |

## Content Packs

Packs are the main way to add new content. Each pack is a JSON file with a `js` field containing JavaScript that runs inside the pet's WebView. Packs have full access to the engine.

### Pack JSON structure

```json
{
  "id": "my-pack",
  "version": 1,
  "name": "My Pack",
  "greeting": "Hello from my pack!",
  "expression": "happy",
  "js": "(function(){ /* your code */ })()"
}
```

| Field | Description |
|---|---|
| `id` | Unique identifier |
| `greeting` | Optional speech bubble shown after intro settles |
| `expression` | Expression during greeting |
| `js` | JavaScript code injected into the WebView |

### What packs can do

Packs have access to all engine globals. Wrap your code in an IIFE to avoid conflicts.

#### Add custom sprites

```javascript
var _=null, P='P', W='W', D='D';
var myGrid = [
  [_,_,P,P,P,P,P,P,P,P,P,P,P,P,P,P,_,_],
  // ... 15 rows, 18 columns
];
spriteCache['mysprite'] = renderSprite(myGrid, false);
spriteCache['mysprite_flip'] = renderSprite(myGrid, true);
```

Sprite grids are 18 columns x 15 rows. Use the color shortcuts P (pink), W (white), D (dark), L (lavender), B (black), or hex strings like '#ff4477'. Use null for transparent.

#### Add expressions

```javascript
expressionKeys['Digit8'] = 'love';  // press 8 to trigger
```

#### Add idle behaviors

```javascript
if (!window._contentBehaviors) window._contentBehaviors = [];
window._contentBehaviors.push({
  action: 'myaction',
  cat: 'expression',  // category: expression, jump, wander, idle
  w: 6                // weight (higher = more likely)
});
```

#### Add action handlers

```javascript
if (!window._contentActions) window._contentActions = {};
window._contentActions['myaction'] = function(b) {
  b.expression = 'love';
  b.expressionTimer = 2.0;
  b.lastAutoCategory = 'expression';
};
```

#### Add visual overlays

```javascript
if (!window._contentOverlays) window._contentOverlays = [];
window._contentOverlays.push(function(ctx, b, bx, by, px, colors, gameTime) {
  if (b.expression !== 'love') return;
  // draw something using ctx (canvas 2d context)
  // bx, by = bunny top-left position
  // px = pixel size
});
```

#### Add per-frame update hooks

```javascript
if (!window._contentUpdateHooks) window._contentUpdateHooks = [];
window._contentUpdateHooks.push(function(b, dt) {
  // runs every frame
  // b = bunny object, dt = delta time in seconds
});
```

#### Add combo chains

```javascript
if (!window._contentCombos) window._contentCombos = [];
window._contentCombos.push(['kickflip', 'wave_both', 'myaction']);
```

#### Add toggleable accessories (hats, bows, etc.)

```javascript
var _=null, P='P';
var hatGrid = [
  [_,_,_,_,_,P,P,P,P,P,P,P,P,_,_,_,_,_],
  [_,_,_,P,P,P,P,P,P,P,P,P,P,P,P,_,_,_],
  [_,_,_,P,P,P,P,P,P,P,P,P,P,P,P,_,_,_],
];

window._contentAccessoryDefs['tophat'] = {
  sprite: hatGrid,     // pixel grid (same format as sprites)
  position: 'head',    // 'head' (above), 'face' (over face), 'body' (over body)
  offsetX: 0,          // horizontal offset in PX units
  offsetY: 5,          // vertical offset in PX units
  key: 'KeyH'          // keyboard shortcut to toggle
};
```

Accessory functions available:
- `toggleAccessory('tophat')` to toggle on/off
- `bunny.accessories['tophat']` to check state
- Cartoon action: `{ "action": "accessory", "id": "tophat", "value": true }`

## Available globals

These are available inside pack JS code:

| Global | Description |
|---|---|
| `bunny` | The bunny state object (x, y, vx, vy, expression, facingRight, etc.) |
| `spriteCache` | Object mapping sprite names to pre-rendered canvases |
| `renderSprite(grid, flip)` | Renders a pixel grid to a canvas. flip=true mirrors horizontally |
| `COLORS` | Color palette: `{ P: '#ff96be', W: '#ffffff', D: '#2d233c', L: '#a08cd2', B: '#000000' }` |
| `PX` | Pixel size (4 in desktop mode, 8 in web mode) |
| `GY` | Ground Y position |
| `expressionKeys` | Object mapping key codes to expression names |
| `isDay` | Boolean, current day/night state |
| `toggleSunglasses()` | Toggle sunglasses |
| `toggleSkateboard()` | Toggle skateboard |
| `toggleDayNight()` | Toggle day/night |
| `toggleAccessory(id)` | Toggle a custom accessory |
| `setExpression(name)` | Set bunny expression |
| `startJump()` | Make bunny jump |
| `startKickflip()` | Start kickflip animation |
| `startWave(type)` | Wave: 'left', 'right', 'both' |
| `speak(text)` | Speak text with Flo voice (via Swift AVSpeechSynthesizer) |
| `gameTime` | Current game time in seconds |

## TTS Voice

All speech uses the Flo (English US) voice via macOS AVSpeechSynthesizer. The app automatically replaces "AIVA" with "Aivah" for correct pronunciation. Display text is unchanged.

## What stays in the app (engine)

These are compiled into the app and not loaded from GitHub:

- Base bunny sprites (standing, walking, blinking, expressions)
- Rendering engine (canvas, draw loop)
- Physics (gravity, jumping, walking)
- Window management, mouse tracking
- Keyboard forwarding from Swift to WebView
- Day/night toggle, star field
- Speech recognition + TTS (Flo voice)
- WebSocket companion client
- The content fetcher itself

Everything else comes from this repo.
