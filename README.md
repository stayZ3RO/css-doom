# cssDOOM

> I rebuilt Niels Leenheer's cssDOOM (https://nielsleenheer.com/articles/2026/css-is-doomed/) as a Vite-based static site, with GitHub Actions deploying it to GitHub Pages. Niels Leenheer created the rendering engine, game logic, and CSS techniques described below, based on id Software's DOOM. I added the build and deployment pipeline. The site is live at https://stayz3ro.github.io/css-doom/.

A recreation of the original DOOM rendered entirely with CSS. Every wall, floor, sprite, and effect is a styled DOM element positioned in 3D space via CSS transforms and `preserve-3d`, without `<canvas>` or WebGL.

The game logic is written in JavaScript using id Software's [open-source release](https://github.com/id-Software/DOOM) as a reference.

**[Play it live at cssdoom.wtf](https://cssdoom.wtf)**

**[Read the blog post](https://nielsleenheer.com/articles/2026/css-is-doomed/)**

## Run locally

```sh
npm ci
npm run dev
```

For a production build, run `npm run build -- --base=/css-doom/`, the same base path used for GitHub Pages.

## How it works

The renderer starts with the linedefs, sidedefs, and sectors from the DOOM WAD file. It builds a scene from `<div>` elements placed in 3D space with CSS transforms. JavaScript supplies the raw DOOM vertex geometry as custom properties; CSS computes the transforms rather than receiving finished positions.

```html
<div class="wall" style="
  --start-x: 2560;
  --start-y: -2112;
  --end-x: 2560;
  --end-y: -2496;
  --floor-z: 32;
  --ceiling-z: 88;
">
```

CSS calculates the correct width, height and 3D transforms using trigonometry functions:

```css
.wall {
    --delta-x: calc(var(--end-x) - var(--start-x));
    --delta-y: calc(var(--end-y) - var(--start-y));

    width: calc(hypot(var(--delta-x), var(--delta-y)) * 1px);
    height: calc((var(--ceiling-z) - var(--floor-z)) * 1px);

    transform:
        translate3d(
            calc(var(--start-x) * 1px),
            calc(var(--ceiling-z) * -1px),
            calc(var(--start-y) * -1px)
        )
        rotateY(atan2(var(--delta-y), var(--delta-x)));
}
```

DOOM's coordinate system doesn't map directly to CSS 3D. DOOM uses a top-down 2D system where Y increases going north. CSS 3D has Y going up and Z going toward the viewer. That's why the renderer uses `translate3d(x, -z, -y)`: its custom properties are in DOOM coordinates while the transform needs CSS coordinates.

Once the scene is built, a JavaScript game loop tracks the game state: player position, input, collisions, and enemy AI. It recreates the original game's logic in JavaScript.

There is a strict separation between the game loop in JavaScript and the rendering in CSS. JavaScript sets a limited number of CSS custom properties such as `--player-x`, `--player-y`, `--player-z` and `--player-angle` which determine the player's location in the scene.

CSS moves the entire world in the opposite direction of the player, since CSS doesn't have a camera:

```css
#scene {
    translate: 0 0 var(--perspective);
    transform:
        rotateY(calc(var(--player-angle) * -1rad))
        translate3d(
            calc(var(--player-x) * -1px),
            calc(var(--player-z) * 1px),
            calc(var(--player-y) * 1px)
        );
}
```

Moving and looking around is just updating four custom properties.


## Input

The game supports mouse, keyboard, touch, and gamepad input, all funnelled through a unified input system that combines contributions from each source every frame.

Mouse-look uses the **Pointer Lock API**, which activates automatically when entering fullscreen using the ICON top right (not F11). Raw `movementX` deltas from `mousemove` are multiplied by a sensitivity constant and accumulated as a `turnDelta` each frame. The game loop adds that delta directly to `--player-angle`, bypassing the rate-based turning used by keyboard and gamepad so the camera tracks the mouse exactly.

Keyboard arrow keys and the gamepad's right stick apply a turn rate scaled by `deltaTime`, giving a constant angular speed regardless of frame rate. Touch controls use a drag gesture on the right half of the screen with half the mouse sensitivity. Left click (or right trigger on gamepad) fires the weapon.


## CSS features used

### 3D transforms and CSS trig functions

The entire scene is built with `transform-style: preserve-3d`. Wall dimensions use `hypot()`, wall angles use `atan2()`, and the spectator follow camera uses `sin()` and `cos()`: all computed by the browser's CSS engine from raw DOOM coordinates.

### Animating custom properties with `@property`

`@property` lets the renderer animate and transition CSS custom properties. Sector lighting uses a `--light` property that inherits down to all elements in a sector and can be animated for flickering effects. The `--player-z` property is registered as a number to enable smooth falling transitions when the player walks off a ledge.

### Irregular shapes with `clip-path`

DOOM's floors and ceilings can be any polygon. The renderer uses `clip-path` with `polygon()` to clip rectangular divs into the correct shape. For sectors with holes (pillars, platforms), it uses `shape()` with the `evenodd` fill rule, which allows percentage-based coordinates and multiple subpaths in a single clip path.

### Sprite animation with `steps()`

DOOM sprites are combined into spritesheets with frames side by side. CSS animates `background-position` across the frames using `steps()` for discrete frame changes. Each enemy has sprites for 5 viewing angles: rotations 6 through 8 are mirrors of rotations 2 through 4, handled with `scaleX(-1)`.

### Projectiles with CSS animations

Projectiles use separate `translate` and `rotate` properties. The position is animated from `--start-x/y/z` to `--end-x/y/z` via a CSS `@keyframes` animation, while `rotate` stays reactive to `--player-angle` so the sprite keeps facing the camera. When a collision is detected, JavaScript removes the element mid-flight and spawns an explosion: a three-frame spritesheet that self-destructs on `animationend`.

### Doors and lifts with CSS `transition`

Doors group their face walls and ceiling surfaces into a container `<div>`. Opening a door is just setting `data-state="open"`, which triggers a CSS transition on the container's `translateY`:

```css
.door > .panel {
    transform: translateY(0px);
    transition: transform 1s ease-in-out;
}

.door[data-state="open"] > .panel {
    transform: translateY(var(--offset));
}
```

No JavaScript animation loop needed. The CSS renderer handles the visual animation while the game loop independently tracks the door state for collision detection.

### Anchored positioning for the weapon

The HUD status bar wraps over multiple rows on narrow screens using `flex-wrap`. The weapon sprite anchors itself to the top edge of the status bar using CSS anchor positioning, so it follows regardless of how tall the status bar gets.

### Lighting with `filter: brightness()`

DOOM stores a light level per sector. The renderer sets it as a `--light` custom property on a sector container element and everything inside inherits it. Flickering lights are keyframe animations on `--light`, made possible by `@property`.

### Billboarding sprites with `rotateY()`

Enemies, decorations, barrels, and pickups always face the camera via `rotateY(calc(var(--player-angle) * 1rad))`. Because `--player-angle` is inherited from the viewport, all sprites rotate in sync as the player looks around.

### Spectator mode with separate transform properties

Spectator mode offers a top-down map view and a third-person follow camera. By using separate `translate`, `rotate`, and `transform` CSS properties instead of a single combined `transform`, transitions between first-person and spectator views are smooth: the camera tilts, rises, and repositions independently rather than arcing through 3D space.

The follow camera position is computed entirely in CSS using `sin()` and `cos()` to place the camera behind the player at a configurable distance.

### Special effects using SVG filters

The Spectre's invisibility effect uses an SVG filter applied via CSS: `feColorMatrix` creates a black silhouette, `feTurbulence` generates procedural noise, and `feDisplacementMap` distorts the pixels.


## Performance

The browser's compositor has to handle thousands of 3D-transformed elements. Large maps can overwhelm it; Safari on iOS will crash if the workload is too large. It is impressive that a browser can render this at all, given that it was not built for this workload.

The renderer culls elements outside the player's view. Its default approach is JavaScript-based: every few frames it checks each element's position and hides it if it is behind the player, too far away, or outside the frustum. Sky culling hides geometry that DOOM's sky walls should occlude. The original engine rendered this as a 2D hack that does not translate directly to a true 3D scene.

There is also an experimental pure-CSS culling implementation that uses a type grinding hack: a paused animation with a computed negative delay that converts a numeric 0/1 into a `visibility` keyword. When CSS `if()` gets wider support, this can be replaced with a clean conditional.


## Known bugs

### View Transitions flatten `preserve-3d`

View Transitions cannot be used with CSS 3D scenes in Safari. During a view transition the browser captures the old and new states as 2D snapshots, which flattens `preserve-3d` and causes the entire scene to appear flat for the duration of the transition.

### Chromium compositor instability

On Chromium-based browsers (Chrome, Edge) textures can disappear during gameplay under certain circumstances. The exact cause is unknown, but it appears to be a compositor limitation when handling a large number of 3D-transformed surfaces simultaneously. Safari and Firefox do not exhibit this issue.

### `background-image` cannot use CSS custom properties

Setting `background-image` via a CSS custom property (e.g. `background-image: var(--texture-image)`) causes severe compositor issues in both Safari and Chrome. The browser re-resolves all `var()` references on every element every frame, triggering massive re-rasterization of thousands of textures. The workaround is to set `background-image` directly as an inline style.

### `@starting-style` re-triggers continuously in Safari

A `@starting-style` transition of `opacity` combined with `display: none` on 3D-positioned elements triggers the transition continuously in Safari, rather than firing once on entry. The workaround is to drive these transitions from JavaScript with inline styles.


## Credits

- Game design, code, and samples by [id Software](https://github.com/id-Software/DOOM) (1993)
- CSS 3D rendering and JavaScript reimplementation by Niels Leenheer
