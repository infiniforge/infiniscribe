# InfiniScribe application

## Purpose

InfiniScribe is InfiniForge's independent document-oriented application for reading, inspecting, and selectively editing text and document content. Its long-term direction is a flexible document viewer/editor rather than a set of unrelated format shortcuts. The current implementation has useful viewers and editing paths, but format, editing, and rendering coverage remains incomplete.

## Primary use cases

- Open documents through Android intents, local files, SAF, or an optional Infinifolder storage handoff.
- Read and search text, source code, Markdown, HTML, EPUB, PDF, CSV/TSV, supported office/ebook formats, fonts, and legacy mind-map/text content.
- Make guarded edits to supported text, Markdown, HTML, or delimited-text documents and save or Save As when the source/provider permits it.
- Inspect document metadata, navigate supported pages/chapters/sheets, and open supported content stored remotely or inside a container through the resource-selection contract.

## Implementation status

Status note: implementation statements below reflect inspected source and current repository documentation. This audit did not rerun builds or device acceptance; existing checkpoints and remaining checks remain in the README and format guides.

### Implemented in this checkout

This checkout contains a standalone Android application whose viewer modules cover text, Markdown, HTML, EPUB, CSV/workbooks, PDF, fonts, and a supported legacy mind-map format; the text module also handles CP437/DOS and ANSI-related content. Actual behavior varies by reader and format. Current documentation describes plain-text/source editing, Markdown and HTML source editing, CSV/TSV cell editing, PDF page viewing, EPUB navigation, font preview, and bounded/read-only routes for office, RTF, FB2, and MOBI/KF8 formats. Office workbooks are read-only; formulas are not recalculated. This is not a full desktop office suite, general PDF editor, or universal conversion tool.

The [generated ecosystem map](ECOSYSTEM.md) lists component owners, dependencies, and current interapplication contracts.

The app owns document navigation, viewer/editor UI, selection of supported editing operations, source staging, and app-owned state. Parsing and format-specific processing are consumed through libinfiniscribe; common Android presentation and image viewing use libinfiniui/libinfinicanvas; libinfiniselect handles storage selection and resource resolution. The current dependency lock does not include the libinfinivault repository. ZIP resource staging used by document readers is provided by the pinned Scribe processing dependency even where the implementation retains legacy Vault class/package names.

## Responsibilities and shared libraries

- libinfiniscribe: reusable document parsers, readers, text processing, office/ebook backends, and capability providers.
- libinfiniselect: Android open/save selection, URI grants, authoritative resource references, and relative-resource resolution.
- libinfiniui and libinfinicanvas: shared viewer UI and image decoding/presentation.
- libinfinisound: selected soundbank text inspection and ANSI Music support used by the text/legacy-content route; it is not the media-player app.
- Android's PdfRenderer, WebView, and SAF contracts provide platform host behavior where used.

The app does not bundle Infinifolder's network clients or filesystem/archive storage engines. It should use public shared APIs for parsing and resource access rather than reimplementing them.

## Interoperability

Infiniscribe uses libinfiniselect and preserves authoritative resource identity separately from the current Android content URI. Infinifolder is an optional provider for its remote and container locations; SAF and local workflows remain available without it. Folder hands off documents through Android intents, content URIs, and grants. Media playback belongs to InfiniMedia, and the apps do not share private files or implementation classes.

## Planned and potential work

Potential work includes broader standards coverage, more capable but safe editing, improved spreadsheet/document navigation, format conversion, structured-data tools, and richer search/metadata workflows. Each needs an owning library API and reader-specific compatibility limits. Editing and conversion should be explicitly scoped by format and source write capabilities; unsupported files must remain read-only or fail clearly.

## Limitations and outstanding work

- “Universal” describes the direction; registered formats and formats with partial backends are not equivalent to complete rendering support.
- Office formats are read-only and limited; cached formula values may be shown but are not recalculated. PDF viewing does not imply annotation or editing.
- EPUB, MOBI/KF8, FB2, RTF, Office/OpenDocument, mind-map, and legacy text coverage has documented edge cases. Encrypted/DRM-protected and unsupported variants may fail explicitly.
- Save access depends on Android grants and provider write support. Compilation and JVM tests do not establish all SAF/provider, WebView, rendering, or lifecycle behavior on devices.
- This app does not own network storage clients, archive navigation as a general file-manager feature, or media playback.

## Documentation audit

- Consistent: the README and architecture guide assign viewer/editor UI to this app and reusable format parsing to linked libraries.
- Outdated/contradictory: some reader guides attributed ZIP expansion to the separate libinfinivault repository, while the current app lock and module build use libinfiniscribe's scribe-office artifact. The detailed reader guides are corrected to reflect the pinned dependency and retained legacy Vault type name.
- Incomplete: “viewer/editor” could be read as full editing support across all formats; this document calls out the actual editable routes and read-only office/PDF scope.
- Unverified: format edge cases and Android rendering/grant behavior still require the targeted checks listed in the reader guides.
