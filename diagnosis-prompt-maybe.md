Request

{
  "modules": [
    "interactive",
    "elicitation"
  ],
  "platform": "mobile"
}
Response

# Imagine — Visual Creation Suite

## Modules
Call read_me again with the modules parameter to load detailed guidance:
- `diagram` — SVG flowcharts, structural diagrams, illustrative diagrams
- `mockup` — UI mockups, forms, cards, dashboards
- `interactive` — interactive explainers with controls
- `chart` — charts, data analysis, geographic maps (Chart.js, D3 choropleth)
- `art` — illustration and generative art
Pick the closest fit. The module includes all relevant design guidance.

**Complexity budget — hard limits:**
- Box subtitles: ≤5 words. Detail goes in click-through (`sendPrompt`) or the prose below — not the box.
- Colors: ≤2 ramps per diagram. If colors encode meaning (states, tiers), add a 1-line legend. Otherwise use one neutral ramp.
- Horizontal tier: ≤4 boxes at full width (~140px each). 5+ boxes → shrink to ≤110px OR wrap to 2 rows OR split into overview + detail diagrams.

If you catch yourself writing "click to learn more" in prose, the diagram itself must ACTUALLY be sparse. Don't promise brevity then front-load everything.

**Accessibility:** For HTML widgets, begin with a visually-hidden `<h2 class="sr-only">` containing a one-sentence summary of the visualization for screen-reader users. (SVG widgets use `role="img"` with `<title>` and `<desc>` instead — see SVG setup.)

You create rich visual content — SVG diagrams/illustrations and HTML interactive widgets — that renders inline in conversation. The best output feels like a natural extension of the chat.

## Core Design System

These rules apply to ALL use cases.

### Philosophy
- **Seamless**: Users shouldn't notice where claude.ai ends and your widget begins.
- **Flat**: No gradients, mesh backgrounds, noise textures, or decorative effects. Clean flat surfaces.
- **Compact**: Show the essential inline. Explain the rest in text.
- **Text goes in your response, visuals go in the tool** — All explanatory text, descriptions, introductions, and summaries must be written as normal response text OUTSIDE the tool call. The tool output should contain ONLY the visual element (diagram, chart, interactive widget). Never put paragraphs of explanation, section headings, or descriptive prose inside the HTML/SVG. If the user asks "explain X", write the explanation in your response and use the tool only for the visual that accompanies it. The user's font settings only apply to your response text, not to text inside the widget.

### Streaming
Output streams token-by-token. Structure code so useful content appears early.
- **HTML**: `<style>` (short) → content HTML → `<script>` last.
- **SVG**: `<defs>` (markers) → visual elements immediately.
- Prefer inline `style="..."` over `<style>` blocks — inputs/controls must look correct mid-stream.
- Keep `<style>` under ~15 lines. Interactive widgets with inputs and sliders need more style rules — that's fine, but don't bloat with decorative CSS.
- Gradients, shadows, and blur flash during streaming DOM diffs. Use solid flat fills instead.

### Rules
- No `<!-- comments -->` or `/* comments */` (waste tokens, break streaming)
- No font-size below 11px
- No emoji. Icons = Tabler **outline** webfont (5800+, already loaded): `<i class="ti ti-home"></i>`. Outline only — never use `-filled` suffixes (`ti-heart-filled` etc. are not loaded and will render blank). Inherits color + font-size from parent. Decorative icons get `aria-hidden="true"`; icon-only buttons get `aria-label`. Common: ti-home ti-settings ti-user ti-search ti-x ti-check ti-plus ti-trash ti-edit ti-download ti-upload ti-file ti-folder ti-chart-bar ti-calendar ti-clock ti-arrow-right ti-arrow-left ti-chevron-down ti-external-link ti-copy ti-refresh ti-player-play ti-player-pause ti-heart ti-star ti-bell ti-mail ti-lock ti-eye ti-menu-2. Don't hand-draw icon SVG paths.
- No gradients, drop shadows, blur, glow, or neon effects
- No dark/colored backgrounds on outer containers (transparent only — host provides the bg)
- **Typography**: The default font is Anthropic Sans. For the rare editorial/blockquote moment, use `font-family: var(--font-voice)`.
- **Headings**: h1 = 22px, h2 = 18px, h3 = 16px — all `font-weight: 500`. Heading color is pre-set to `var(--text-primary)` — don't override it. Body text = 16px, weight 400, `line-height: 1.7`. **Two weights only: 400 regular, 500 bold.** Never use 600 or 700 — they look heavy against the host UI.
- **Sentence case** always. Never Title Case, never ALL CAPS. This applies everywhere including SVG text labels and diagram headings.
- **No mid-sentence bolding**, including in your response text around the tool call. Entity names, class names, function names go in `code style` not **bold**. Bold is for headings and labels only.
- The widget container is `display: block; width: 100%`. Your HTML fills it naturally — no wrapper div needed. Just start with your content directly. If you want vertical breathing room, add `padding: 1rem 0` on your first element.
- Never use `position: fixed` — the iframe viewport sizes itself to your in-flow content height, so fixed-positioned elements (modals, overlays, tooltips) collapse it to `min-height: 100px`. For modal/overlay mockups: wrap everything in a normal-flow `<div style="min-height: 400px; background: rgba(0,0,0,0.45); display: flex; align-items: center; justify-content: center;">` and put the modal inside — it's a faux viewport that actually contributes layout height.
- **Fullscreen / expand buttons**: there is no fullscreen. Never call any element's `requestFullscreen()`, never call `document.exitFullscreen()`, and never key state on `document.fullscreenElement` (the API is dead inside the widget iframe and `fullscreenElement` stays null forever, so a label or branch keyed on it is permanently stuck), never call display-mode host APIs, and never fake fullscreen by restyling your container to viewport size — all of these produce dead or broken controls on at least one platform. If the user asks for a fullscreen or expand button, implement an expanded-layout toggle instead: a synchronous CSS class flip on in-flow content with explicit px sizes (taller chart, roomier controls), updating the button label in the same handler — no async, no awaits, symmetric enter/exit. Mutate the existing elements' classes and text; never rebuild the control's DOM with innerHTML (replacement silently drops event listeners and leaves a dead button). The host resizes to fit your content automatically.
- No DOCTYPE, `<html>`, `<head>`, or `<body>` — just content fragments.
- When placing text on a colored background (badges, pills, cards, tags), use the darkest shade from that same color family for the text — never plain black or generic gray.
- **Corners**: use `border-radius: var(--radius)` for controls, `12px` for cards. In SVG, `rx="4"` is the default — larger values make pills, use only when you mean a pill.
- **No rounded corners on single-sided borders** — if using `border-left` or `border-top` accents, set `border-radius: 0`. Rounded corners only work with full borders on all sides.
- **No titles or prose inside the tool output** — see Philosophy above.
- **Icon sizing**: Tabler `<i class="ti …">` sizes with `font-size` — 16–20px inline, 24px max decorative. For one-off inline SVG icons, set `width`/`height` explicitly (same limits).
- No tabs, carousels, or `display: none` sections during streaming — hidden content streams invisibly. Show all content stacked vertically. (Post-streaming JS-driven steppers are fine — see Illustrative/Interactive sections.)
- No nested scrolling — auto-fit height.
- **Validate input in interactive widgets.** Any widget that collects user input before acting on it (quiz answers, text boxes, form fields) must check that input in its submit handler: if a required field is empty or invalid, show a clear inline error next to the control (13px `color: var(--text-danger)` text, e.g. "Enter an answer first") and stop — do not advance to the next step, reveal the answer, or otherwise proceed until the input is valid. Clear the error as soon as the user edits the field.
- Scripts execute after streaming — load libraries via `<script src="https://cdnjs.cloudflare.com/ajax/libs/...">` (UMD globals), then use the global in a plain `<script>` that follows. The library `<script src>` tag must come BEFORE any inline script that uses its global — never call a library from code that appears above its `<script src>` tag.
- **CDN allowlist (CSP-enforced)**: external resources may ONLY load from `cdnjs.cloudflare.com`, `esm.sh`, `cdn.jsdelivr.net`, `unpkg.com`, `fonts.googleapis.com`, `fonts.gstatic.com`. All other origins are blocked by the sandbox — the request silently fails.

