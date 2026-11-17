# Text, markup, document, and text-based media formats

> This guide describes InfiniScribe's current document-viewing scope. See the app [architecture](architecture.md), [storage integration](storage.md), and [documentation index](README.md). Format registration is not a guarantee that every file variant renders.

Current implementation ownership: libinfiniscribe/scribe-core owns the existing MarkdownRenderer and bounded charset/text decoding. Text, Markdown, HTML and CSV viewers consume it through MediaSourceAdapter while retaining original complete bytes for revision checks. Android WebView, editing UI and guarded save/recovery remain host adapters. Input and actual text-output metadata distinguish encoding, completeness and warnings.

Debug builds of the text viewer offer an optional **Reparse** checkbox for JSON,
YAML, XML and TOML. Scribe's separate `scribe-reparse-*` modules reconstruct a display representation on
the document worker before Android performs word wrapping. Files with a known
size up to 4 MiB are cached in memory; paragraphs longer than 16 Ki characters
use a bounded rendering window. Editing always restores the original source.
Malformed input falls back to the original text. YAML reparsing retains scalar
spelling and comments; TOML's reconstructed view can omit comments and normalize
value spelling. XML preserves mixed content and `xml:space`, and declines DTDs.
The source file changes only through an explicit editor save.

Updated 2026-11-02. This guide covers text, markup, document, ebook, and text-based media routing across InfiniScribe's eight viewer modules: text, Markdown, HTML, PDF, EPUB, CSV, font, and legacy mind-map. It records actual implemented behavior and the remaining format boundaries. The dedicated [document and ebook guide](document-and-ebook-readers.md), [font preview guide](font-viewer.md), and [DOS/ANSI guide](dos-text-and-ansi-music.md) contain the detailed coverage for those routes.

---

## Implemented

In addition to the six routes below, the app has bounded read-only RTF, FB2,
MOBI/KF8, supported Office/OpenDocument, and font-preview routes. Their partial
coverage and limits are documented in [document and ebook readers](document-and-ebook-readers.md)
and [font preview](font-viewer.md). CP437/DOS text and supported ANSI music are
covered in [DOS text and ANSI music](dos-text-and-ansi-music.md).

### 1. Plain text & source code (`:viewer-text`)

`TextViewerActivity` provides a monospace viewer and in-place editor with UTF-8 / UTF-16 BOM detection, line numbers, a word-wrap checkbox, find-in-page with match highlighting, undo/redo stacks, and source-revision-checked save/Save As. The optional Reparse checkbox runs JSON, YAML, XML or TOML reconstruction before word wrap:

- **Plain text, logs & configuration**: `.txt`, `.log`, `.cfg`, `.conf`, `.ini`, `.properties`, `.toml`, `.env`, `.gitignore`, `.gitattributes`, `.editorconfig`, `.bashrc`, `.zshrc`, `.profile`
- **Shell & automation scripts**: `.sh`, `.bash`, `.zsh`, `.fish`, `.bat`, `.cmd`, `.ps1`, `.tcl`, `dockerfile`, `makefile`, `.mk`, `.cmake`, `.gradle`
- **Programming languages**: `.py`, `.kt`, `.kts`, `.java`, `.c`, `.cpp`, `.cc`, `.cxx`, `.h`, `.hpp`, `.cs`, `.go`, `.rs`, `.rb`, `.php`, `.js`, `.mjs`, `.cjs`, `.ts`, `.tsx`, `.jsx`, `.lua`, `.r`, `.swift`, `.dart`, `.asm`, `.s`, `.pl`, `.pm`, `.sql`
- **Structured data & patches**: `.json`, `.xml`, `.yaml`, `.yml`, `.diff`, `.patch`
- **Magic-byte fallback**: Extensionless files with UTF-16 BOM, `#!` shebangs, JSON/XML prefixes, or printable UTF-8 content (<5% non-whitespace control bytes and no NUL bytes) are classified as `text/plain` by `FileMagicDetector` (noting that `0x1B` ESC is explicitly permitted in `detectTextFormat` so ANSI escape sequences pass text detection).

### 2. Markdown (`:viewer-markdown`)

`MarkdownViewerActivity` renders CommonMark with GFM tables, strikethrough, task-list
items, and autolinks, alongside source editing with Save / Save As. Raw HTML is
escaped and unsafe link/image URLs are sanitized before the HTML reaches WebView:

- **Extensions & MIME**: `.md`, `.markdown`, `.mdown`, `.mkd`, `.mkdn` (`text/markdown`, `text/x-markdown`)
- **Supported syntax**: Headings (`#`–`######`), bold/italic/strikethrough, inline code and fenced code blocks, blockquotes, ordered/unordered lists, task lists (`- [ ]` / `- [x]`), tables, horizontal rules, links, and images.

