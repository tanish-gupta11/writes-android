# Writes for Android

Writes is a separate, offline Android document editor. Download the [Writes 0.6.0 beta APK from GitHub Releases](https://github.com/tanish-gupta11/writes-android/releases/tag/v0.6.0) (Android 8.0 or newer). The APK requests no network or storage permission; files are chosen through Android's system picker.

## Features

- Create and edit text documents with headings, bold, italic, underline, lists, alignment, indentation and text size from a compact Word-style tab ribbon
- Text colour, highlight and strikethrough, preserved in DOCX/PDF/HTML exports; bold/italic/underline toggles and colour/font changes preserve unselected text
- Choose among six lightweight system font families, set an exact text size, and select A4, A5, Letter, Legal, B5 or Executive paper in portrait or landscape; these settings are used in DOCX/PDF export and printing
- Open `.docx`, `.txt`, `.html` and `.pdf` files
- Export editable work to `.docx`, `.pdf`, `.txt` and `.html`, including editable tables and inline PNG/JPEG pictures
- Insert and edit bounded tables (up to 8 rows × 6 columns), insert pictures, and keep rich DOCX drafts through process restarts
- Read PDFs page by page with zoom, and convert selectable PDF text into a new editable document with reconstructed font sizes, bold/italic runs, indentation and page breaks
- PDF conversion keeps mixed character styles and solid text colours, reports pages with no selectable text, and retains blank-page numbering. Choose editable text with raster page references (up to 30 pages / 24 MB of images), text only, or page images only
- PDF tools: rotate pages, extract/reorder pages, merge PDFs, add/read text comments, search selectable text, export a page as PNG, and export a page-image DOCX when editable text reflow would be unacceptable
- Save a byte-for-byte copy of the opened PDF without converting or changing it
- Use Android's print dialog for both documents and opened PDFs
- Find and replace, undo and redo, word/character counts, and a local autosaved draft

Exports use **Save As** so imported originals are not overwritten. The app works entirely on-device and does not contain accounts, ads, or analytics.

## PDF conversion in 0.5.0

Open a PDF, then tap **Convert to Word → Editable text + page references**. Each source page gets a labelled image reference followed by a separate editable-text section. Review the conversion report, edit the extracted text, and use **File → Save DOCX**. The image references retain the rendered page appearance at a maximum of 1600 pixels on the long edge; they are not editable Word objects, are scaled to fit, and are not lossless PDF preservation. Source PDF bytes remain unchanged. Text-only mode omits graphics and may change layout. Neither mode includes OCR or promises lossless conversion.

Document exports finish conversion in an app-private temporary file before opening the chosen destination. A conversion error therefore does not truncate the destination; a storage/provider failure during the final copy can still leave an incomplete exported file.

### Font scaling fix in 0.5.1

PDF font sizes now use the text rendering matrix, including graphics-state scaling. Version 0.5.0 could double text sizes in PDFs whose producer used half-scale graphics transforms. A new on-device regression test covers a 24-point font rendered through a 0.5 scale transform, which must convert as 12 points.

A locally supplied 19-page document was tested on the connected device. Editable DOCX save/edit/reopen succeeded; selectable text matched the library's baseline extraction after whitespace normalization, and the source bytes were unchanged. Correcting font sizes reduced editable PDF output from 52 pages to 24 pages. This still fails exact-layout fidelity: editable output has no reconstructed PDF tables or images. Reference mode included 19 page images, but its separate text sections produced 44 output pages. Private input and outputs are excluded from version control and are not packaged into the APK.

## Important compatibility limits

This is **not** a complete Microsoft Word or Adobe Acrobat replacement and is not qualified for customer production release. The tab ribbon groups implemented commands; it does not imply feature parity with the desktop Word ribbon. Font choices use Android system families, not a bundled commercial or premium font library; another device or desktop editor may substitute fonts. Paper choice controls exported document dimensions, not a WYSIWYG paged editor preview. DOCX text, paragraphs, headings, basic inline formatting, bounded tables and inline PNG/JPEG pictures are handled, but complex table styling, floating picture layout, images in arbitrary positions, headers/footers, comments, tracked changes, fields, complex page layout, macros and some styles are not preserved on export. Legacy `.doc` is not supported. PDF-to-document conversion reconstructs some styling from selectable text, but exact layout and scanned text are not preserved; scanned PDFs need OCR, which is not included. PDF tools operate on a cached copy and support page rotation, extraction/reordering, merging, comments, search and PNG export, but do not provide full PDF object editing, form filling, signatures, redaction, bookmarks, attachments, permissions, or conversion to spreadsheet/slides/image formats. Keep originals and use a copy for valuable or complex files.

## Reliability changes in 0.6.0

- Find & Replace now invalidates the original conversion model, so DOCX exports contain replacements. Oversized replacements are rejected before changing the document.
- New, Open and PDF conversion save one previous draft before replacement. **File → Recover draft** restores it. This is not a permanent backup or full document library.
- Unreadable drafts pause autosave instead of being silently overwritten. Explicit replacement preserves their bytes in a separate app-private recovery file.
- PDF page/zoom and pending export state survive Activity recreation. Document saves use a serialized process-wide queue; stale open results cannot replace a draft edited while loading.
- Printing honors selected page ranges and cancellation, runs off the UI thread and avoids duplicating overlapping ranges.
- Table page breaks survive editor/DOCX round trips. Decoded inline pictures share a bounded 12 MiB bitmap cache rather than retaining a bitmap per span.
- DOCX import detects several unsupported features and asks before simplifying them, including headers/footers, tracked changes, comments, merged cells and fields. Detection is not exhaustive and does not implement those features.

## Build and verification

Open the project in Android Studio with JDK 17, Android SDK Platform 35 and Build Tools 35.0.0, then run `:app:testDebugUnitTest` and `:app:assembleDebug`. The signed release APK is built with `tools/build-release.ps1` on the owner's Windows workstation using the local SDK and private release key; this manual build does not apply R8 shrinking. The private key is not included in distributable source. The PDFBox dependency is downloaded from Maven Central if needed. The key password backup is protected by that Windows account and must be separately secured before migrating to another machine.

On 2026-10-02, version 0.6.0 passed **14 JVM tests and 60 on-device checks**, twice for the device suite, on a Samsung SM-X115 in the isolated `com.writes.qa` app. Coverage includes table/picture/draft/PDF operations, mixed PDF fonts/sizes/colours, missing-text reports, image references, blank pages, rotated/scaled fonts, formatting round trips, recovery, corrupt-draft protection, selected-page printing, replacement exports and real Activity recreation. The signed release verifies with APK signature schemes v2 and v3. It is 2,428,724 bytes; SHA-256: `9260758BB86232CA7833155683A814ED1974F3422DE3AC698C733E74D013DA7C`.

Run `tools/run-regression.ps1` with one authorized device connected (or `-Serial DEVICE_ID`). It builds/installs the isolated QA package, runs JVM/device tests and fails on incomplete instrumentation results even when adb exits zero. Existing JUnit/Hamcrest jars in `build/vendor` and the release builder prerequisites are required. `-SkipBuild` reuses a previously built QA APK; `-SuppliedPdf` additionally tests an owner-provided PDF already placed in QA's `files/supplied.pdf`. Test logs are local, ignored build artifacts. The script does not approve production release or install over the main app.

Independent `python-docx` and `pypdf` checks validate the synthetic exported files using `tools/verify-device-artifacts.py`. Source and converted PDF pages were rendered with Poppler and visually inspected. A manual QA-app UI smoke test covered opening the fixture PDF and the new conversion chooser/report. Desktop DOCX visual verification remains incomplete: the prescribed renderer could not run because the runtime has no LibreOffice. These tests are synthetic, on one device, not full production qualification or Word/Acrobat parity.

In 0.6.0, the Android system-picker DOCX Save As check completed successfully; independent parsing confirmed the saved draft's full text. The supplied 19-page PDF was retested: text matched the library baseline ignoring whitespace, edits survived DOCX reopening, and original bytes remained unchanged. Editable PDF output still has 24 pages and reference output 44 pages; exact layout reconstruction remains unsupported. The main app was updated in place without clearing its data. See [RELEASE_CHECKLIST.md](RELEASE_CHECKLIST.md) for launch blockers and unverified areas.

The app's original code is MIT-licensed; [PDFBox-Android](THIRD_PARTY.md) keeps its Apache 2.0 license.