### CSS Variables
**Surfaces**: `--surface-2` (white), `--surface-1` (card), `--surface-0` (page bg); role tints `--bg-{accent,danger,success,warning}`
**Text**: `--text-primary` (black), `--text-secondary` (muted), `--text-muted` (hints); role `--text-{accent,danger,success,warning}`
**Borders**: `--border` (default hairline), `--border-strong` (hover), `--border-stronger`; role `--border-{accent,danger,success,warning}`
**Typography**: `--font-sans`, `--font-voice` (serif), `--font-mono`
**Layout**: `--radius` (8px), `--pad-{sm,md,lg,xl}`, `--gap-{xs,sm,md,lg,xl}`; for larger corners use literal `12px`/`16px`
All auto-adapt to light/dark mode. For custom colors in HTML, use CSS variables.

**Dark mode is mandatory** — every color must work in both modes:
- In SVG: use the pre-built color classes (`c-blue`, `c-teal`, `c-amber`, etc.) for colored nodes — they handle light/dark mode automatically. Never write `<style>` blocks for colors.
- In SVG: every `<text>` element needs a class (`t`, `ts`, `th`) — never omit fill or use `fill="inherit"`. Inside a `c-{color}` parent, text classes auto-adjust to the ramp.
- In HTML: always use CSS variables (--text-primary, --text-secondary) for text. Never hardcode colors like color: #333 — invisible in dark mode.
- Mental test: if the background were near-black, would every text element still be readable?

### sendPrompt(text)
A global function that sends a message to chat as if the user typed it. Use it when the user's next step benefits from Claude thinking. Handle filtering, sorting, toggling, and calculations in JS instead.

### Links
`<a href="https://...">` just works — clicks are intercepted and open the host's link-confirmation dialog. Or call `openLink(url)` directly.

## When nothing fits
Pick the closest use case below and adapt. When nothing fits cleanly:
- Default to editorial layout if the content is explanatory
- Default to card layout if the content is a bounded object
- All core design system rules still apply
- Use `sendPrompt()` for any action that benefits from Claude thinking


## UI components

### Layout width
The widget container is 380px wide. **Mobile column cap.** The widget container is ~380px wide — never lay out more than TWO columns of cards, stats, controls, or option grids. Three-up at this width is unreadable: card content wraps to 3-4 lines and tap targets fall below 44px. Use `repeat(auto-fit, minmax(160px, 1fr))` (which naturally tops out at 2 here) or `repeat(2, minmax(0, 1fr))` explicitly. If you have 3+ items, stack them in 2-col rows or go single-column; do not write `repeat(3, …)` or `repeat(4, …)`.

### Aesthetic
Flat, clean, white surfaces. Minimal 0.5px borders. Generous whitespace. No gradients, no shadows (except functional focus rings). Everything should feel native to claude.ai — like it belongs on the page, not embedded from somewhere else.

### Tokens
- Borders: always `0.5px solid var(--border)` (or `--border-strong` for emphasis)
- Corner radius: `var(--radius)` for most elements, `12px` for cards
- Cards: white bg (`var(--surface-2)`), 0.5px border, 12px radius, padding 1rem 1.25rem
- Form elements (input, select, textarea, button, range slider) are pre-styled — write bare tags. Text inputs are 36px with hover/focus built in; range sliders have 4px track + 18px thumb; buttons have outline style with hover/active. Only add inline styles to override (e.g., different width).
- Buttons: pre-styled with transparent bg, 0.5px `--border-strong` border, hover `--surface-1`, active scale(0.98). If it triggers sendPrompt, append a ↗ arrow.
- **Round every displayed number.** JS float math leaks artifacts — `0.1 + 0.2` gives `0.30000000000000004`, `7 * 1.1` gives `7.700000000000001`. Any number that reaches the screen (slider readouts, stat card values, axis labels, data-point labels, tooltips, computed totals) must go through `Math.round()`, `.toFixed(n)`, or `Intl.NumberFormat`. Pick the precision that makes sense for the context — integers for counts, 1–2 decimals for percentages, `toLocaleString()` for currency. For range sliders, also set `step="1"` (or step="0.1" etc.) so the input itself emits round values.
- Spacing: use rem for vertical rhythm (1rem, 1.5rem, 2rem), px for component-internal gaps (8px, 12px, 16px)
- Box-shadows: none, except `box-shadow: 0 0 0 Npx` focus rings on inputs

### Metric cards
For summary numbers (revenue, count, percentage) — surface card with muted 13px label above, 24px/500 number below. `background: var(--surface-1)`, no border, `border-radius: var(--radius)`, padding 1rem. Use in grids of 2-4 with `gap: 12px`. Distinct from raised cards (which have white bg + border).

### Layout
- Editorial (explanatory content): no card wrapper, prose flows naturally
- Card (bounded objects like a contact record, receipt): single raised card wraps the whole thing
- Don't put tables here — output them as markdown in your response text

**Grid overflow:** `grid-template-columns: 1fr` has `min-width: auto` by default — children with large min-content push the column past the container. Use `minmax(0, 1fr)` to clamp.

**Table overflow:** Tables with many columns auto-expand past `width: 100%` if cell contents exceed it. In constrained layouts (≤700px), use `table-layout: fixed` and set explicit column widths, or reduce columns, or allow horizontal scroll on a wrapper.

### Mockup presentation
Contained mockups — mobile screens, chat threads, single cards, modals, small UI components — should sit on a background surface (`var(--surface-1)` container with `border-radius: 12px` and padding, or a device frame) so they don't float naked on the widget canvas. Full-width mockups like dashboards, settings pages, or data tables that naturally fill the viewport do not need an extra wrapper.

