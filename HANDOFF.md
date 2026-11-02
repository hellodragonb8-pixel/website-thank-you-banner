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
| `client/scripts/jquery/download.js` | No change expected. It already shows `#confirmation` with `.show()` and sets the `href` of `#direct-link`. Keep those IDs. Add the dismiss handler here or in a small script for the page. |

`download.js` keeps the banner hidden until it calls `.show()`. CSS animations don't run while an element is `display: none`, so the headline animation starts when the banner appears. That's the intended timing.

## Content

- **Headline**, always on two lines:
  `Thanks for downloading` / `Visual Studio Code!`
- **Body:** `Click here if your download hasn’t started.` Only "here" is the link (`#direct-link`). Use a typographic apostrophe in "hasn’t".
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
```

## Motion: line reveal on arrival

When the banner appears, each headline line slides up from behind its own mask. The body text then fades up as the second line lands. The animation plays once and doesn't loop.

| Element | Animation | Duration | Delay | Easing |
|---|---|---|---|---|
| Headline line 1 | Moves up from 110% below into place, clipped to the line | 1.1s | 150ms | `cubic-bezier(.19, 1, .22, 1)` |
| Headline line 2 | Same as line 1 | 1.1s | 270ms | Same as line 1 |
| Body text | Fades from 0 to 1 opacity while moving up 8px | 0.8s | 770ms | `cubic-bezier(.2, .7, .1, 1)` |

Each line's mask has 12px of extra room at the bottom (`padding-bottom: 12px; margin-bottom: -12px`), so letters that drop below the line, like "g", aren't cut off.

When the user has asked for reduced motion (`prefers-reduced-motion: reduce`), turn all of these animations off and show the banner in its final state.

```html
<h1>
  <span class="ln"><span class="li" style="--l:0">Thanks for downloading</span></span>
  <span class="ln"><span class="li" style="--l:1">Visual Studio Code!</span></span>
</h1>
```

```scss
.thankyou-confirmation {
  h1 .ln { display: block; overflow: hidden; padding-bottom: 12px; margin-bottom: -12px; }
  h1 .li {
    display: inline-block;
    vertical-align: top;
    animation: thankyou-rise 1.1s cubic-bezier(.19, 1, .22, 1) both;
    animation-delay: calc(150ms + var(--l) * 120ms);
  }
  p { animation: thankyou-fade-up .8s cubic-bezier(.2, .7, .1, 1) .77s both; }

  @media (prefers-reduced-motion: reduce) {
    h1 .li, p { animation: none; }
  }
}
@keyframes thankyou-rise { from { transform: translateY(110%); } }
@keyframes thankyou-fade-up { from { opacity: 0; transform: translateY(8px); } }
```

The page has no `<h1>` today: its headings start at "What's new" (`<h2>`), and the layout's header, updates banner and footer have none. So the banner headline is the page's `<h1>`.

## Dismiss button

The button follows the site's existing `.dismiss-btn` pattern, the one on the updates banner, but uses the codicon close icon instead of the "×" glyph.

| Property | Value |
|---|---|
| Position | Absolute, 16px from the top and 16px from the right of the banner |
| Size | 32 × 32px, 4px corner radius |
| Icon | `codicon codicon-close`, 16px, color #424242 |
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
- Clicking the button hides the banner straight away, with no closing animation. That matches the site's updates banner.
- The content below moves up into the space.
- After the banner hides, move keyboard focus to the "What's new" heading (`#whats-new-heading`, given `tabindex="-1"`), so keyboard and screen reader users aren't left on a removed element.
- The dismissal isn't saved. The banner shows again on the next visit to `/thank-you`. The site's updates banner does save its dismissal in `localStorage`, but that would be wrong here, because each visit follows a new download.

## Acceptance criteria

- [ ] After a download redirect, the banner appears where the old confirmation bar was, and the headline plays the line reveal once.
- [ ] "here" links to the same direct-download URL that `#direct-link` gets today, for every OS and build.
- [ ] At a 1392px window, the banner matches the reference: 225px tall, 24px corners, centered type at the sizes above, and the soft lighter edge.
- [ ] At 640px wide or less, the headline is 34/40 and nothing is clipped or overlapping.
- [ ] With reduced motion turned on, the banner appears in its final state without animation.
- [ ] The dismiss button can be reached with Tab, shows the focus outline, and works with Enter and Space. Screen readers announce it as "Dismiss". After dismissing, focus moves to "What's new".
- [ ] The banner headline is the page's only `<h1>`.
- [ ] The header **Download** button and the rest of the page are unchanged.
