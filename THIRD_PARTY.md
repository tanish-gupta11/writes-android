# Third-party components

Writes uses [PDFBox-Android 2.0.27.0](https://github.com/TomRoush/PdfBox-Android) for extracting text from PDFs. It is licensed under the [Apache License 2.0](https://github.com/TomRoush/PdfBox-Android/blob/master/LICENSE). The build excludes optional Bouncy Castle cryptography libraries; encrypted PDFs are not a supported conversion input. Android's platform APIs provide PDF rendering, PDF generation and printing.

The development-only tests use JUnit 4.13.2, python-docx, pypdf and ReportLab; these are not bundled into the APK.