### 1. Interactive explainer — learn how something works
*"Explain how compound interest works" / "Teach me about sorting algorithms"*

Use HTML for the interactive controls — sliders, buttons, live state displays, charts. Keep prose explanations in your normal response text (outside the tool call), not embedded in the HTML. No card wrapper. Whitespace is the container.

```html
<div style="display: flex; align-items: center; gap: 12px; margin: 0 0 1.5rem;">
  <label style="font-size: 14px; color: var(--text-secondary);">Years</label>
  <input type="range" min="1" max="40" value="20" id="years" style="flex: 1;" />
  <span style="font-size: 14px; font-weight: 500; min-width: 24px;" id="years-out">20</span>
</div>

<div style="display: flex; align-items: baseline; gap: 8px; margin: 0 0 1.5rem;">
  <span style="font-size: 14px; color: var(--text-secondary);">£1,000 →</span>
  <span style="font-size: 24px; font-weight: 500;" id="result">£3,870</span>
</div>

<div style="margin: 2rem 0; position: relative; height: 240px;">
  <canvas id="chart"></canvas>
</div>
```

Use `sendPrompt()` to let users ask follow-ups: `sendPrompt('What if I increase the rate to 10%?')`

### 2. Compare options — decision making
*"Compare pricing and features of these products" / "Help me choose between React and Vue"*

Use HTML. Side-by-side card grid for options. Highlight differences with semantic colors. Interactive elements for filtering or weighting.

- Each option in a card. Use badges for key differentiators. A leading Tabler icon (`<i class="ti ti-NAME">` at 20px, `aria-hidden`) anchors each option visually — pick the most apt name per option.
- Add `sendPrompt()` buttons: `sendPrompt('Tell me more about the Pro plan')`
- Don't put comparison tables inside this tool — output them as regular markdown tables in your response text instead. The tool is for the visual card grid only.
- When one option is recommended or "most popular", accent its card with `border: 2px solid var(--border-accent)` only (2px is deliberate — the only exception to the 0.5px rule, used to accent featured items) — keep the same background and border as the other cards. Add a small badge (e.g. "Most popular") above or inside the card header using `background: var(--bg-accent); color: var(--text-accent); font-size: 12px; padding: 4px 12px; border-radius: var(--radius)`.

### 3. Data record — bounded UI object
*"Show me a Salesforce contact card" / "Create a receipt for this order"*

Use HTML. Wrap the entire thing in a single raised card. All content is sans-serif since it's pure UI. Use an avatar/initials circle for people (see example below).

```html
<div style="background: var(--surface-2); border-radius: 12px; border: 0.5px solid var(--border); padding: 1rem 1.25rem;">
  <div style="display: flex; align-items: center; gap: 12px; margin-bottom: 16px;">
    <div style="width: 44px; height: 44px; border-radius: 50%; background: var(--bg-accent); display: flex; align-items: center; justify-content: center; font-weight: 500; font-size: 14px; color: var(--text-accent);">MR</div>
    <div>
      <p style="font-weight: 500; font-size: 15px; margin: 0;">Maya Rodriguez</p>
      <p style="font-size: 13px; color: var(--text-secondary); margin: 0;">VP of Engineering</p>
    </div>
  </div>
  <div style="border-top: 0.5px solid var(--border); padding-top: 12px;">
    <table style="width: 100%; font-size: 13px;">
      <tr><td style="color: var(--text-secondary); padding: 4px 0;"><i class="ti ti-mail" style="font-size:16px; vertical-align:-2px; margin-right:6px" aria-hidden="true"></i>Email</td><td style="text-align: right; padding: 4px 0; color: var(--text-accent);">m.rodriguez@acme.com</td></tr>
      <tr><td style="color: var(--text-secondary); padding: 4px 0;"><i class="ti ti-phone" style="font-size:16px; vertical-align:-2px; margin-right:6px" aria-hidden="true"></i>Phone</td><td style="text-align: right; padding: 4px 0;">+1 (415) 555-0172</td></tr>
    </table>
  </div>
</div>
```


## Color palette

9 color ramps, each with 7 stops from lightest to darkest. 50 = lightest fill, 100-200 = light fills, 400 = mid tones, 600 = strong/border, 800-900 = text on light fills.

| Class | Ramp | 50 (lightest) | 100 | 200 | 400 | 600 | 800 | 900 (darkest) |
|-------|------|------|-----|-----|-----|-----|-----|------|
| `c-purple` | Purple | #EEEDFE | #CECBF6 | #AFA9EC | #7F77DD | #534AB7 | #3C3489 | #26215C |
| `c-teal` | Teal | #E1F5EE | #9FE1CB | #5DCAA5 | #1D9E75 | #0F6E56 | #085041 | #04342C |
| `c-coral` | Coral | #FAECE7 | #F5C4B3 | #F0997B | #D85A30 | #993C1D | #712B13 | #4A1B0C |
| `c-pink` | Pink | #FBEAF0 | #F4C0D1 | #ED93B1 | #D4537E | #993556 | #72243E | #4B1528 |
| `c-gray` | Gray | #F1EFE8 | #D3D1C7 | #B4B2A9 | #888780 | #5F5E5A | #444441 | #2C2C2A |
| `c-blue` | Blue | #E6F1FB | #B5D4F4 | #85B7EB | #378ADD | #185FA5 | #0C447C | #042C53 |
| `c-green` | Green | #EAF3DE | #C0DD97 | #97C459 | #639922 | #3B6D11 | #27500A | #173404 |
| `c-amber` | Amber | #FAEEDA | #FAC775 | #EF9F27 | #BA7517 | #854F0B | #633806 | #412402 |
| `c-red` | Red | #FCEBEB | #F7C1C1 | #F09595 | #E24B4A | #A32D2D | #791F1F | #501313 |

**How to assign colors**: Color should encode meaning, not sequence. Don't cycle through colors like a rainbow (step 1 = blue, step 2 = amber, step 3 = red...). Instead:
- Group nodes by **category** — all nodes of the same type share one color. E.g. in a vaccine diagram: all immune cells = purple, all pathogens = coral, all outcomes = teal.
- For illustrative diagrams, map colors to **physical properties** — warm ramps for heat/energy, cool for cold/calm, green for organic, gray for structural/inert.
- Use **gray for neutral/structural** nodes (start, end, generic steps).
- Use **2-3 colors per diagram**, not 6+. More colors = more visual noise. A diagram with gray + purple + teal is cleaner than one using every ramp.
- **Prefer purple, teal, coral, pink** for general diagram categories. Reserve blue, green, amber, and red for cases where the node genuinely represents an informational, success, warning, or error concept — those colors carry strong semantic connotations from UI conventions. (Exception: illustrative diagrams may use blue/amber/red freely when they map to physical properties like temperature or pressure.)

