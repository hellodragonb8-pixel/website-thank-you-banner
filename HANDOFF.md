# Thank-you page banner: design handoff

For: [microsoft/vscode-website#3000](https://github.com/microsoft/vscode-website/issues/3000), "Thank you page experiment"

## Goal

People who land on `/thank-you` often start a second download, because the redirect can arrive before the first download begins and the current note is easy to miss. This change replaces the small confirmation bar at the top of the page with a large banner that says the download is under way.

**In scope:** the confirmation banner only.
**Out of scope:** the header **Download** button, the "What's new" and "Get started" sections, and the download logic itself.

Reference prototype: [hellodragonb8-pixel/website-thank-you-banner](https://github.com/hellodragonb8-pixel/website-thank-you-banner) (`index.html`). 

## Where it goes

| File | Change |
|---|---|
| `server/views/thankyou.handlebars` | Replace the inner markup of `#confirmation`. Keep `id="confirmation"`, `role="status"` and `aria-live="polite"`. |
| `client/styles/includes/_thankyou.scss` | Replace the `.thankyou-confirmation` styles. |
| `client/scripts/jquery/download.js` | It already shows `#confirmation` with `.show()` and sets the `href` of `#direct-link`. Keep those IDs. Start the light grid right after `.show()` (see [Motion: light grid](#motion-light-grid)). Add the dismiss handler here or in a small script for the page. |

`download.js` keeps the banner hidden until it calls `.show()`. CSS animations don't run while an element is `display: none`, so the headline animation starts when the banner appears. That's the intended timing. The light grid is drawn by script, so it has to be started explicitly at the same moment.

## Content

- **Headline**, always on two lines:
  `Thanks for downloading` / `Visual Studio Code!`
- **Body:** `Click here if it doesn’t start soon.` Only "here" is the link (`#direct-link`). Use a typographic apostrophe in "doesn’t".
- **Dismiss button** in the top-right corner (see [Dismiss button](#dismiss-button)).

## Layout and visuals

The banner fills the width of `.thankyou-container`. In the design it is 1211 × 225 px at a 1392 px window.

| Property | Value |
|---|---|
| Min height | 225px |
| Padding | 38px 56px 21px |
| Corner radius | 24px |
| Text alignment | Centered |
| Background | White, with a #E2EFFF field blurred over it. The field extends past the edges, so the banner fades slightly lighter toward its border. |
| Grain | Faint neutral noise over the background, 14% opacity, overlay blend |
| Headline | SF Pro Display Semibold (600), 52px, 60px line height, letter spacing 0, #000 |
| Body | SF Pro Regular (400), 14px, 20px line height, 26px below the headline, #000 |
| Link | #0B77FF, underlined, 1px underline 2px below the text |

**Small screens (640px wide or less):** padding 48px 24px 28px. Headline 34px with a 40px line height.

**Edges:** keep the edges as designed. The light fade at the border comes from blurring the #E2EFFF field, not from a border or a gradient. In the reference render, the edge is about #E8F2FE and reaches flat #E2EFFF about 60px in.

### Dark theme

Layout, type and motion are the same as in the light theme. Only the colors change, along with how far the blurred field extends past the banner.

| Property | Light | Dark |
|---|---|---|
| Base under the field | #FFFFFF | #0D1014 (the page background) |
| Field | #E2EFFF, extends 28px past each edge | #081626, extends 6px past each edge |
| Grain | 14% opacity, overlay blend | Same |
| Headline and body | #000000 | #ECF4FE |
| Link | #0B77FF | #4DAAFC |
| Dismiss icon | #424242 | #C5C5C5 |
| Dismiss hover | rgba(0, 0, 0, .06) | rgba(255, 255, 255, .1) |
| Dismiss focus outline | #005A9E | #007ACC |

The dark field is #081626 because the grain lifts it a little. The rendered interior is about #091729, which matches the design. The smaller extension makes the edge fade into the page: it renders at about #0A1522 and reaches the flat interior about 40px in. The hover and focus values are the site's `--icon-button-hover-background` and `--focus-border` in each theme.

### Reference CSS (from the prototype)

Adapt this to the site's SCSS conventions.

```scss
.thankyou-confirmation {
  position: relative;
  isolation: isolate;
  overflow: hidden;
  min-height: 225px;
  padding: 38px 56px 21px;
  border-radius: 24px;
  background: #fff;
  color: #000;
  text-align: center;

  // Blurred #E2EFFF field; it extends past the box so the edges fade lighter
  &::before {
    content: "";
    position: absolute;
    inset: -28px;
    z-index: -2;
    background: #e2efff;
    filter: blur(36px);
  }

  // Faint grain
  &::after {
    content: "";
    position: absolute;
    inset: 0;
    z-index: -1;
    pointer-events: none;
    opacity: .14;
    mix-blend-mode: overlay;
    background-image: url("data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='160' height='160'><filter id='n'><feTurbulence type='fractalNoise' baseFrequency='.85' numOctaves='2' stitchTiles='stitch'/><feColorMatrix values='0 0 0 1 0  0 0 0 1 0  0 0 0 1 0  0 0 0 0 1'/></filter><rect width='100%' height='100%' filter='url(%23n)'/></svg>");
  }

  h1 {
    font-family: "SF Pro Display", -apple-system, BlinkMacSystemFont, var(--font-display);
    font-size: 52px;
    line-height: 60px;
    font-weight: 600;
    letter-spacing: 0;
    margin: 0;
    color: #000;
  }

  p {
    font-family: "SF Pro Text", "SF Pro", -apple-system, BlinkMacSystemFont, var(--font);
    font-size: 14px;
    line-height: 20px;
    font-weight: 400;
    margin: 26px 0 0;
    color: #000;

    a {
      color: #0b77ff;
      text-decoration: underline;
      text-underline-offset: 2px;
      text-decoration-thickness: 1px;
    }
  }

  @media (max-width: 640px) {
    padding: 48px 24px 28px;

    h1 {
      font-size: 34px;
      line-height: 40px;
    }
  }
}

// Dark theme overrides
.thankyou-confirmation {
  background: #0d1014;

  &::before {
    inset: -6px;
    background: #081626;
  }

  h1,
  p {
    color: #ecf4fe;
  }

  p a {
    color: #4daafc;
  }

  .dismiss-btn {
    color: #c5c5c5;
  }
}
```

## Motion: text fade-up on arrival

When the banner appears, the headline fades up into place, then the body text follows. The animation plays once and doesn't loop.

| Element | Animation | Duration | Delay | Easing |
|---|---|---|---|---|
| Headline (both lines together) | Fades from 0 to 1 opacity while moving up 8px | 0.7s | 100ms | `cubic-bezier(.2, .7, .1, 1)` |
| Body text | Same as the headline | 0.7s | 350ms | Same as the headline |

When the user has asked for reduced motion (`prefers-reduced-motion: reduce`), turn all of these animations off and show the banner in its final state.

```html
<h1>
  <span class="ln">Thanks for downloading</span>
  <span class="ln">Visual Studio Code!</span>
</h1>
```

```scss
.thankyou-confirmation {
  h1 .ln { display: block; }
  h1, p { animation: thankyou-fade-up .7s cubic-bezier(.2, .7, .1, 1) both; }
  h1 { animation-delay: .1s; }
  p { animation-delay: .35s; }

  @media (prefers-reduced-motion: reduce) {
    h1, p { animation: none; }
  }
}
@keyframes thankyou-fade-up { from { opacity: 0; transform: translateY(8px); } }
```

The page has no `<h1>` today: its headings start at "What's new" (`<h2>`), and the layout's header, updates banner and footer have none. So the banner headline is the page's `<h1>`.

## Motion: light grid

Behind the text, the banner holds a dense grid of small codicons. When the banner appears, a band of light starts at the center and spreads out to the edges as a diamond with 45° sides. Cells light up as the front passes, stay lit for a moment, then fade back. Afterwards the grid rests as faint dots, and the animation stops. It plays once and doesn't loop. The text's fade-up plays at the same time, in front of it.

The reference implementation is the script in `index.html`: `LIGHT_GRID` (the tuning), `GRID_CODICONS` (the icons), `launchLightGrid()` and `playLightGrid()`. Port it as it is; the values below describe what it does.

### Timeline (1211 × 225 banner)

| Time | What happens |
|---|---|
| 0s | The grid appears as resting dots and fades in over 1s (ease-out) |
| About 0.7s | The light first shows at the center, behind the headline |
| About 2.6s | The light front reaches the corners |
| About 5.6s | The last lit cells have faded back to rest |
| 6.3–7.8s | The resting dots fade down to 40% of their brightness, then drawing stops |

### Grid and light

| Property | Value |
|---|---|
| Layer | A `<canvas>` filling the banner, `aria-hidden="true"`, `pointer-events: none`. It sits above the blurred field and the grain, and below the headline, body and dismiss button. |
| Grid | 9px icons with 6px gaps (15px pitch), centered in the banner |
| Resting cells | `circle-small-filled` dots |
| Lit cells | 15 codicons from microsoft/vscode-codicons: `dash`, `add`, `code`, `json`, `terminal`, `git-branch`, `source-control`, `sparkle`, `search`, `files`, `extensions`, `symbol-event`, `copilot`, `vscode`, `circle-filled`. Brighter cells use icons with more ink. |
| Brightness | 16 steps, for an LED look. Every lit cell is about equally bright, with a brighter crest at the leading edge. |
| Leading edge | Icons scramble right at the front, about 18 changes per second, then settle |
| Light color | #1F7BFF (dark: #0F66F0), drawn at partial opacity so the banner's own background shows through |
| Shape | Each cell lights when the front passes its distance from the center, measured as the horizontal distance plus the vertical distance. That makes the diamond. |
| Speed | The front moves about 0.25 × (banner width + height) per second, about 366px/s at 1211 × 225, so wider banners take the same time to fill |
| Band | Cells stay lit for 2.7s after the front passes, then fade back to rest |
| Resolution | The canvas uses `devicePixelRatio`, capped at 2 |

### Behavior

- **Start:** call `playLightGrid()` right after `download.js` shows the banner. The grid measures the banner when it starts, so it can't start while the banner is hidden.
- **Reduced motion:** with `prefers-reduced-motion: reduce`, don't draw the grid at all. The banner shows in its final state, as for the text fade-up.
- **Dismiss:** the grid stops drawing as soon as the banner is hidden, whether by the `hidden` attribute or `display: none` (such as jQuery's `.hide()`).
- **Cost:** about 1,000 cells per frame for about 8 seconds, from icons drawn ahead of time into an atlas, then nothing.

```scss
.thankyou-confirmation {
  .light-grid {
    position: absolute;
    inset: 0;
    width: 100%;
    height: 100%;
    pointer-events: none;
    animation: thankyou-grid-in 1s ease-out both;
  }
  h1, p { position: relative; z-index: 1; }
  .dismiss-btn { z-index: 2; }
}
@keyframes thankyou-grid-in { from { opacity: 0; } }
```

## Dismiss button

The button follows the site's existing `.dismiss-btn` pattern, the one on the updates banner, but uses the codicon close icon instead of the "×" glyph.

| Property | Value |
|---|---|
| Position | Absolute, 16px from the top and 16px from the right of the banner |
| Size | 32 × 32px, 4px corner radius |
| Icon | `codicon codicon-close`, 16px, color #424242 (dark: #C5C5C5) |
| Background | Transparent; on hover `var(--icon-button-hover-background)`, which is rgba(0, 0, 0, .06) in the light theme |
| Keyboard focus | `outline: 2px solid var(--focus-border); outline-offset: 2px` on `:focus-visible` |
| Label | `title="Dismiss"`, plus visually hidden text "Dismiss" (`.sr-only`) |

```html
<button type="button" class="dismiss-btn" title="Dismiss" id="thankyou-dismiss">
  <span class="sr-only">Dismiss</span>
  <i class="codicon codicon-close" aria-hidden="true"></i>
</button>
```

**Behavior:**
- Clicking the button animates the banner out, then hides it:

  | Step | Animation | Duration | Delay | Easing |
  |---|---|---|---|---|
  | 1. Headline and body | Fade from 1 to 0 opacity while moving up 8px | 200ms | 0 | `ease-in` |
  | 2. Banner | Height, min height, top margin, top and bottom padding and opacity all go to 0, so the content below moves up into the space | 360ms | 200ms | `cubic-bezier(.4, 0, .2, 1)` |
  | 3. Banner | Hidden (`display: none`) | | 560ms | |

  The light grid keeps playing while the banner folds, and stops once it's hidden.
- With reduced motion turned on, the button hides the banner straight away, with no animation, as the site's updates banner does.
- Clicking again while the banner is leaving does nothing.
- After the banner hides, move keyboard focus to the "What's new" heading (`#whats-new-heading`, given `tabindex="-1"`), so keyboard and screen reader users aren't left on a removed element.
- Reference code: the `.leaving` styles and `dismissAnimated()` in `index.html`:

  ```scss
  .thankyou-confirmation.leaving {
    h1, p { animation: thankyou-fade-out .2s ease-in forwards; }
  }
  @keyframes thankyou-fade-out { to { opacity: 0; transform: translateY(-8px); } }
  ```

  ```js
  async function dismissAnimated(el) {
    if (el.classList.contains('leaving')) return;
    el.classList.add('leaving');
    await new Promise(r => setTimeout(r, 200));
    const cs = getComputedStyle(el);
    const from = { height: `${el.offsetHeight}px`, minHeight: cs.minHeight, marginTop: cs.marginTop,
      paddingTop: cs.paddingTop, paddingBottom: cs.paddingBottom, opacity: 1 };
    const to = { height: '0px', minHeight: '0px', marginTop: '0px', paddingTop: '0px', paddingBottom: '0px', opacity: 0 };
    await el.animate([from, to], { duration: 360, easing: 'cubic-bezier(.4, 0, .2, 1)' }).finished;
    el.hidden = true;   // or $(el).hide()
  }
  ```
- The dismissal isn't saved. The banner shows again on the next visit to `/thank-you`. The site's updates banner does save its dismissal in `localStorage`, but that would be wrong here, because each visit follows a new download.

## Acceptance criteria

- [ ] After a download redirect, the banner appears where the old confirmation bar was, and the headline and body fade up once.
- [ ] As the banner appears, the light grid spreads once from the center as a diamond, settles to faint dots, and stops drawing after about 8 seconds.
- [ ] The text, link and dismiss button stay in front of the grid, and the link and button can be clicked while it plays.
- [ ] "here" links to the same direct-download URL that `#direct-link` gets today, for every OS and build.
- [ ] At a 1392px window, the banner matches the reference: 225px tall, 24px corners, centered type at the sizes above, and the soft lighter edge.
- [ ] In the dark theme, the banner uses the dark colors above, and its edge fades into the page background.
- [ ] At 640px wide or less, the headline is 34/40 and nothing is clipped or overlapping.
- [ ] With reduced motion turned on, the banner appears in its final state without animation, and without the light grid.
- [ ] Dismissing fades the text out, then folds the banner closed as the page moves up, in about 0.56s. With reduced motion, the banner hides straight away.
- [ ] The dismiss button can be reached with Tab, shows the focus outline, and works with Enter and Space. Screen readers announce it as "Dismiss". After dismissing, focus moves to "What's new".
- [ ] The banner headline is the page's only `<h1>`.
- [ ] The header **Download** button and the rest of the page are unchanged.
