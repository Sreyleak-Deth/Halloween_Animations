# Halloween CSS Animations

This page is two files and no JavaScript.

- `index.html` is the structure: the words, the ghost, the pumpkin, the spider, the house, the potion, and the switch.
- `styles.css` is the behavior: colors, motion, hover, and day versus night.

The browser reads the HTML first, then applies the CSS. A checkbox named `#spooky` is the only piece of state. Checked means spooky night. Unchecked means daylight.

## How a page load runs

1. The browser reads `index.html` from top to bottom.
2. In `<head>`, it loads the Creepster and Nunito fonts, then `styles.css`.
3. It builds the header, the six scenes, and the footer.
4. The checkbox starts checked, because the tag is `<input id="spooky" type="checkbox" checked>`.
5. CSS sees that checked box and paints the night theme before you do anything.
6. Animations that say `infinite` start looping immediately: the ghost floats, the candle flickers, the bats fly, the bubbles rise.

Nothing in this project stores data or calls a server. Hover and the switch only change CSS.

## Shared logic

### Colors live in variables

`:root` sets the daylight palette: `--ink`, `--muted`, `--card`, `--line`, `--accent`, `--shadow`, `--sil`, and `--ghost`.

`body:has(#spooky:checked)` replaces those variables with the night palette. Any rule that uses `var(--ink)` or `var(--card)` updates on its own when the checkbox changes. `transition` on `body`, `.card`, and `.scene` makes that change fade instead of popping.

### One checkbox drives the whole theme

The switch is a `<label class="switch">` wrapped around the checkbox. Clicking the visible track clicks the label, and the label toggles the checkbox.

From there, two kinds of selectors react:

| Selector | What it matches | What it does |
| --- | --- | --- |
| `#spooky:checked + .track` | The track that sits directly after the checked box | Slides the knob and swaps Trick / Treat |
| `body:has(#spooky:checked)` | The page, whenever that box is checked | Night colors, moon, stars, dark copy |
| `body:not(:has(#spooky:checked))` | The page, whenever the box is not checked | Hides night-only words |

`.day-only` and `.night-only` are the words "Daylight", "Spooky night", "Light mode · day", and "Dark mode · spooky night". One pair is `display: none` at a time, so only the matching sentence is on screen.

### Layout

`.wrap` centers the text column at a max width of 1080px. `.grid.three` is three cards. `.grid.two` is two cards. Under 900px wide, both grids become one column. The haunted house is a full-width `.scene` between those grids.

## 01 Floating ghost

HTML, inside `.ghost-stage`:

1. `.ghost` is the white body.
2. `.eyes span` draws two black eyes. A tiny `::after` on each eye is the white highlight.
3. `.smile` is a curved border, not a filled shape.
4. `.scallops` is five spans in a row. Each span is round on the bottom, so the gaps between them look like a wavy hem.

CSS then moves it:

1. `@keyframes float` runs forever. At 0% and 100% the ghost is in place. At 50% it is 20px higher and tilted 2 degrees. The browser fills in the frames between those points.
2. `animation: float 2.8s ease-in-out infinite` attaches that loop to `.ghost`.
3. `transition: opacity 0.45s` tells the browser to fade opacity instead of snapping it.
4. `.ghost-stage:hover .ghost` sets `opacity: 0`. The stage is the hover target, not the ghost itself, so the ghost can fade out and the pointer still has something to hover.
5. `:focus-within` does the same fade for keyboard focus.
6. When the pointer leaves, the hover rule no longer matches, opacity returns to 1, and the transition fades the ghost back in. The float animation never stopped.

## 02 Jack-o'-lantern

HTML, inside `.pumpkin`:

1. `.aura` is a circle behind the pumpkin. It does not look like a face. It only glows.
2. `.lobes span` is five orange circles. They overlap. The middle one is tallest and sits on top (`z-index: 2`).
3. `.stem` is the green stalk. `::after` on the stem is the leaf.
4. `.carve` marks every lit cutout. Extra classes pick the shape: `.eye.left`, `.eye.right`, `.nose`, `.mouth`.