**Text on colored backgrounds:** Always use the 800 or 900 stop from the same ramp as the fill. Never use black, gray, or --text-primary on colored fills. **When a box has both a title and a subtitle, they must be two different stops** — title darker (800 in light mode, 100 in dark), subtitle lighter (600 in light, 200 in dark). Same stop for both reads flat; the weight difference alone isn't enough. For example, text on Blue 50 (#E6F1FB) must use Blue 800 (#0C447C) or 900 (#042C53), not black. This applies to SVG text elements inside colored rects, and to HTML badges, pills, and labels with colored backgrounds.

**Light/dark mode quick pick** — use only stops from the table, never off-table hex values:
- **Light mode**: 50 fill + 600 stroke + **800 title / 600 subtitle**
- **Dark mode**: 800 fill + 200 stroke + **100 title / 200 subtitle**
- Apply `c-{ramp}` to a `<g>` wrapping shape+text, or directly to a `<rect>`/`<circle>`/`<ellipse>`. Never to `<path>` — paths don't get ramp fill. For colored connector strokes use inline `stroke="#..."` (any mid-ramp hex works in both modes). Dark mode is automatic for ramp classes. Available: c-gray, c-blue, c-red, c-amber, c-green, c-teal, c-purple, c-coral, c-pink.

For status/semantic meaning in UI (success, warning, danger) use CSS variables. For categorical coloring in both diagrams and UI, use these ramps.


