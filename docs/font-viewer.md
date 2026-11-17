# Font preview

> Historical implementation and validation details. Current application
> ownership is described in the [architecture guide](architecture.md), and the
> standalone build command is in the [application README](../README.md).

Tap a supported font with InfiniScribe's built-in viewers enabled, or open it
through Android's Open with flow. The standalone app includes the font preview.

| Files | Backend |
| --- | --- |
| `.ttf` | TrueType font |
| `.otf` | OpenType font |
| `.ttc`, `.otc` | TrueType/OpenType collections, with a face selector |

The screen provides custom sample text, an 8–144 sp size control, alphabets,
numbers, punctuation and multilingual samples, plus a light/dark specimen toggle.
The selected face, sample, size, preview background and scroll offset survive
Activity recreation. Details show available family/style/full/PostScript names,
glyph count, units per em and weight. Variable fonts show their axes and render
the default instance; axis editing is not implemented.

## Ownership and actual open path

- **Store:** MIME associations/aliases and bounded signature detection. Font
  identification does not imply a ROM, archive, image or executable capability.
- **Scribe core:** `FontFormats` and `FontDocuments.inspect`. This reads SFNT table
  directories, names and variation metadata with bounds checks; it does not
  implement a new glyph decoder.
- **Scribe Android:** `FontTypefaces.open` delegates real shaping/rasterization to
  Android's [Typeface.Builder](https://developer.android.com/reference/android/graphics/Typeface.Builder),
  including collection indices. A failed load raises an error; no default font is
  substituted as a successful file load.
- **viewer-font:** native Android text widgets, collection selection and specimen
  controls. `MediaTypeDetector` → `ViewerCatalog` → `ViewerLaunch` connects the
  normal browser/shortcut/external-intent paths to `FontViewerActivity`.
- **Shared source adapter:** `MediaSourceAdapter` in libinfiniui stages provider
  and archive sources when a native decoder requires a file. `stageBounded` adds
  a limit for font inspection while other viewers retain the ordinary staging API.

Canvas retains graphics/ANSI font rendering. No Canvas decoder or storage
implementation is copied into the font viewer.

Metadata follows the OpenType
[font-file](https://learn.microsoft.com/en-us/typography/opentype/spec/otff) and
[naming-table](https://learn.microsoft.com/en-us/typography/opentype/spec/name)
structures. Inspection is capped at 64 MiB, 256 faces, 4,096 tables/name records
per face and 64 variation axes. Table ranges and string offsets must remain
inside the supplied file. Reads/native font loads run on a single worker, with
stale-result rejection and cancellation on destruction. Local files are used
directly; cached sources retain a lease until the viewer closes. Unknown-length
streams also enforce the byte limit while staging, rather than after downloading
an arbitrarily large font. Permission, cache capacity and renderer failures show
an error without changing the source.

The shared MIME/icon registry and generated Android registrations include these
formats and legacy MIME aliases. Font icons use the document base and extension
label. Read-only font previews are not advertised as editors. WOFF/WOFF2, Type 1,
FON/FNT, PSF and other legacy bitmap/vector font containers have no new renderer.
Actual native support, color fonts and shaping depend on the Android device.
Android may supply fallback glyphs for characters missing from a loaded font;
the specimen explicitly explains this. Viewing never installs a system font.

## Build and runtime status

Build the standalone app with `./build.sh --max-workers=1 --console=plain`;
APK details and prerequisites are in the [application README](../README.md).

The owner confirmed font preview, size control, dark/light mode, rotation,
navigation-bar layout, and use with the keyboard open in 2026. `content://` font
access, collection face selection, custom sample editing, archive/remote sources,
status-bar layout, cancellation, and device error presentation need separate
acceptance. Build and parser results do not establish native glyph rendering or
provider grants.

### Automated regression coverage — 2026-11-03

The full app/viewer/library gate reports **980 cases, all passing**, with no skips,
new failures or missing protected cases. All 30 historical baseline failures now
pass in the tested workspace. Historical baseline files are retained. The run
includes existing pending workspace changes.

Twenty-two new tests exercise the actual portable font reader, MIME/detection
services and shared viewer/icon registry contracts:

- Fourteen Scribe tests cover Unicode/typographic names, metrics, collection
  offsets and face names, variable-axis metadata, unnamed fonts, malformed tables,
  truncated/unsupported inputs, the 64 MiB size limit and cancellation.
- Four Store tests cover font MIME aliases and signatures, source MIME refinement
  and the non-audio GEN association.
- Four shared-viewer tests cover read-only font dispatch, unsupported font routes,
  GEN identity/user overrides and inspection of the real bundled CP437 TTF without
  modifying its bytes.

These JVM tests inspect metadata and routing; native glyph rendering and provider
grants remain Android runtime behavior. See the
[historical recorded results in Infinifolder](https://github.com/infiniforge/infinifolder/blob/main/docs/validation/font-gen-automated-2026-11-03.json). This is a historical workspace-level result, not a current coverage count. Use
`./build.sh` to build the app from this checkout.