CSS then lights it:

1. Each lobe is a `radial-gradient` circle plus an inset `box-shadow`, so the orange reads as a round, shaded pumpkin.
2. Eyes and nose use `clip-path: polygon(...)`. The polygon is a triangle. Anything outside the triangle is not drawn.
3. The mouth uses a longer polygon. The dips along the top edge are the teeth. The bottom edge stays open, so it reads as a grin.
4. `@keyframes aura` changes the aura's `box-shadow` and opacity on an uneven rhythm, like a candle, not a smooth pulse.
5. `@keyframes candle` does the same idea on the cutouts with `brightness` and `drop-shadow`. `drop-shadow` follows the clipped triangle. A plain `box-shadow` would stay rectangular.
6. `.eye.right`, `.nose`, and `.mouth` each have a different `animation-delay`, so the eyes, nose, and mouth do not blink at the same instant.

## 03 Hanging spider

HTML, inside `.spider-stage`:

1. `.web` is an empty element. The web is only a background.
2. `.silk` is the thread. It starts short.
3. `.hanger` is a zero-size pivot at the bottom of the thread.
4. `.spider` hangs from that pivot. Eight `.leg` spans and `.spider-body` make the spider. `::before` on the body is the head. `::after` is the pair of red eyes.

CSS then draws and drops it:

1. The web background stacks two gradients. `repeating-radial-gradient` draws the rings. `repeating-conic-gradient` draws the spokes. A `mask-image` radial gradient fades the square into a corner quarter-circle.
2. `.silk` is 34px tall at rest, so the spider sits in the web.
3. `.spider-stage:hover .silk` changes the height to 198px. `transition: height 0.75s` animates that growth from the top, so the thread looks like it unspools.
4. The spider is `position: absolute; top: 100%` of the thread, so it stays glued to the thread's end while the thread gets longer.
5. `.spider-stage:hover .hanger` starts `@keyframes dangle`, which rotates the hanger between -7deg and 7deg. The spider is a child of the hanger, so it swings. The thread itself does not swing.
6. Leaving the stage removes both rules. The thread eases back to 34px and the swing stops.

## 04 Haunted house

`.scene` is `position: relative` and full viewport width. Every piece inside it is `position: absolute`, so they stack like paper cutouts. `overflow: hidden` keeps bats and fog inside the sky.

Paint order, from back to front:

1. The scene background is the sky.
2. `.stars` are two spans. Each span's `box-shadow` list is a field of extra dots, so two elements look like many stars. `@keyframes twinkle` fades them.
3. `.moon-glow` is a soft circle behind the orb.
4. `.orb` holds `.rays`, `.disc`, and three `.crater` spans.
5. `.house` is built from borders and rectangles, not an image. `.roof` and `.tower-roof` are zero-size boxes with thick borders. Transparent left and right borders plus a solid bottom border make the triangles. `.walls` and `.tower` are black rectangles in `--sil`.
6. `.window` elements sit on the walls. `@keyframes flicker` drops opacity on purpose at uneven steps. `::before` draws the cross bars with two linear gradients.
7. `.door::before` is the thin glowing gap under the door and uses the same flicker.
8. `.smoke` spans rise from the chimney with `@keyframes smoke`: they move up, grow, and fade.
9. `.tomb` stones use `::before` and `::after` for the cross.
10. `.ground.back` and `.ground.front` are wide ellipses. The front hill covers the bottom of the house so the walls look planted in the ground.
11. `.fog` drifts sideways with `@keyframes drift`.
12. Each `.bat-wrap` runs `@keyframes fly`, which moves it from off the left edge to off the right edge. The inner `.bat` only scales, so wing flapping does not fight the flight transform. `.wing.left` and `.wing.right` run `flap-left` and `flap-right`. `.b1`, `.b2`, and `.b3` use different `top`, `animation-duration`, and negative `animation-delay`, so the three bats are spread out when the page opens.

