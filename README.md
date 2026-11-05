# website-thank-you-banner

Prototype of the "Thanks for downloading Visual Studio Code!" banner for the VS Code download thank-you page.

- Light grid: a grid of small codicons behind the text; a band of light spreads once from the center, then the grid rests
- Text animation: the headline fades up on load, then the link; dismissing fades the text out and folds the banner closed
- Codicon close button in the top-right corner, following the dismiss button pattern on code.visualstudio.com
- Light and dark themes; the page follows your OS setting
- Preview controls at the bottom: Motion or Reduced motion on the left, Replay in the middle, light or dark theme on the right. Press H to hide them for screenshots

See it at https://hellodragonb8-pixel.github.io/website-thank-you-banner/, or open `index.html` in a browser. Everything is in that one file. Add `#dark` or `#light` to the URL to force a theme, or `?motion=reduced` to see the banner as people with reduced motion do.

To build this on code.visualstudio.com, see [HANDOFF.md](HANDOFF.md).
