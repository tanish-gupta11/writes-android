# Writes 0.6.0 release assessment

Date: 2026-10-02. **Decision: not qualified for customer production release as a Word/Acrobat replacement.** This is a hardened beta for limited evaluation using disposable copies. No public launch was performed.

## Verified for this build

- [x] 14 JVM tests passed: document codec/formatting/unsupported-feature warnings, invalid XML/document rejection and page-selection parsing.
- [x] 60 Android checks passed twice in `com.writes.qa` on one connected Samsung SM-X115.
- [x] Regression cases cover replacement/export consistency, pre-mutation size checks, draft recovery and corrupt-file quarantine, table page breaks, selected/overlapping/canceled printing and Activity recreation while reading PDFs and exporting.
- [x] Existing document, table, image, PDF page operations, comments, search and conversion tests remain green.
- [x] Android system-picker DOCX Save As completed; independent `python-docx` parsing confirmed the full saved draft text.
- [x] Independent `python-docx`/`pypdf` checks validated synthetic reference images, mixed styles, colour, text and round trips.
- [x] All four synthetic reference-output pages were rendered with Poppler and visually inspected. A spot check of the newly exported real document confirms remaining layout differences; it is not a full visual certification.
- [x] The owner's local 19-page PDF was processed on device: no pages lacked selectable text; 33,863 non-whitespace characters matched PDFBox baseline extraction; editable DOCX save/edit/reopen passed; original PDF hash unchanged.
- [x] Signed release verifies with v2/v3; package `com.writes`, version 0.6.0/code 7, Android 8+. No requested permissions or debuggable flag. QA harness classes and private document fixtures are absent from the release APK.
- [x] Release installed over the previous main app with `adb install -r`; no main-app data clearing or draft editing used for tests.

These checks are evidence for specific behavior, not a guarantee against all crashes, content loss or security defects. The manual release build is not R8-shrunk. The main app and QA APK use the same production sources but separate application data.

## Production blockers for the requested product

- [ ] Faithful editable PDF reconstruction: the 19-page input exports as **24 editable PDF pages**, with PDF tables/graphics not reconstructed as editable objects. Reference mode contains 19 raster page images plus separate text sections, yielding **44 pages**. Matching extracted text without whitespace does not prove spacing, reading order, visual fidelity or semantic correctness.
- [ ] OCR for scanned PDFs/images is not implemented. Direct PDF text/image object editing is not implemented. PDF forms, signatures, reliable redaction, bookmarks and attachments are not implemented.
- [ ] Word compatibility is partial: no tracked-change workflow, DOCX comments, faithful headers/footers, fields, complex tables/layout, floating objects or legacy `.doc`. Import warnings reduce surprise but do not preserve unsupported content.
- [ ] Desktop Word/LibreOffice visual round-trip qualification is incomplete: the prescribed DOCX renderer's bundled environment has no LibreOffice. XML/structure tests cannot substitute for it.
- [ ] Broader Android device/API/low-memory, accessibility, multilingual/RTL/IME and long-session testing remains outstanding.
- [ ] True process death during active disk writes, out-of-storage/provider interruption, revoked URI access and power-loss fault injection are not fully covered. Activity recreation is tested but is not process-death testing.
- [ ] Actual physical-printer output and the complete Android print-dialog range workflow remain unverified; the range-writing backend is tested.
- [ ] Security/dependency and untrusted-document fuzz testing, measured performance budgets, signing-key recovery and a repeatable release/rollback process still need qualification.

## Reproduction

Use `tools/run-regression.ps1` with the local SDK, Java, private signing prerequisites and JUnit dependencies installed. It exclusively modifies the QA package and saves logs under ignored `build/regression`. Run `tools/verify-device-artifacts.py` with `python-docx` and `pypdf` against artifacts pulled from QA's `files/verification` directory. Keep private documents and rendered outputs out of source control and public releases.

The limited closed-test agreement should explicitly identify unsupported formats/features, require retained originals, and ask testers to report reproducible defects using non-sensitive sample documents. Passing the current suite must not remove the production blockers above.