### 3. HTML & XHTML (`:viewer-html`)

`HtmlViewerActivity` provides a sandboxed `WebView` renderer (with external network requests blocked and relative VFS asset loading) plus a Rendered / Source toggle and in-place editing:

- **Extensions & MIME**: `.html`, `.htm`, `.xhtml` (`text/html`, `application/xhtml+xml`)

### 4. PDF documents (`:viewer-pdf`)

`PdfViewerActivity` uses Scribe's `scribe-android` PDF backend, which wraps the
existing Android `PdfRenderer`. The host retains pinch-to-zoom (`ZoomImageView`),
fit-width / fit-page toggle, previous/next page buttons, direct page jump and
explicit staging leases. Page work is serialized; stale frames are discarded,
and document destruction releases the renderer before the staging lease.

- **Extensions & MIME**: `.pdf` (`application/pdf`, `%PDF-` magic header)

### 5. EPUB eBooks (`:viewer-epub`)

`EpubViewerActivity` uses bounded leased-directory ZIP expansion from the pinned Scribe scribe-office dependency and
Scribe's portable EPUB pull processor. Scribe reads container.xml, OPF metadata,
manifest/spine order and NCX chapter titles through common related-resource
sources. Existing manifest/HTML-title fallback is retained, including EPUB 3
spines; EPUB 3 NAV-label interpretation remains follow-up. The host renders
chapters with Dark/Sepia/Light themes, adjustable font scaling, TOC/Previous/Next
and fragment-preserving internal links. Cache leases remain active through
presentation, and loading cancels before worker cleanup releases them.

- **Extensions & MIME**: `.epub` (`application/epub+zip`)

### 6. Delimited spreadsheets (`:viewer-csv`)

`CsvViewerActivity` parses comma- and tab-delimited tables with quoted fields and
embedded newlines, quote-aware delimiter detection, row numbering, column
search/filtering, cell inspection/editing, and in-place saving. Cell whitespace
is preserved, and UTF-8 BOMs are removed from the first cell while retained in
the original bytes used for guarded saves:

- **Extensions & MIME**: `.csv`, `.tsv` (`text/csv`, `text/tab-separated-values`)

---

## Unimplemented

### Difficulty Rating Scale

- **Tier 1 — Trivial**: Charset table registration, delimiter expansion, or simple regex/tag transformation (<150 LOC).
- **Tier 2 — Low**: Self-contained state machine, binary cell grid, or XML-to-HTML converter (150–500 LOC).
- **Tier 3 — Medium**: Requires a virtual terminal screen buffer + bitmap font renderer, a real-time tone synthesizer synchronized with text output, or multi-file ZIP/XML document styling (500–2,000 LOC).
- **Tier 4 — High**: Requires a character-cell art/music editor UI, complex binary compound-document (OLE2 / PDB) parsing, or full page layout engine (2,000+ LOC).
- **Tier 5 — Very High**: Full TeX/LaTeX typesetting engine or DjVu IW44/JB2 wavelet/bi-level decoder.

---

### 1. Terminal and legacy media gaps

The bounded CP437/DOS text viewer and supported ANSI music path are implemented;
see [DOS text and ANSI music](dos-text-and-ansi-music.md). This does not provide
full terminal emulation, arbitrary binary terminal-art formats, animated BBS
sessions, or a character-cell art editor. Image-format previews are owned by the
Canvas library and Infinifolder's image UI.

### 2. Plain Text & Code Viewer (`:viewer-text`) Enhancements

