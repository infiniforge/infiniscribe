# InfiniScribe architecture

InfiniScribe is an independent Android document viewer and editor. Its Gradle
root owns the application and its document modules. It does not include
Infinifolder or InfiniMedia as Gradle projects and does not import either
application's source.

## Application ownership

- The application root owns document launch routing, app configuration,
  user-facing navigation, and generated Android file-opening filters.
- `modules/document-common` owns app-level document staging and shared viewer
  integration.
- `modules/viewer-*` own text, Markdown, HTML, EPUB, CSV, PDF, font, and legacy
  mind-map presentation and editing workflows.
- `libinfiniscribe` owns reusable document parsing and rendering backends;
  `libinfiniui`, `libinfinicanvas`, and `libinfiniselect` provide shared public
  APIs.

Some library classes retain the `com.infinifolder.*` namespace for compatibility.
That package name does not indicate a dependency on the Infinifolder application.
Pinned library revisions and the standalone build are documented in the
[application README](../README.md).

## Storage boundary

InfiniScribe uses Android local/SAF workflows and the public `libinfiniselect`
API. When Infinifolder is installed, the library can resolve resources through
its provider; Infinifolder owns the remote/container backends and credentials.
Scribe does not package those implementations or read another app's private
files. With Infinifolder unavailable, local files and user-granted SAF resources
remain available. See [storage integration](storage.md).