Day and night only recolor this scene. The same house stays. Checked night shows stars, craters, a pale moon, dark ground, and purple fog. Unchecked day hides stars and craters, turns `.rays` on, turns `.disc` gold, and turns the ground green. Rays spin with `@keyframes spin` and are masked into a ring so they sit around the sun instead of covering it.

## 05 Bubbling potion

HTML, inside `.flask`:

1. `.cork`, `.neck`, and `.glass` stack into a bottle. The neck overlaps the glass with a negative margin so the join has one outline.
2. `.liquid` fills the bottom 70% of the glass. `overflow: hidden` on `.glass` clips the bubbles when they leave the bottle.
3. Seven `.bubble` spans live in the liquid.
4. `.shine` is a white streak on the glass, above the liquid.
5. `.brew` is the word "Brewing" plus three dot spans.

CSS then stirs it:

1. The liquid background is one vertical gradient: purple at the top, green at the bottom.
2. `.liquid::before` is the surface ellipse. `@keyframes slosh` slides it left and right.
3. Each bubble is a small circle. `@keyframes rise` moves it up about 125px and fades it out. `@keyframes rise-left` does the same path toward the left.
4. `.bubble:nth-child(even)` uses `rise-left`. The others use `rise`.
5. Each nth-child also sets its own `left`, size, `animation-delay`, and `animation-duration`. The delays are the whole trick: the bubbles do not start together, so the bottle looks like it is boiling instead of blinking.
6. The three dots in "Brewing" use `@keyframes blink` with delays of 0s, 0.2s, and 0.4s, so they appear one after another.

## 06 Trick or treat switch

This is the same checkbox as the theme. The visible control and the page theme are not two systems.

HTML order inside the label matters:

1. `<input id="spooky">` comes first.
2. `<span class="track">` comes immediately after it, so the `+` sibling selector can see it.
3. Inside the track: the words Trick and Treat, then `.knob`.
4. The knob holds `.mini-pumpkin` and `.mini-ghost` in the same grid cell, so they sit on top of each other.

What happens when you click:

1. The label toggles `#spooky`.
2. If it becomes checked, `#spooky:checked + .track .knob` sets `left: calc(100% - 67px)`. The knob's `transition` on `left` slides it to the right. The ghost fades in and the pumpkin fades out.
3. `body:has(#spooky:checked) .day-side` hides the word Treat. The night rule shows Trick.
4. The same `:has()` rule repaints the page background, cards, pill, title glow, sky, sun or moon, ground, tombs, fog, and the line under the switch.
5. If the click clears the check, every one of those rules stops matching. The knob slides left, the pumpkin comes back, Treat replaces Trick, and the scene becomes day.
6. `:focus-visible` on the input draws an outline on the track, so keyboard users can see the switch. The input itself is `opacity: 0` and stretched over the label, but the label is what you see.

The knob icons are CSS, not emoji. The pumpkin is a circle, a stem from `::before`, and two triangle eyes from `::after`. The ghost is a rounded box, two eyes, and a scalloped bottom from `::after`.

## Motion and small screens

`@media (hover: hover)` lifts a card only on devices that actually hover, so a tap does not leave a card stuck in the raised state.

`@media (max-width: 720px)` shortens the sky and scales the house down from its base so the silhouette stays on screen.

`@media (max-width: 420px)` narrows the switch to 200px.

`@media (prefers-reduced-motion: reduce)` turns off the infinite animations and the theme transitions. Bats are given fixed `left` and `top` so they stay visible instead of sitting off screen. Bubbles are pinned in the liquid.

## What to touch if you change a scene

| You want to change | Open |
| --- | --- |
| Words on the page | `index.html` |
| Ghost bob speed | `@keyframes float` and the `animation` line on `.ghost` |
| How fast the ghost fades | `transition` on `.ghost` |
| Candle timing | `@keyframes aura` and `@keyframes candle` |
| How far the spider falls | `.spider-stage:hover .silk` height |
| Bat path | `@keyframes fly` |
| Bubble timing | `.bubble:nth-child(...)` delay and duration |
| What night looks like | `body:has(#spooky:checked)` |
| What day looks like | the base rules that are not inside `:has(#spooky:checked)` |
