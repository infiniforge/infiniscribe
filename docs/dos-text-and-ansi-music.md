# DOS text and ANSI music

InfiniScribe owns the CP437/DOS text viewing and optional ANSI music playback UI.
Detection and rendering use bounded reads; text remains selectable. Reusable
character art decoding and the supported music parser/synthesizer are provided by
linked libraries. This page describes implemented behavior and limits, not a
full terminal emulator.

## Detection and rendering

Every text source receives a bounded 16 KiB encoding probe. Positional sources
keep their cursor and random-access capability; sequential sources replay the
probe prefix. Unicode BOMs and valid multibyte UTF-8 determine encoding. Repeated
valid ANSI controls, CP437 box/line patterns, OEM letter patterns, and SAUCE
metadata provide content evidence independently of the extension. DOS filename
hints and 8.3 names with DOS line endings/EOF are secondary clues. Single-byte
encodings cannot always be identified automatically; the text-source encoding
picker includes manual CP437 and Automatic options.

DOS text opens in the CP437 viewer with fixed 80-column cells, VGA/extended ANSI
colors, cursor movement and erase controls, saved cursors, tabs, CR/LF and DOS
EOF. The view uses native text selection and scrolling, pinch zoom and fit-width
controls. Zoom changes cell size without reflowing the columns. Long documents
remain native text rather than a full-document bitmap. The terminal view is
bounded to 4 MiB source bytes, 65,536 rows, and 262,144 non-default style runs;
exceeding these limits fails explicitly while ordinary text viewing remains
available where supported.

UTF-8 ANSI art is decoded before color and cursor controls are applied, with or
without a UTF-8 BOM. Legacy files retain CP437 decoding. Unicode box/block
characters use CP437 cell geometry; other characters use Android font fallback.
The view uses one cell per printable scalar, so it does not implement full
Unicode wide-character or emoji terminal layout.

## ANSI music

The viewer can start and stop supported BASIC PLAY-style melodies embedded in
ANSI streams. It uses `libinfinisound`'s mono PC-speaker-style PCM synthesizer and
Android media output. Supported note controls include tempo, octave, default or
explicit note length, A–G accidentals, numeric notes, rests, dotted durations,
and normal/legato/staccato articulation. Recognized introducers include classic
`CSI M`, BananaCom `CSI N`, and `CSI |`, terminated by Ctrl-N. This is a bounded
dialect subset; `.ans` alone does not establish that a file contains music. The
static view does not synchronize a live animated BBS session with audio.

The current renderer reads music on user request and stops playback when the
viewer leaves the foreground. Generated PCM cache files are cleaned up with the
viewer lifecycle. This does not add general music-player routing or full terminal
emulation.

## Related ownership

- [Text and document format coverage](text-and-document-formats.md)
- [Audio playback formats and limits in InfiniMedia](https://github.com/infiniforge/infinimedia/blob/main/docs/audio-formats.md)
- [Folder's image and file-icon behavior](https://github.com/infiniforge/infinifolder/blob/main/docs/image-formats.md)
- [Storage integration](storage.md)
