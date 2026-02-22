# Easy HTML Partials

A zero-install browser tool for converting a flat HTML site into a partials-based workflow.

Drop in your site folder, review what it finds, download extracted partials and updated source files ready to use with `{{> partial-name}}` includes. A second tab compiles those includes back to flat HTML for deployment.

**No Node, no npm, no install of any kind.** Open `index.html` in a browser and go.

---

## How it works at a glance

```
Your site folder         →  Extract tab  →  partials/ folder + updated source files
                                                     ↓
Deployment-ready HTML    ←  Compile tab  ←  same folder (with partials/)
```

---

## Extract tab — turning a flat site into partials

### Step 1 — Drop your site folder

Drag your entire site folder onto the drop zone, or click to open a folder picker.

The tool reads all `.html` / `.htm` files recursively. It automatically skips:

| Skipped folder | Why |
|---|---|
| `partials/` | Already a partials folder — nothing to extract |
| `dist/` | Build output, not source |
| `node_modules/` | Never source HTML |
| `.git/` `.svn/` | Version control internals |

You should see a chip for every source file found. If a file is missing, check that its folder name isn't on the skip list.

### Step 2 — Review candidates

After clicking **Detect Partials**, the tool scans every file for repeated or semantically meaningful blocks and lists them as candidates.

Each candidate shows:

- **Tag name** — the HTML element type (`<header>`, `<nav>`, `<footer>`, etc.)
- **Badge** — one of three states:

  | Badge | Meaning |
  |---|---|
  | `repeated` (green) | Byte-for-byte identical block found in 2+ files |
  | `similar` (blue) | Same structure across files, but volatile attributes differ (e.g. `class="active"` on a different link per page) |
  | `semantic` (yellow) | Found in only one file, but it's a meaningful structural element |

- **File count** — how many pages the block appears in
- **Name input** — the name that will become `{{> name}}` in your source files and `partials/name.html` as the output file

**What "similar" means in practice:**

If your nav looks like this across pages —

```html
<!-- index.html -->
<nav>
  <a href="/" class="active">Home</a>
  <a href="/about">About</a>
</nav>

<!-- about.html -->
<nav>
  <a href="/">Home</a>
  <a href="/about" class="active">About</a>
</nav>
```

— the tool detects these as the same partial despite the `class="active"` difference. The extracted partial will use the version from the first file encountered, with no active state set. You handle active states separately (JavaScript, server-side, or a build step).

**Attributes treated as volatile (ignored during matching):**
`class`, `aria-current`, `aria-selected`, `aria-expanded`, `aria-pressed`, `aria-checked`, `tabindex`

All other attributes (`href`, `src`, `id`, `data-*`, etc.) are part of the block's identity and must match.

#### Customising candidates

- **Toggle the checkbox** on any candidate to include or skip it
- **Edit the name field** to rename a partial — use lowercase letters, numbers, and hyphens only (e.g. `site-header`, `main-nav`, `page-footer`)
- **Click ▼ preview** to expand the raw HTML and see exactly which files the block came from

### Step 3 — Download results

Click **Extract Selected** to generate your files.

You get:

1. **`name.html`** — one file per extracted partial, containing just the block's HTML
2. **Updated source files** — your original pages with the extracted blocks replaced by `{{> name}}` placeholders

Example — before extraction:

```html
<!-- index.html -->
<html>
<body>
  <header>
    <a href="/"><img src="logo.svg" alt="My Site"></a>
  </header>
  <main>
    <h1>Welcome</h1>
  </main>
  <footer>
    <p>&copy; 2026 My Site</p>
  </footer>
</body>
</html>
```

After extracting `header` and `footer`:

```html
<!-- index.html (updated) -->
<html>
<body>
  {{> header}}
  <main>
    <h1>Welcome</h1>
  </main>
  {{> footer}}
</body>
</html>
```

```html
<!-- partials/header.html -->
<header>
  <a href="/"><img src="logo.svg" alt="My Site"></a>
</header>
```

```html
<!-- partials/footer.html -->
<footer>
  <p>&copy; 2026 My Site</p>
</footer>
```

Each output file is shown with a **↓ Download** button. **Download All Files** triggers all downloads sequentially. Save partial files into a `partials/` folder inside your project.

---

## Compile tab — turning partials back into flat HTML

Use this when you need deployment-ready flat HTML files (no server, no runtime dependencies).

### Step 1 — Drop your site folder

Drop the same folder you work in — the one containing both your source pages (with `{{> }}` includes) and the `partials/` subfolder.

Unlike the Extract tab, this tab **does not skip** `partials/`. It needs those files to resolve the includes.

### Step 2 — Compile

Click **Compile**. The tool:

1. Identifies partial files — any `.html` file that lacks a full `<html>` structure, or any file inside a folder named `partials`
2. Builds a lookup map of `partial-name → content`
3. Resolves every `{{> partial-name}}` include in each source file
4. Handles nested partials (a partial that itself contains `{{> }}` includes)

### Warnings

If a source file contains a `{{> name}}` that doesn't match any uploaded partial, a warning is shown:

```
⚠ about.html: unresolved partials — sidebar
```

The placeholder is left in place in the output so you can see exactly what was missed.

### Step 3 — Download

Each compiled file is shown with its resolved HTML and a **↓ Download** button.

---

## Include syntax reference

| Syntax | Resolves to |
|---|---|
| `{{> header}}` | `partials/header.html` |
| `{{> partials/header}}` | `partials/header.html` (path prefix stripped) |
| `{{> footer.html}}` | `partials/footer.html` (extension stripped) |
| `{{>  nav  }}` | `partials/nav.html` (whitespace trimmed) |

Nesting works — partials can include other partials. The depth limit is 10 levels to prevent infinite loops from circular references.

---

## Running the tests

Open `test.html` in any modern browser. Tests run automatically on load.

The suite covers:

| Suite | What it tests |
|---|---|
| `hashStr` | FNV-1a hash consistency and sensitivity |
| `normalise` | Whitespace collapsing and trimming |
| `structuralHTML` | Volatile attribute stripping for fuzzy matching |
| `detectCandidates` | Detection, repeated/similar/semantic classification, sort order |
| `replaceBlock` | Block replacement and placeholder injection |
| `resolveIncludes` | Include resolution, unknown partials, nesting, circular depth limit |

No install required. No test runner to configure.

---

## Project structure

```
index.html   — the tool itself (self-contained, no dependencies)
test.html    — browser-based test suite (self-contained)
LICENSE
README.md
```

---

## Limitations and known edge cases

**Extraction uses the first-seen version of a block.**
For `similar` candidates, the HTML stored is from whichever file was processed first. If that version has an active-state class and you'd prefer a clean one, edit the downloaded partial file before using it.

**The compile step rewrites the full document.**
Because browsers normalise HTML when parsing it, the compiled output may differ in minor ways from the source (e.g. attribute quote style, self-closing tags). The content will be correct; formatting may shift slightly.

**Partial detection only covers semantic HTML elements.**
The tool looks inside `<header>`, `<footer>`, `<nav>`, `<aside>`, `<main>`, `<section>`, and `<article>`. A repeated `<div class="banner">` won't be auto-detected. You can still use `{{> }}` includes manually for any block — the compile tab resolves them regardless of how they got there.

**`webkitdirectory` is required for folder-picker mode.**
All modern browsers support it. If the click-to-browse picker opens in file mode instead of folder mode, use drag-and-drop instead.