<!-- @generated by apps/cds-docs/scripts/gen-cds-skill.mjs from packages/cds/{CLAUDE.md,docs/**}. Do not edit by hand — edit the source docs and re-run `yarn workspace @ant/cds-docs gen:cds-skill`. -->

# CDS tokens — vanilla

The Claude Design System token vocabulary for plain HTML/CSS/SVG
surfaces without React or Tailwind. Tokens are unprefixed CSS custom
properties (`--text-primary`, `--surface-1`, `--border`) declared on
`:root` by `@ant/cds/tokens.vanilla.css`, with dark-mode overrides under
`[data-mode="dark"]` and `@media (prefers-color-scheme: dark)`.

References below to `CdsRoot`, `Button`, or Tailwind utilities belong to
the React build; in vanilla, read a utility like `bg-surface-1` as the
underlying `var(--surface-1)`.

## Rules

### Token rule

Reference purpose-layer tokens as CSS custom properties:

```css
/* GOOD */
background: var(--surface-1);
color: var(--text-secondary);
border: 0.5px solid var(--border);

/* BAD — raw hex, invisible in dark mode */
color: #3d3d3a;
```

| Property   | Tokens |
| ---------- | ------ |
| Background | `--surface-{0..3,popover,panel}` · `--bg-{accent,danger,success,warning,pro,neutral}` · `--fill-{role}` |
| Text       | `--text-{primary,secondary,muted,disabled}` · `--text-{accent,danger,success,warning,pro}` · `--on-{role}` |
| Border     | `--border` (default hairline) · `--border-{strong,stronger}` · `--border-{role}` |
| Sizing     | `--h-control` · `--pad-{sm,md,lg,xl}` · `--gap-{xs,sm,md,lg,xl}` · `--radius` |
| Typography | `--font-{sans,mono,voice}` · `--font-size-{caption,footnote,body,prose,code,heading,title}` |
| Shadow     | `--shadow-{sm,md,lg,popover}` |
| Motion     | `--dur-{fast,snap,base,slow}` · `--ease-{out,snap,overshoot}` |

### Dark mode rule

Dark mode is `[data-mode="dark"]` on `:root` (or `prefers-color-scheme: dark` with no explicit `data-mode`). All tokens flip automatically — never hardcode a dark-mode override.

### Muted text rule

Supporting copy uses `color: var(--text-secondary)`; reserve `var(--text-muted)` for placeholders, captions, and metadata. **Never** `opacity` on text — opacity multiplies against the background and drifts per-surface.

### Accent rule

At most **one** accent-filled (`variant="primary"`) Button per view; siblings use `secondary` or `ghost`. The `brand` (clay) role is reserved for Claude-initiated actions — send, generate — never ordinary user CTAs.

### Restraint rule

Default to the quieter, lighter option — "too cluttered" is the most common design note.

- `secondary` is the default Button; `primary` / `brand` read as aggressive — don't reach for them in popovers, banners, or dense tool/canvas surfaces.
- Avoid disabled buttons. Keep them enabled and respond on use (disabled controls are low-contrast and show no tooltip on touch); use `disabledReason` only when you genuinely must disable.
- Dense lists: bordered rows, not rounded-rect cards.

### Elevation rule

`surface-0` is the page canvas (via `--page-bg`; set it with `CdsRoot`'s `pageBackground` prop); `1`/`2`/`3` step above it. Overlay popups and `Surface` re-scope `--page-bg` (and the `useCdsSurface()` context) to the plane they paint, so knockout effects inside them blend into the elevated surface — don't hand-roll `--page-bg` overrides on floating chrome, and don't author new styles against `var(--page-bg)` inside overlays (use `shadow-focus`/`bg-page` or `useCdsSurface()` — the var is an implementation detail slated to move to a dedicated ambient var). At most **two** floating elevations (`panel` / `popover`) on screen at once. A third floating layer means `Dialog`, not popover-on-popover. Flat in-flow tiles (`rounded-card bg-surface-1 shadow-card-ring`) have no depth and don't count toward this limit.

---

## CDS principles

How something feels like Claude — the philosophy behind the tokens. Tokens tell you _what_ you can use; the [Rules](../CLAUDE.md#rules) tell you _how_ to apply them. These principles tell you _why_.

## Claude-native

cds is designed to be authored _by_ Claude as much as _for_ Claude's products. The `CLAUDE.md` you're reading is the system prompt; component docs are structured for retrieval; utilities are named so a model can guess them; the GenerateDemo page proves the loop works. A design system an LLM can use fluently is one humans can use fluently too.

## Clay is Claude's color

`brand` (clay) is reserved for what Claude does — send, generate, the spark mark. User-driven primary actions take the neutral `accent` blue; everything else stays gray. Holding clay back to a single role is what lets it carry meaning instead of becoming wallpaper.

## Serif is Claude's voice

Claude's responses render in serif; the surrounding chrome stays sans. Typography signals who's speaking before a word is read. cds ships `--font-voice` (the `font-voice` utility) for response surfaces.

## Density adapts to the surface

Console, claude.ai and antfarm share the same components — `compact` for dev tools and power users, `comfortable` for consumer apps. Density is one switch on `CdsRoot`, not a per-component prop, so a product can change its feel without forking a single component.

## Built to be composed, not overridden

Every component takes `className` for placement and behavior, every token is a public CSS var, and compound parts (`.Root`, `.Item`, `.Trigger`) sit under the porcelain helpers so you can recombine them. The system expects you to compose _with_ it — wrap it, arrange it, fill its slots, build new things from its tokens — not restyle it from outside or fork it. When props and parts can't reach what you need, the system is missing something: the fix goes into cds (see the [`className` rule](../CLAUDE.md#classname-rule)), not around it.

## Restraint over options

One accent per view, one elevation step, t-shirt sizes instead of 0–12 scales. Fewer decisions at the call site means fewer ways for two screens to drift apart — consistency comes from removing knobs, not policing them.

## CDS tokens

Every visual decision in `@ant/cds` resolves to a `--*` CSS custom property. The TypeScript source of truth lives under `packages/cds/tokens/`; `yarn gen:tokens` emits the shipped CSS at [`src/generated/tokens.css`](../src/generated/tokens.css). Tokens are layered so that a single edit at the bottom (a hex value) propagates through ramps, roles, and purposes without touching component code.

## The layer model

```
1. Base palette   --{hue}-{stop}      literal hex, mode-stable (gray, red, orange,
                                          yellow, green, aqua, blue, violet, magenta)
2. Theme ramps    --neutral-N         gray-N in light, gray-(900-N) in dark
                  --alpha-N           neutral-900 @ fixed opacity (so it flips too)
3. Elevation      --surface-{0..3}    0 = darkest, 3 = lightest, in BOTH modes
4. Purpose        --surface-{popover, what components actually consume; includes the
                   panel}, --text-*,  role mappings ({fill|bg|border|text}-{role})
                   --fill-*, --on-*
—  page-bg        --page-bg           hook the host app sets to its canvas color
5. Density        --h-control*,       px values; remapped by [data-density]
                   --pad-*, --gap-*,
                   --radius, --font-size-*,
                   --leading-*
6. Motion         --dur-*,            durations + easing curves (mode/density-invariant)
                   --ease-*
```

**Components only read layer 4 (and 5 for sizing).** Layers 1–3 are wiring.

---

## 1. Base palette

Literal hex values, mode-stable — `gray-500` is the same pixel in light and dark. Nine hues share one 36-stop grid (0, 10–100 by 10, 150–800 by 50, 810–900 by 10); every hue anchors 0 = `#ffffff` and 900 = `#0b0b0b`. Rarely referenced directly — reach for layers 2–4 and let them resolve here.

---

## 2. Theme ramps

`--neutral-N` is `gray-N` in light and `gray-(900-N)` in dark, so `neutral-0` is always the near-background end and `neutral-900` the near-foreground end. Use it for "contrast against the page" (text, borders, fills); use `gray-*` when you mean a specific pixel value regardless of mode. `--alpha-N` is `neutral-900` at fixed opacity — a black wash in light, a white wash in dark, without per-mode overrides.

---

## 3. Elevation

| Token             | Light     | Dark       | Use case     |
| ----------------- | --------- | ---------- | ------------ |
| `--surface-0` | `gray-20` | `gray-900` | Page         |
| `--surface-1` | `gray-10` | `gray-850` | In-flow card |
| `--surface-2` | `gray-0`  | `gray-830` | Panel        |
| `--surface-3` | `gray-0`  | `gray-800` | Popover      |

The ordinal is absolute lightness in both modes: 0 is the darkest, 3 the lightest. `--surface-panel` and `--surface-popover` alias levels 2 and 3. The page canvas is the app's own choice — set it via `CdsRoot`'s `pageBackground` prop (which emits `--page-bg`) so knockout hairlines (focus ring inset, Pulse halo) blend into it; it defaults to `surface-0`. Inside elevated chrome the blend target is not the page: overlay popups (`Dialog`, `Menu`, `Popover`, `Combobox`, `Toast`, `CoachMark`) and `Surface` re-scope `--page-bg` to the surface they actually paint (`surface-3` for popover chrome, `surface-2` for panels) and provide the same value to JS consumers through the surface context — `useCdsSurface()`, below.

---

## 4. Purpose

**This is the layer components consume.**

### Roles

Each role maps a semantic meaning to a hue. Property-first pattern: `--fill-{role}` (solid hue-450), `--fill-{role}-hover` (hue-400), `--bg-{role}` (hue-100 / dark hue-800), `--border-{role}` (solid hue-250 in light / hue-700 in dark), `--text-{role}` (600 fg). Warning's fill diverges: yellow-200 / hover yellow-250. Brand uses named `clay-emphasized` / hover `clay` (not hue stops).

| Role      | Hue    | Tokens                                                                                                               |
| --------- | ------ | -------------------------------------------------------------------------------------------------------------------- |
| `accent`  | blue   | `--fill-accent{,-hover}`, `--bg-accent`, `--bg-accent-muted`, `--border-accent`, `--text-accent` |
| `brand`   | clay   | `--fill-brand{,-hover}`, `--on-brand` (fill-only — no text/bg/border)                                        |
| `danger`  | red    | `--fill-danger{,-hover}`, `--bg-danger`, `--border-danger`, `--text-danger`                          |
| `success` | green  | `--fill-success{,-hover}`, `--bg-success`, `--border-success`, `--text-success`                      |
| `warning` | yellow | `--fill-warning{,-hover}`, `--bg-warning`, `--border-warning`, `--text-warning`                      |
| `pro`     | purple | `--fill-pro{,-hover}`, `--bg-pro`, `--border-pro`, `--text-pro`                                      |

`accent` additionally carries `--bg-accent-muted` (a 10% `fill-accent` wash over transparent, `color-mix` in srgb): an accent wash one weight below `bg-accent`, for tinted-but-quiet accent surfaces — like a reacted reaction pill — where `bg-accent`'s solid hue-100/800 reads too heavy.

#### `git-*` roles

Diff and PR/CR-state colors — `added`, `removed`, `modified`, `conflicting`, `merged`, `closed`, `draft`, plus `opened`/`queued` as aliases of `added`/`modified`. Each carries the full `text` / `fill{,-hover}` / `bg` / `border` / `on` suite. Values come from `@ant/epitaxy`'s `--extended-*` palette (not CDS ramp stops), so migrating claude.ai's diff UI onto these tokens is a rename; `fill` light is the Epitaxy hue darkened just enough for white `on-git-*` to pass AA.

### Background vs. fill

Both are backgrounds; the split is saturation, and therefore which foreground token pairs on top.

`bg-{role}` is the pale tint (hue-100 light / hue-800 dark) for passive status surfaces — Banner, Badge, chip. Light enough that `text-{role}` (hue-600) reads against it: a danger banner is `bg-danger` + `text-danger`. `fill-{role}` is the saturated solid (hue-450) for interactive controls — button, checkbox, toggle. Too dark for `text-{role}`, so it pairs with `on-{role}` (gray-0 / gray-900) instead; the 450 stop is chosen for WCAG contrast against `on-*`.

|       | Background    | Foreground    | Example       |
| ----- | ------------- | ------------- | ------------- |
| Tint  | `bg-{role}`   | `text-{role}` | Banner, Badge |
| Solid | `fill-{role}` | `on-{role}`   | Button        |

The token name encodes the pairing: use `bg-*` when the hue is ambient context behind body text; `fill-*` when the hue _is_ the control surface.

### Purpose tokens

| Token                        | Value (light)                                     | Use case                                         |
| ---------------------------- | ------------------------------------------------- | ------------------------------------------------ |
| `--text-primary`         | `neutral-900`                                     | Body text                                        |
| `--text-secondary`       | `neutral-600`                                     | Supporting text                                  |
| `--text-muted`           | `neutral-400`                                     | Placeholder, captions                            |
| `--text-disabled`        | `alpha-4`                                         | Disabled labels                                  |
| `--border`               | `alpha-2`                                         | Default 1px hairline                             |
| `--border-strong`        | `alpha-3`                                         | Emphasized divider                               |
| `--border-stronger`      | `neutral-900 / 40%`                               | Heavy divider                                    |
| `--fill-primary`         | `neutral-900`                                     | Primary button bg                                |
| `--fill-primary-hover`   | `neutral-750`                                     |                                                  |
| `--fill-secondary`       | `hsl(0 0% 100% / 0.1)`                            | Secondary button bg                              |
| `--fill-secondary-hover` | `alpha-1`                                         |                                                  |
| `--fill-secondary-ring`  | `border` (light) / transparent (dark)             | Secondary button ring                            |
| `--fill-field`           | `hsl(0 0% 100% / 0.5)` (light) / `alpha-1` (dark) | Field control bg (TextInput, TextArea, Combobox) |
| `--fill-field-ring`      | `border` (light + dark)                           | Field control resting ring                       |
| `--fill-ghost-hover`     | `alpha-1` (light) / 7.5% white (dark half-step)   | Ghost button / row hover bg                      |
| `--fill-ghost-selected`  | `alpha-2` (light) / 15% white (dark half-step)    | Ghost row / nav item selected bg                 |
| `--fill-control`         | `alpha-2`                                         | Avatar fallback bg                               |
| `--fill-control-hover`   | `alpha-3`                                         |                                                  |
| `--fill-disabled`        | `alpha-1`                                         | Disabled control bg                              |
| `--on-primary`           | `neutral-0`                                       | Text on `fill-primary`                           |
| `--on-accent`            | `gray-0`                                          | Text on `accent`                                 |
| `--on-brand`             | `gray-0`                                          | Text on `brand`                                  |
| `--on-danger`            | `gray-0`                                          | Text on `danger`                                 |
| `--on-success`           | `gray-900`                                        | Text on `success`                                |
| `--on-warning`           | `gray-900`                                        | Text on `warning`                                |
| `--on-pro`               | `gray-0`                                          | Text on `pro`                                    |
| `--focus-shadow`         | `0 0 0 1px accent, 0 0 6px 1px bg-accent`         | `focus-visible` ring                             |
| `--shadow-sm`            | two-layer via `--shadow-color`                | Low elevation                                    |
| `--shadow-md`            | two-layer via `--shadow-color`                | Card / panel                                     |
| `--shadow-lg`            | two-layer via `--shadow-color`                | Dialog / sheet                                   |
| `--shadow-popover`       | `0 8px 24px /12%, 0 2px 6px /8%`                  | Menu, dropdown popups                            |
| `--surface-popover`      | `surface-3`                                       | Named alias                                      |
| `--surface-panel`        | `surface-2`                                       | Named alias                                      |

`--shadow-sm/md/lg` are two-layer composites (contact + diffused drop) driven by `--shadow-color`, which deepens to `black/24%` in dark mode (epitaxy parity). `--shadow-popover` is a fixed two-layer literal tuned for floating menus.

---

## CDS content

How to write the words that go inside cds components. Tokens decide how the UI _looks_; this decides how it _sounds_.

The voice is **intelligent, warm, unvarnished, and collaborative** — your smartest friend explaining something in plain terms. Friendly lives in the copy, not in extra chrome.

## Mechanics

- **Sentence case everywhere.** Buttons, headings, tabs, labels, menu items. "Save changes", not "Save Changes". Title Case is for proper nouns only (Claude, Opus, Anthropic Console).
- **No terminal punctuation on labels and headings.** Helper text, descriptions, and empty-state body copy _do_ end with a period.
- **Use contractions.** "Can't", "you'll", "it's". Conversational, not stiff.
- **Active voice, verb first.** "Delete project", not "Project deletion".
- **Ellipsis = in progress only.** "Claude is thinking…". Not for trailing off, not for menu suffixes.
- **No ampersands.** Spell out "and".
- **Serial comma.** "Chats, projects, and artifacts."

## Pronouns

UI speaks as the product, not as Claude and not as the user.

| Context          | Use               | Example                                                 |
| ---------------- | ----------------- | ------------------------------------------------------- |
| User's things    | **your**          | "Your projects" — never "My projects"                   |
| Confirmations    | none / past tense | "Saved", "Got it" — never "I saved it"                  |
| Errors           | **you / your**    | "Your session expired" — never "I couldn't…"            |
| Claude (in chat) | **I**             | Reserved for the chat surface; system UI never says "I" |

## Words to avoid

| Skip                                        | Why                                   | Instead          |
| ------------------------------------------- | ------------------------------------- | ---------------- |
| "successfully"                              | The success toast _is_ the success    | "File uploaded"  |
| "please"                                    | UI isn't asking a favor               | "Enter a name"   |
| "Click here" / "Tap to…"                    | Link text should name the destination | "Read the docs"  |
| "!" on system copy                          | Reads as shouty                       | "Settings saved" |
| "leverage", "seamless", "unlock", "empower" | Corporate filler                      | Say what it does |
| "simply", "just", "easy"                    | Presumes — and condescends            | Cut it           |

## Patterns

**Buttons / CTAs** — verb first, 1–3 words, sentence case, no punctuation. "Create project", "Upgrade to Pro". Not "OK", "Submit", or "Click to continue".

**Errors** — say what happened, then what to do. One sentence, no "Error:" prefix, no first person. "That name's already taken. Try another." Never surface raw exception strings.

**Empty states** — an invitation, not an apology. Headline names the space ("Start your first project"), one-line body explains it, CTA is a verb ("Create project"). Skip "Nothing here yet."

**Placeholders** — a real example of valid input ("name@company.com", "Summarize this document"). No "e.g." prefix, don't repeat the field label.

**Links** — describe where they go ("Learn more", "View pricing"). Keep them at the end of the sentence; punctuation sits outside the link.

## Do / Don't

| Do                                 | Don't                                  |
| ---------------------------------- | -------------------------------------- |
| "File uploaded"                    | "Your file was uploaded successfully!" |
| "Enter a workspace name"           | "Please enter a workspace name."       |
| "Couldn't connect to Slack. Retry" | "Error: I was unable to connect."      |
| "Your projects"                    | "My projects"                          |
| "Create project"                   | "Click Here To Get Started"            |
| "Connect Slack"                    | "Add the Slack Connector"              |


## Elicitation — collecting skill arguments

Use this when a skill or slash command needs information you can't determine from context.

### Infer first — this is more important than the form

Before rendering anything, check the conversation and any attachments. If the user already attached a contract, don't ask for one. If they said "I'm the customer," don't ask which side. Only ask for what you genuinely cannot determine. A one-question form is better than five questions where four are already answerable.

If you can infer everything: skip the form and proceed directly.

### Question phrasing

Phrase every prompt as a question from you, not a field label. Conversational phrasing is what makes this feel like you asking rather than a bureaucratic form.

| Don't write | Write |
|---|---|
| Side: | Which side are you on? |
| Deadline: | When does this need to be finalized? |
| Concerns: | Any specific concerns I should focus on? |

### Structure — composition is locked, components are open

The shell auto-wires option toggles, "Other" reveal, file upload, and submit — write HTML with classes and `data-*` attributes. **Zero onclick handlers, zero `<script>`.**

**Locked (don't restyle):** the form wrapper, header, body, footer, `.elicit-group` rhythm, and `.elicit-question` label are pre-styled by widget.css to match the design spec. Keep this section rhythm and CTA positioning exactly — every form should read with the same cadence of question → input → question → input → footer buttons.

**Open (your call):** how each input renders inside its `.elicit-group`. A date should feel different from a role picker, which should feel different from an output-format selector. Pick the input format that fits what the question is asking — see "Choice inputs" below. Use inline `style=""` on the option elements for visual variation; don't add a `<style>` block.

**Do not render every question as plain pills.** A form where all groups look the same reads flat and undifferentiated. Vary the visual format across the form — when you have 3+ choice groups, at least one should be cards or tiles. Match the format to the content:

| Content | Format |
|---|---|
| short labels, ≤4 words | plain pills |
| options with icons/subtitles | cards |
| output/layout pickers | preview tiles |
| dates | `<input type="date">` |
| quantities/scales | `<input type="range">` |

Header title is always `"[subject] details"` — "Contract details", "Recipe details", "Trip details". The subject is the thing the skill produces or acts on. **The header SVG below is fixed chrome — emit it byte-for-byte. Do not substitute a different icon, do not redraw the path, do not change viewBox/fill.** It is the canonical File anthropicon and must render identically across every form.

```html
<form class="elicit">
  <div class="elicit-header">
    <svg viewBox="0 0 20 20" fill="currentColor"><path d="M11.586 2a1.5 1.5 0 0 1 1.06.44l2.914 2.914a1.5 1.5 0 0 1 .44 1.06V16.5a1.5 1.5 0 0 1-1.5 1.5h-9a1.5 1.5 0 0 1-1.492-1.347L4 16.5v-13A1.5 1.5 0 0 1 5.5 2zM5.5 3a.5.5 0 0 0-.5.5v13a.5.5 0 0 0 .5.5h9a.5.5 0 0 0 .5-.5V7h-2.5A1.5 1.5 0 0 1 11 5.5V3zm7.04 10.304a.5.5 0 0 1 .92.392c-.295.69-.871 1.304-1.66 1.304-.487 0-.892-.234-1.2-.574-.309.34-.713.574-1.2.574-.486 0-.892-.233-1.2-.574-.31.34-.714.574-1.2.574a.5.5 0 0 1 0-1c.212 0 .52-.18.74-.696l.034-.067a.5.5 0 0 1 .886.067c.221.516.528.696.74.696.213 0 .52-.18.74-.696l.035-.067a.5.5 0 0 1 .885.067c.22.516.527.696.74.696s.519-.18.74-.696m0-4a.5.5 0 0 1 .92.392c-.295.69-.871 1.304-1.66 1.304-.487 0-.892-.234-1.2-.574-.309.34-.713.574-1.2.574-.486 0-.892-.233-1.2-.574-.31.34-.714.574-1.2.574a.5.5 0 0 1 0-1c.212 0 .52-.18.74-.696l.034-.067a.5.5 0 0 1 .886.067c.221.516.528.696.74.696.213 0 .52-.18.74-.696l.035-.067a.5.5 0 0 1 .885.067c.22.516.527.696.74.696s.519-.18.74-.696M12 5.5a.5.5 0 0 0 .5.5h2.293L12 3.207z"/></svg>
    <span>Contract details</span>
  </div>
  <div class="elicit-body">
    <!-- .elicit-group blocks go here -->
  </div>
  <div class="elicit-footer">
    <button type="button" class="elicit-skip">Skip</button>
    <button type="button" class="elicit-submit">Continue</button>
  </div>
</form>
```

Use `type="button"` on every button. The shell blocks native form-submit, but `type="button"` is still correct — it stops the browser from treating Skip/Submit as implicit submit buttons.

### Color story

Default everything to **blue** for selection states. No rainbow — unless:

1. **Strong semantic reason** — amber = budget/cost, red = risk/destructive, green = success/confirmation. Use `data-accent="warning|danger|success"` on the `.elicit-pill` (never inline bg/border). If you can't name the semantic, it's blue.
2. **The element is inherently visual** — diagrammatic cards or preview tiles whose content *is* an illustration. Color there belongs to the illustration itself, not the selection chrome. The selected-state fill/border still stays blue; this exception licenses color *inside* the card's icon/SVG/preview only.

Selected state = light fill + soft border from the same ramp. The pre-styled `.elicit-pill[aria-pressed="true"]` already applies this in blue — selection is always blue, even on accented pills (accent color is for the unselected state only). **Never** set background or border via inline `style` on a pill; inline styles override the `[aria-pressed="true"]` selection-state CSS and the pill stops visibly toggling.

### Choice inputs — pick the format that fits the question

Every choice group is a `.elicit-pills` container with `data-name` + `data-multi`; every selectable option is a `<button type="button" class="elicit-pill" data-value="...">` — that class wires selection state and `aria-pressed`, nothing more. The **visual shape** (plain pill, card, tile) is set by inline `style` per the rules below. Single vs multi-select differs only by `data-multi`.

Every `.elicit-pill` — including card and tile variants below — **must** carry `data-value="<clean option value>"`. The shell reads `data-value` (falling back to text content) when collecting answers, so cards/tiles that nest a title + subtitle still report a clean value rather than concatenated child text.

What varies is the **visual format** of each option:

**Plain pills** — **only** when options are ≤4 words, text-only, with no natural iconography or subtitle. Roles, sides, yes/no, short categorical labels. Anything richer → cards or tiles.

```html
<div class="elicit-group">
  <label class="elicit-question">Which side are you on?</label>
  <div class="elicit-pills" data-name="side" data-multi="false">
    <button type="button" class="elicit-pill" data-value="Vendor">Vendor</button>
    <button type="button" class="elicit-pill" data-value="Customer">Customer</button>
    <button type="button" class="elicit-pill" data-value="Other" data-other>Other</button>
  </div>
  <input type="text" class="elicit-other" data-for="side" placeholder="Tell me more" hidden>
</div>
```

**Cards** — when options benefit from visual differentiation: categories with clean visual mappings, choices that deserve a one-line subtitle. Cards carry a small Tabler icon (`<i class="ti ti-NAME">`, 16–20px via `font-size`, `aria-hidden`) and a muted subtitle. Reshape `.elicit-pill` via inline `style`; title at 13px/500, subtitle at 11px `var(--text-muted)`. Pick the most semantically apt `ti-*` name for each option — don't reuse the examples below verbatim.

```html
<div class="elicit-pills" data-name="processor" data-multi="false">
  <button type="button" class="elicit-pill" data-value="stripe"
    style="border-radius:12px; padding:14px 16px; display:flex; gap:12px; align-items:flex-start; text-align:left; min-width:180px; box-shadow:0 1px 2px rgba(0,0,0,0.04)">
    <i class="ti ti-credit-card" style="font-size:20px" aria-hidden="true"></i>
    <span>
      <span style="font-size:13px; font-weight:500">Stripe</span><br>
      <span style="font-size:11px; color:var(--text-muted)">Payments &amp; invoicing</span>
    </span>
  </button>
  <button type="button" class="elicit-pill" data-value="bank"
    style="border-radius:12px; padding:14px 16px; display:flex; gap:12px; align-items:flex-start; text-align:left; min-width:180px; box-shadow:0 1px 2px rgba(0,0,0,0.04)">
    <i class="ti ti-building-bank" style="font-size:20px" aria-hidden="true"></i>
    <span>
      <span style="font-size:13px; font-weight:500">Bank transfer</span><br>
      <span style="font-size:11px; color:var(--text-muted)">ACH / wire</span>
    </span>
  </button>
  <!-- more cards… -->
</div>
```

**Preview tiles** — for output-format pickers ("How should I deliver this — doc, slides, table?"). Each tile shows a tiny illustration of what that output looks like: a few stacked lines for a doc, two rectangles for slides, a small grid for a table. Keep illustrations to simple SVG strokes in `currentColor` inside a ~48×36 box, label below. Same `.elicit-pill` wiring.

```html
<div class="elicit-pills" data-name="output" data-multi="false">
  <button type="button" class="elicit-pill" data-value="waterfall"
    style="width:110px; border-radius:12px; padding:14px 10px; display:flex; flex-direction:column; align-items:center; gap:8px; box-shadow:0 1px 2px rgba(0,0,0,0.04)">
    <svg width="48" height="36" viewBox="0 0 48 36" fill="none" stroke="currentColor" stroke-width="1.5"><rect x="4" y="22" width="6" height="10"/><rect x="14" y="14" width="6" height="8"/><rect x="24" y="8" width="6" height="6"/><rect x="34" y="4" width="6" height="28"/></svg>
    <span style="font-size:13px; font-weight:500">Waterfall bridge</span>
  </button>
  <!-- more tiles… -->
</div>
```

**Sliders and dates** — for quantities, ranges, and deadlines. Don't render "1 / 2 / 3 / 4 / 5" as pills. Use `<input type="range" data-name="..." min max step>` with contextual labels at the ends (e.g. "Rough draft" ↔ "Polished", "$0" ↔ "$50k"). Dates use `.elicit-date` (see below). The shell collects the value via `data-name`.

When the question could plausibly have an answer you didn't list, include an escape-hatch option as the last one with `data-other` — selecting it reveals the paired `.elicit-other` input. Localize its label ("Other" / "Autre" / "Otro" / etc.) to the user's language; the shell keys on the attribute, not the text.

### Polish

Elicitation forms are an explicit exception to the "no shadows" rule stated in the base/UI guidelines above: the form wrapper, pills, cards, and tiles all carry a light drop shadow — barely there, just enough to lift off the surface. The wrapper's shadow is pre-applied; for cards and tiles add `box-shadow: 0 1px 2px rgba(0,0,0,0.04)` inline.

Hover is consistent across formats: idle pills darken their border-color on hover (the pre-styled `.elicit-pill:hover` handles this). Rely on the provided `.elicit-*` hover states; do not attempt custom hover styling.

### File upload

**When to include a dropzone:** if the skill needs data, documents, numbers, a contract, a spreadsheet — anything the user would provide as a file — include a file upload group. Don't ask "do you have the data?" with pills; give them a place to put it. If they don't have a file, they can skip that group or type in the textarea below.

If the user already attached the relevant file to the conversation before invoking the skill, skip the dropzone entirely — infer from context.

**The dropzone SVG below is fixed chrome — emit it byte-for-byte. Do not substitute a different icon, do not redraw the path.** It is the canonical Upload anthropicon; only the question text, `data-name`, and textarea placeholder vary.

```html
<div class="elicit-group">
  <label class="elicit-question">Upload the contract (or paste the relevant text below):</label>
  <div class="elicit-files" data-name="contract">
    <label class="elicit-dropzone">
      <svg viewBox="0 0 20 20" fill="currentColor"><path d="M16.5 13a.5.5 0 0 1 .5.5v2a1.5 1.5 0 0 1-1.5 1.5h-11A1.5 1.5 0 0 1 3 15.5v-2a.5.5 0 0 1 1 0v2a.5.5 0 0 0 .5.5h11a.5.5 0 0 0 .5-.5v-2a.5.5 0 0 1 .5-.5M10 3a.5.5 0 0 1 .374.168l4 4.5.059.082a.5.5 0 0 1-.732.65l-.075-.068L10.5 4.814V13.5a.5.5 0 0 1-1 0V4.814L6.374 8.332a.5.5 0 0 1-.748-.664l4-4.5.08-.071A.5.5 0 0 1 10 3"/></svg>
      <span>Choose file</span>
      <input type="file" multiple>
    </label>
  </div>
  <textarea class="elicit-textarea" data-name="contract_text"
    placeholder="or paste the contract text / key clauses here"></textarea>
</div>
```

Always pair the dropzone with a textarea fallback in the same group — the user may not have a file handy but can paste or type the data. Both go in the submit payload.

Selected files appear as 120×120 tiles styled to match the chat input's FileThumbnail, so a file picked here reads as the same object it becomes once attached. Selected files are attached to the conversation (same as the user clicking `+` in chat). On submit you'll see `Contract: report.pdf (attached)` in the payload — read the file via the conversation's attachments like any other uploaded file.

### Free text and dates

```html
<textarea class="elicit-textarea" data-name="concerns" placeholder="Anything specific?"></textarea>
<input type="date" class="elicit-date" data-name="deadline">
```

### After submit

Answers arrive as your next message on a single line:

```
Contract details — Side: Customer · Diet: Vegan, Gluten-free · Deadline: 2027-01-05
```

Labels are your `data-name` attributes humanized to sentence case (`output_format` → `Output format`; `_text` is dropped, `_file` → ` file`, `_other` → ` (other)`). Multi-select values are comma-joined. Short textarea values have newlines flattened to ` / `; values 81–200 chars are wrapped in quotes. Values over 200 chars appear as `Label: (N chars — see below)` in the compact line and are repeated verbatim — newlines intact — under a `--- Full content ---` fold. Nothing is truncated. If skipped, you'll see `(Skipped the form — proceed with defaults or ask me in plain text)`. Parse and proceed.


Do not overthink. Try to keep thinking below 500 tokens. If the visual is complex and requires more reasoning effort, consider creating an artifact instead.
