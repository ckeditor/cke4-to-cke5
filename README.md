# cke4-to-cke5

A reference setup that demonstrates how to configure **CKEditor 5** so it looks and behaves as close as possible to **CKEditor 4**, for integrators migrating from CKE4 to CKE5.

The repository ships:

- Two stand-alone demo pages (`index-ckeditor4.html`, `index-ckeditor5.html`) loaded with the same source content.
- A side-by-side page (`index-both.html`) for visual comparison.
- A small CSS override layer (`app/ckeditor5/ckeditor5-content-overrides.css`) that bridges the visual differences between the two editors.
- A CKE5 editor configuration tuned for CKE4-shaped content (plugin choices, toolbar, GHS, heading list, etc.).

## Why this exists

CKEditor 4 reached End-of-Life. Existing content authored in CKE4 has visual expectations baked in: tables drawn with dotted helper borders, images rendered inline rather than wrapped in `<figure>`, deprecated HTML attributes still in use, and so on. Loading that content into a default CKEditor 5 install produces a different look that surprises users and breaks expectations.

This repository is the answer to *"what's the minimum I need to change in CKEditor 5 so my CKE4 content keeps looking the way authors expect?"* It is not a drop-in compatibility shim. It is a working reference that integrators copy and adapt.

## Approach: overrides, not stylesheet manipulation

There are two ways to align CKE5 output with CKE4:

1. **Modify the generated `ckeditor5-content.css`** that ships in the npm package. Centralized, but every CKE5 release regenerates the file and you have to re-apply your edits.
2. **Load CKE5 vendor stylesheets unchanged and ship a thin overrides stylesheet on top.** No vendor files touched. Standard `pnpm update` workflow.

The override approach is the one used here. It avoids the recurring "copy the file, re-apply edits, hope nothing upstream changed" cycle that comes with modifying the generated stylesheet on every CKE5 release.

## Repository layout

```
.
├── index-ckeditor4.html       CKE4 demo (vendored)
├── index-ckeditor5.html       CKE5 demo (npm)
├── index-both.html            Side-by-side iframes
├── package.json               CKE5 is a regular npm dependency
├── app/
│   └── ckeditor5/
│       └── ckeditor5-content-overrides.css   The CKE4-look stylesheet
├── assets/                    Image assets shared by both demos
├── node_modules/ckeditor5/    CKE5 from npm (not committed)
└── vendor/ckeditor4/          Vendored CKE4 (not on npm)
```

## Setup

```bash
pnpm install
pnpm run serve
```

Then open `http://localhost:8080/index-both.html` and scroll through the comparison content.

## What's been changed and why

### CKE5 editor configuration

The CKE5 setup in `index-ckeditor5.html` is *not* a default install. Several deliberate choices steer the editor toward CKE4-shaped output:

- **`ImageInline` only — no `ImageBlock`.** CKE4's image plugin produces inline `<img>` (sometimes inside a `<p>`); it does not wrap images in `<figure>`. Loading `ImageBlock` in CKE5 would re-wrap CKE4 images and change their alignment / styling rules.
- **`PlainTableOutput`.** Default CKE5 wraps tables in `<figure class="table">` for the captioned-table feature; CKE4 outputs plain `<table>`. `PlainTableOutput` keeps the output aligned with CKE4.
- **GeneralHtmlSupport + HtmlComment with permissive allow-list.** CKE4 preserves whatever attributes/classes/styles it doesn't recognize. The GHS `allow: [ { name: /.*/, attributes: true, classes: true, styles: true } ]` mirrors that.
- **Heading list including `h1`.** CKE5's default heading config starts at `h2`. CKE4 exposes `h1` in the heading dropdown, so the list is extended here.
- **`fontFamily.supportAllValues: true` and `fontSize.supportAllValues: true`.** Preserves arbitrary inline font sizes/families produced by CKE4 instead of normalizing them.
- **`list.properties.styles: true`, `startIndex: true`, `reversed: true`.** Round-trips CKE4 list attributes (custom `list-style-type`, `start`, `reversed`).
- **`table.tableProperties.alignment.useInlineStyles: false`** and `defaultProperties.alignment: 'blockLeft'`. Uses CSS classes for alignment (matching the override stylesheet) and defaults unaligned tables to the left (CKE4 behavior, not CKE5's centered default).
- **`menuBar` enabled.** CKE4 had a context menu; the CKE5 menu bar is the closest equivalent.

Read `index-ckeditor5.html` top-to-bottom for the full plugin list and toolbar layout.

### CSS overrides

`app/ckeditor5/ckeditor5-content-overrides.css` is loaded **after** `node_modules/ckeditor5/dist/browser/ckeditor5-content.css`. Each section addresses a specific visual difference:

- **Typography variables** — CKE4 used `sans-serif, Arial, …` at 13px / line-height 1.6 with `#333` text. CKE5 uses `Helvetica, Arial, …` at `medium` size with `#000` text and tighter line-height.
- **Editing-area padding** — CKE5 has no default content padding; CKE4 did.
- **Blockquote** — CKE4 used a serif blockquote with asymmetric padding.
- **Links** — CKE4 used the blue `#0782C1`; CKE5 inherits user-agent styling.
- **Lists** — CKE4's default left-padding is `40px`; CKE5 inherits user-agent values.
- **Headings** — CKE4 rendered headings with `font-weight: normal` and tight line-height; CKE5 uses user-agent defaults.
- **`<hr>`** — CKE4 rendered it as a thin line; CKE5 ships a chunky 4 px bar.
- **`<pre>`** — CKE5 wraps `<pre>` in a styled gray box; CKE4 leaves it largely unstyled.
- **Images** — CKE5 auto-centers unaligned images; CKE4 left-aligned them.
- **Tables** — the bulk of the overrides:
  - Borderless tables (`<table>` without `border`, or `border="0"`, or `style="border-width:0px"`) get CKE4-style dotted helper lines.
  - Non-zero `border` attribute → solid borders.
  - In **v48+**, CKE5 upcasts `<table border="N">` to a model attribute and re-emits as inline `style="border-width:Npx"`, dropping the `border=` attribute. An additional rule handles this style-based representation.
  - Nested-editable borders (the dotted lines inside table cells) restored to CKE4-style `#D3D3E3` dotted.

Each rule has an inline comment explaining the CKE4 behavior it restores.

## Maintaining this when you upgrade CKEditor 5

### To update

```bash
pnpm update ckeditor5
```

Or pin a specific version in `package.json` and run `pnpm install`.

That's the entire upgrade procedure for the npm package. The vendor stylesheets in `node_modules/ckeditor5/dist/browser/` change with each release, but you never edit them — the override file sits on top.

### To check the overrides are still correct

After updating, the overrides may or may not still produce the same visual output. CKE5 occasionally changes:

- How a legacy HTML attribute is upcasted/downcasted (e.g. v48 changed `border="N"` handling for tables — see the inline comment in the overrides file).
- The set of CSS classes emitted around an element (e.g. `figure.table` wrapper, `.media`, `.image-inline`).
- Default values of `--ck-content-*` CSS variables.

A reasonable check after each upgrade:

1. `pnpm run serve` and open `index-both.html`.
2. Scroll through every section: welcome letter, tables, headings, lists, images, links, code, legacy attributes.
3. Compare CKE4 (left) and CKE5 (right) for visible differences.
4. For any regression, inspect the DOM in DevTools to see what CKE5 is now emitting, and either:
   - Update an existing override rule to match the new selector/attribute, or
   - Add a new rule. Keep them grouped by the existing section comments so the file stays readable.

The override file fits in a single review session: ~290 lines total, more than half of which is actual CSS.

### When you should NOT add an override

Some differences are acceptable and don't need fixing — for example, CKE5's improved widget toolbars that have no CKE4 equivalent, or features that CKE4 didn't have (mentions, find-and-replace, etc.). The override stylesheet only addresses *content rendering*, not editor UI chrome.

### Adapting for your project

When you copy this repository as a starting point:

1. Replace `index-ckeditor5.html`'s editor configuration with whatever subset of plugins your project actually uses. Keep the **CKE4-shaping decisions** (no `ImageBlock`, `PlainTableOutput`, GHS allow-list, heading list, font-size/family `supportAllValues`).
2. Load `app/ckeditor5/ckeditor5-content-overrides.css` in your build pipeline alongside the vendor `ckeditor5-content.css`.
3. Adjust the override values that are taste-specific (link color, font family, blockquote font) to match your design system. The selectors are the load-bearing part; the values are negotiable.
4. Decide on each entry in `app/ckeditor5/ckeditor5-content-overrides.css`: do you want CKE4's look here, or do you prefer CKE5's default? Delete any rule you don't need.

## License

The CKEditor 5 packages in `node_modules/ckeditor5/` are governed by [CKEditor 5's license](https://ckeditor.com/legal/ckeditor-licensing-options). The CKEditor 4 build in `vendor/ckeditor4/` ships with its own license terms (see `vendor/ckeditor4/LICENSE.md`). The override stylesheet and demo HTML in this repository are MIT-licensed.
