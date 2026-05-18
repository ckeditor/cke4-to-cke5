# cke4-to-cke5

CKEditor 4 to CKEditor 5 migration example.

## Setup

```bash
pnpm install
pnpm run serve
```

Then open `index-both.html` for a side-by-side comparison of CKE4 and CKE5 with the same content.

## Updating CKEditor 5

CKEditor 5 is consumed from npm (see `package.json`). To upgrade:

```bash
pnpm update ckeditor5
```

Or pin a specific version in `package.json` and run `pnpm install`.

CKEditor 4 stays vendored under `vendor/ckeditor4/` because it is not distributed via npm in a usable form.