| Feature / Format | Extensions | Viewer Difficulty | Creation Difficulty | Prerequisites |
| --- | --- | --- | --- | --- |
| **Legacy & CJK Character Encodings** | Any text file (`CP437`, `CP850`, `Windows-1252`, `Shift_JIS`, `GB18030`, `Big5`, `EUC-KR`) | **Tier 1 (Trivial)** | **Tier 1 (Trivial)** | **Prerequisites**: Encoding selector dropdown in `TextViewerActivity` + heuristic charset detector using `java.nio.charset.Charset.forName(...)` (all standard legacy and CJK charsets are already built into Android's ICU-backed JVM runtime) when reading and saving bytes. |
| **Subtitles, Synchronized Lyrics & Cue Sheets** | `.srt`, `.vtt`, `.ass`, `.ssa`, `.lrc`, `.cue` | **Tier 1 (Plain Text)** / **Tier 2 (Formatted Preview)** | **Tier 1 (Plain Text)** / **Tier 2 (Timestamp Helper)** | **Prerequisites**: Register extensions in `MediaTypeDetector.TEXT_EXTENSIONS` & `MimeRegistry`. Formatted preview parses timecodes (`00:00:00,000 --> 00:00:02,000` or `[mm:ss.xx]`) into a timestamp-aligned table. |
| **Unix Manual Pages & troff/groff** | `.1`–`.9`, `.man`, `.mdoc`, `.roff` | **Tier 2–3 (Low–Medium)** | **Tier 2 (Low)** | **Prerequisites**: Macro interpreter for `.TH`, `.SH`, `.SS`, `.TP`, `.PP`, `.B`, `.I`, `.BR` (man) and `.Dd`, `.Dt`, `.Nm`, `.Nd`, `.Fl`, `.Ar` (mdoc), rendering to styled HTML in `:viewer-html` or spans in `:viewer-text`. |
| **Hex / Binary Inspector Mode** | Any binary file | **Tier 2 (Low)** | **Tier 3 (Medium)** (in-place byte patching) | **Prerequisites**: Windowed `RandomAccessFile` / stream pager rendering 16 bytes per row (`offset | hex bytes | ASCII / CP437 glyphs`) without loading the entire file into a `String`. |

---

### 3. Rich Text, Markup & Help Formats (`:viewer-html` / `:viewer-markdown`)

| Format | Extensions | Viewer Difficulty | Creation Difficulty | Prerequisites |
| --- | --- | --- | --- | --- |
| **MIME HTML Web Archives** | `.mht`, `.mhtml` | **Tier 2 (Low)** | **Tier 2 (Low)** | **Prerequisites**: `WebView.saveWebArchive` (creation) and multipart MIME boundary parser (`Content-Type: multipart/related`, `Content-Location`, `Content-Transfer-Encoding: quoted-printable / base64`) feeding `WebViewClient.shouldInterceptRequest` in `HtmlViewerActivity`. |
| **reStructuredText, AsciiDoc & Org-mode** | `.rst`, `.adoc`, `.org` | **Tier 3 (Medium)** | **Tier 1 (Trivial)** (plain text editor) | **Prerequisites**: Block/inline parser in `:viewer-markdown` converting directives, tables, and headings into HTML for the existing `WebView` previewer. |
| **LaTeX / TeX Documents** | `.tex`, `.ltx`, `.bib` | **Tier 3 (Math/Subset HTML)** / **Tier 5 (Full TeX Engine)** | **Tier 1 (Trivial)** (source editor) | Planned Scribe typesetting backend with explicit related resources, fonts, bounded execution and platform packaging. A math/HTML subset must be described as a subset. Guarded source editing can use the text viewer when explicitly opened as text; it is not typeset rendering. No typesetting backend or implementation schedule is currently documented. |
| **AmigaGuide** | `.guide` | **Tier 2 (Low)** | **Tier 2 (Low)** | **Prerequisites**: `@database`, `@node`, `@endnode`, and `@{"label" link "node"}` hypertext parser rendering interactive node pages in `:viewer-html` or `:viewer-text`. |
| **Compiled HTML Help (CHM)** | `.chm` | **Tier 4 (High)** | **Tier 4 (High)** | **Prerequisites**: MS ITSS (`ITSF` / `ITSP`) header parser + LZX (`LZXC`) reset-table decompression stream (can share LZX code with the native CAB archive reader) + internal `#SYSTEM` / `.hhc` / HTML page routing into `HtmlViewerActivity`. |

---

### 4. Paginated Documents, eBooks & Office Formats (`:viewer-pdf` / `:viewer-epub` / `:viewer-csv`)

| Format | Extensions | Viewer Difficulty | Creation Difficulty | Prerequisites |
| --- | --- | --- | --- | --- |
| **PDF Creation / Export (from Text, HTML, Markdown, Images)** | `.pdf` | PDF viewing implemented | **Tier 2 (Low)** | General export is planned separately; the New File PDF template does not establish arbitrary-document export. Platform printing/PDF generation can be an explicit backend. |
| **PostScript / EPS documents** | `.ps`, `.eps` | **Tier 4–5 (Full interpreter)** | **Tier 3–5** | Existing Canvas decoding handles specific raster/preview encodings. Full vector interpretation and multi-page document viewing are planned through Scribe backends. New vector and two-page fixtures exercise this distinction. |
| **DjVu Documents** | `.djvu`, `.djv` | **Tier 5 (Very High)** | **Tier 5 (Very High)** | **Prerequisites**: `AT&TFORM....DJVU`/`DJVM` chunk parser + ZP-coder arithmetic entropy decoder + JB2 bi-level mask decoder + IW44 wavelet background/foreground decoder (`DjVuLibre` JNI). |
| **dBASE / FoxPro & SQLite Tables** | `.dbf`, `.sqlite`, `.db` | **Tier 2 (Low)** | **Tier 2 (Low)** | **Prerequisites**: 32-byte `.dbf` header + fixed-width field descriptor reader (or `android.database.sqlite.SQLiteDatabase.openDatabase` for `.sqlite`/`.db`) populating the tabular grid in `:viewer-csv`. |
