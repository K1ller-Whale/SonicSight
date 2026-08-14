# Rendering the SonicSight technical report

`TECHNICAL_REPORT.md` renders as-is on GitHub and in VS Code — the Mermaid
diagrams are inline fenced blocks and need no build step. This file covers the
two cases where a build step is needed: exporting a PDF for submission, and
exporting individual figures for slides.

Source of truth is the Markdown and the `.mmd` files. **Do not commit rendered
PDFs or SVGs.**

---

## 1. What to install

| Tool | Why | Install |
|---|---|---|
| Node.js 18 or newer | runs `mermaid-cli` | <https://nodejs.org> |
| `@mermaid-js/mermaid-cli` | renders `.mmd` and Markdown Mermaid blocks | `npm install -g @mermaid-js/mermaid-cli` |
| Pandoc 3.x | Markdown to PDF | <https://pandoc.org/installing.html> |
| A LaTeX engine | Pandoc's PDF backend — XeLaTeX recommended for Unicode | TeX Live, MiKTeX, or MacTeX |

Verify:

```bash
node --version
mmdc --version
pandoc --version
```

### 1.1 Read this before your first render

`mermaid-cli` drives a headless Chromium through Puppeteer, and on several
Windows machines — including the one this report was prepared on — its bundled
`chrome-headless-shell` crashes on launch:

```
Error: Failed to launch the browser process:  Code: 3221225595
```

That status is `0xC000027B`, a stowed exception inside the browser process. It
is not a problem with the diagram sources. The fix is to use a Chrome you
already have, and **`docs/puppeteer.json` in this directory already does that**:

```json
{
  "executablePath": "C:\\Program Files\\Google\\Chrome\\Application\\chrome.exe",
  "args": ["--no-sandbox", "--disable-gpu"]
}
```

Pass it to every `mmdc` invocation with `-p puppeteer.json`. All commands below
do. If your Chrome is elsewhere, edit the path:

| Platform | Typical path |
|---|---|
| Windows, Chrome | `C:\\Program Files\\Google\\Chrome\\Application\\chrome.exe` |
| Windows, Edge | `C:\\Program Files (x86)\\Microsoft\\Edge\\Application\\msedge.exe` |
| Linux | `/usr/bin/google-chrome` or `/usr/bin/chromium` |
| macOS | `/Applications/Google Chrome.app/Contents/MacOS/Google Chrome` |

If you rewrite the file, **write it without a byte-order mark**. `mermaid-cli`
parses it as strict JSON and a BOM fails with
`SyntaxError: Unexpected token '\ufeff'`. PowerShell's `>` and `Out-File` add
one; use `[System.IO.File]::WriteAllText(...)` instead.

If the bundled shell works on your machine, drop `-p puppeteer.json` and
everything else is unchanged.

---

## 2. Export the figures

Each figure has a standalone source in `docs/diagrams/`. Render them all to
SVG, which is what you want for slides and for print — vector, no resampling:

```bash
cd docs
mkdir -p build/figures
for f in diagrams/*.mmd; do
  mmdc -p puppeteer.json -i "$f" -o "build/figures/$(basename "${f%.mmd}").svg"
done
```

PowerShell:

```powershell
cd docs
New-Item -ItemType Directory -Force build\figures | Out-Null
Get-ChildItem diagrams\*.mmd | ForEach-Object {
  mmdc -p puppeteer.json -i $_.FullName -o "build\figures\$($_.BaseName).svg"
}
```

This renders all fourteen figures. Verified working on the documentation
machine with the configuration above.

For raster output — slide decks that will not accept SVG, or a quick visual
check — add a background and a width. White background matters: the default is
transparent, which renders as black text on black in a dark-themed viewer.

```bash
mmdc -p puppeteer.json -i diagrams/figure-10b-pipeline-server-inference.mmd \
     -o build/figures/figure-10b.png \
     -b white -w 2000
```

Useful flags:

| Flag | Effect |
|---|---|
| `-b white` | background colour; use it for any raster export |
| `-w 2000` | output width in pixels; raise it for dense figures such as 10b |
| `-s 2` | scale factor, an alternative way to increase raster resolution |
| `-t neutral` | built-in theme; the sources deliberately set none, so this stays a command-line choice |
| `-p puppeteer.json` | Puppeteer configuration; needed for the workaround in §5 |

Figure 11 is a Gantt chart and Figure 12 a state diagram; both export the same
way as the flowcharts.

---

## 3. Export the report to PDF

Pandoc does not understand Mermaid. There are two routes; the first is simpler
and is the recommended one.

### 3.1 Pre-render the diagrams, then convert

`mermaid-cli` will process a Markdown file directly, replacing each Mermaid
block with an image reference and writing the images alongside:

```bash
cd docs
mkdir -p build
mmdc -p puppeteer.json -i TECHNICAL_REPORT.md -o build/TECHNICAL_REPORT.rendered.md
```

That produces `build/TECHNICAL_REPORT.rendered.md` plus one SVG per diagram,
numbered in document order. Then:

```bash
pandoc build/TECHNICAL_REPORT.rendered.md \
  -o build/SonicSight-Technical-Report.pdf \
  --pdf-engine=xelatex \
  --from=gfm+tex_math_dollars \
  --toc --toc-depth=2 \
  --number-sections=false \
  -V documentclass=report \
  -V geometry:a4paper \
  -V geometry:margin=2.2cm \
  -V mainfont="DejaVu Serif" \
  -V monofont="DejaVu Sans Mono" \
  -V linkcolor=blue \
  -V urlcolor=blue \
  --highlight-style=tango
```

Notes on those options, because a few are load-bearing:

- `--from=gfm` is required. The report uses GitHub-flavoured Markdown tables;
  Pandoc's default reader handles them differently.
- `--number-sections=false` is deliberate. The report numbers its own sections
  in the headings, and letting Pandoc number them as well produces
  "1 1. Introduction".
- `mainfont` and `monofont` must exist on your machine. The report contains
  arrows, box-drawing characters and en dashes; a font without them silently
  drops glyphs. DejaVu is a safe default on Linux; on Windows try
  `"Cambria"` and `"Consolas"`, on macOS `"Times New Roman"` and `"Menlo"`.
- If XeLaTeX is unavailable, `--pdf-engine=tectonic` works and downloads what
  it needs on first run.

If a diagram comes out too small in the PDF, re-render that one at a larger
width before converting, or add `-V graphics=true` and set an explicit width in
the generated Markdown.

### 3.2 Convert with a Pandoc filter instead

If you would rather not keep a rendered copy, `mermaid-filter` does the
conversion during the Pandoc run:

```bash
npm install -g mermaid-filter
pandoc TECHNICAL_REPORT.md \
  -o build/SonicSight-Technical-Report.pdf \
  --pdf-engine=xelatex --from=gfm --toc --toc-depth=2 \
  -F mermaid-filter
```

This is one command, but the filter is a separate project from `mermaid-cli`
and can lag behind on Mermaid syntax. If a diagram fails to render under the
filter but succeeds under `mmdc`, use §3.1.

---

## 4. Export to HTML or DOCX

Self-contained HTML, diagrams embedded, opens anywhere:

```bash
pandoc build/TECHNICAL_REPORT.rendered.md \
  -o build/SonicSight-Technical-Report.html \
  --from=gfm --standalone --embed-resources --toc --toc-depth=2
```

Word, if the submission requires it. Convert the diagrams to PNG first — DOCX
handling of SVG is inconsistent across Word versions:

```bash
mmdc -p puppeteer.json -i TECHNICAL_REPORT.md -o build/TECHNICAL_REPORT.png.md -e png -b white -w 1800
pandoc build/TECHNICAL_REPORT.png.md -o build/SonicSight-Technical-Report.docx --from=gfm --toc
```

---

## 5. Troubleshooting

**`Could not find chrome-headless-shell`.** Puppeteer has not downloaded its
browser. Run:

```bash
npx puppeteer browsers install chrome-headless-shell
```

**`Failed to launch the browser process: Code: 3221225595`, or any immediate
browser crash.** See §1.1 — this is the common case on Windows and the fix is
`-p puppeteer.json`, which every command in this file already passes. If you
are hitting it, you dropped that flag or the path inside the file is wrong for
your machine.

**`Parse error on line N`.** The Mermaid source is invalid. Two causes have
actually occurred in this project: a semicolon inside a `sequenceDiagram`
`Note` text, which the parser reads as a statement separator; and unquoted
parentheses, commas or slashes in a node label. Wrap any label containing
punctuation in double quotes, and keep semicolons out of note text.

**Diagram renders but is unreadably wide.** Raise `-w`, or split the figure.
Figures 10a, 10b and 10c are already a split for this reason.

**Text is invisible in the exported PNG.** You omitted `-b white`. The default
background is transparent.

**Glyphs missing from the PDF.** The chosen font lacks them. Change
`mainfont` and `monofont`, or drop to `--pdf-engine=lualatex`.

---

## 6. One-shot build

Everything the submission needs, from a clean checkout:

```bash
cd docs
mkdir -p build/figures

# 1. standalone figures, vector
for f in diagrams/*.mmd; do
  mmdc -p puppeteer.json -i "$f" -o "build/figures/$(basename "${f%.mmd}").svg"
done

# 2. report with diagrams inlined as images
mmdc -p puppeteer.json -i TECHNICAL_REPORT.md -o build/TECHNICAL_REPORT.rendered.md

# 3. PDF
pandoc build/TECHNICAL_REPORT.rendered.md \
  -o build/SonicSight-Technical-Report.pdf \
  --pdf-engine=xelatex --from=gfm --toc --toc-depth=2 \
  -V geometry:a4paper -V geometry:margin=2.2cm \
  -V mainfont="DejaVu Serif" -V monofont="DejaVu Sans Mono"
```

`docs/build/` is derived output. Keep it out of version control.
