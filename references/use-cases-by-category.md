# Use Cases by Category

Quick reference for common PDF tasks with @libpdf/core. Each entry shows when to use the feature and provides a code snippet. For complete working examples, see the [examples directory](https://github.com/LibPDF-js/core/tree/main/examples).

## Table of Contents

1. [Creating PDFs](#1-creating-pdfs)
2. [Loading & Parsing](#2-loading--parsing)
3. [Form Filling & Flattening](#3-form-filling--flattening)
4. [Digital Signatures](#4-digital-signatures)
5. [Encryption & Security](#5-encryption--security)
6. [Merge & Split](#6-merge--split)
7. [Text Extraction & Search](#7-text-extraction--search)
8. [Drawing (Text, Images, Shapes)](#8-drawing-text-images-shapes)
9. [Attachments](#9-attachments)
10. [Annotations](#10-annotations)
11. [Page Manipulation](#11-page-manipulation)
12. [Incremental Saves](#12-incremental-saves)

---

## 1. Creating PDFs

### Create empty PDF
**When:** Starting a new document from scratch
**Example:** [`examples/01-basic/create-empty-pdf.ts`](https://github.com/LibPDF-js/core/tree/main/examples/01-basic/create-empty-pdf.ts)

```typescript
import { PDF } from "@libpdf/core";

const pdf = PDF.create();
const page = pdf.addPage({ size: "letter" });
const bytes = await pdf.save();
```

### Create PDF with specific page size
**When:** Need custom dimensions or standard sizes
**Example:** [`examples/02-pages/add-custom-size-page.ts`](https://github.com/LibPDF-js/core/tree/main/examples/02-pages/add-custom-size-page.ts)

```typescript
const page = pdf.addPage({ size: "a4" }); // Standard size
const page = pdf.addPage({ width: 600, height: 800 }); // Custom
```

### Generate invoice
**When:** Programmatic document generation
**Example:** [`examples/04-drawing/draw-text.ts`](https://github.com/LibPDF-js/core/tree/main/examples/04-drawing/draw-text.ts)

```typescript
import { rgb } from "@libpdf/core";

const pdf = PDF.create();
const page = pdf.addPage();

page.drawText("Invoice #12345", { x: 50, y: 750, fontSize: 20 });
page.drawText("Date: 2024-01-15", { x: 50, y: 720, fontSize: 12 });
page.drawRectangle({ x: 50, y: 600, width: 500, height: 100, borderWidth: 1 });
```

---

## 2. Loading & Parsing

### Load existing PDF
**When:** Need to read, modify, or extract from PDF
**Example:** [`examples/01-basic/load-and-inspect.ts`](https://github.com/LibPDF-js/core/tree/main/examples/01-basic/load-and-inspect.ts)

```typescript
const pdf = await PDF.load(bytes);
console.log(`${pdf.getPageCount()} pages`);
```

### Load encrypted PDF
**When:** Working with password-protected documents
**Example:** [`examples/01-basic/load-encrypted.ts`](https://github.com/LibPDF-js/core/tree/main/examples/01-basic/load-encrypted.ts)

```typescript
const pdf = await PDF.load(bytes, { credentials: "password" });
// or specify user/owner password explicitly
const pdf = await PDF.load(bytes, { credentials: { user: "userpass" } });
```

### Handle malformed PDF
**When:** PDF fails to open in other libraries
**Example:** [`examples/11-advanced/handle-malformed-pdf.ts`](https://github.com/LibPDF-js/core/tree/main/examples/11-advanced/handle-malformed-pdf.ts)

```typescript
const pdf = await PDF.load(malformedBytes);
// Automatically falls back to brute-force recovery
```

### Inspect document structure
**When:** Debugging or understanding PDF internals
**Example:** [`examples/11-advanced/read-document-catalog.ts`](https://github.com/LibPDF-js/core/tree/main/examples/11-advanced/read-document-catalog.ts)

```typescript
const catalog = pdf.context.catalog;
console.log("Pages:", catalog.get("Pages"));
console.log("Metadata:", catalog.get("Metadata"));
```

---

## 3. Form Filling & Flattening

### Fill form fields (batch)
**When:** Populating forms programmatically
**Example:** [`examples/03-forms/fill-form.ts`](https://github.com/LibPDF-js/core/tree/main/examples/03-forms/fill-form.ts)

```typescript
const form = pdf.getForm();
form.fill({
  name: "Jane Doe",
  email: "jane@example.com",
  agreed: true,
  country: "USA",
});
const filled = await pdf.save();
```

### Fill form fields (individual)
**When:** Need fine-grained control per field
**Example:** [`examples/03-forms/fill-form.ts`](https://github.com/LibPDF-js/core/tree/main/examples/03-forms/fill-form.ts)

```typescript
const form = pdf.getForm();
const nameField = form.getTextField("name");
nameField.setValue("Jane Doe");

const checkbox = form.getCheckbox("agreed");
checkbox.check();
```

### Flatten form
**When:** Make form non-editable (print-ready)
**Example:** [`examples/03-forms/flatten-form.ts`](https://github.com/LibPDF-js/core/tree/main/examples/03-forms/flatten-form.ts)

```typescript
const form = pdf.getForm();
await form.flatten(); // Bake all fields into page content
const flattened = await pdf.save();
```

### Flatten specific fields
**When:** Lock some fields, keep others editable
**Example:** [`examples/03-forms/flatten-form.ts`](https://github.com/LibPDF-js/core/tree/main/examples/03-forms/flatten-form.ts)

```typescript
await form.flatten({ fields: ["name", "date"] });
```

### Create form fields
**When:** Building interactive forms
**Example:** [`examples/03-forms/create-form-fields.ts`](https://github.com/LibPDF-js/core/tree/main/examples/03-forms/create-form-fields.ts)

```typescript
form.createTextField("username", {
  page: 0,
  rect: { x: 100, y: 500, width: 200, height: 30 },
  value: "Default",
});

form.createCheckbox("subscribe", {
  page: 0,
  rect: { x: 100, y: 450, width: 20, height: 20 },
  checked: false,
});
```

---

## 4. Digital Signatures

### Sign with P12 certificate
**When:** Adding legally binding signatures
**Example:** [`examples/07-signatures/sign-with-p12.ts`](https://github.com/LibPDF-js/core/tree/main/examples/07-signatures/sign-with-p12.ts)

```typescript
import { P12Signer } from "@libpdf/core";

const signer = await P12Signer.create(p12Bytes, "password");
const signed = await pdf.sign({
  signer,
  reason: "I approve this document",
  location: "San Francisco, CA",
  contactInfo: "jane@example.com",
});
```

### Sign with PAdES-LT/LTA (Long-Term Validation)
**When:** Archival signatures, regulatory compliance
**Example:** [`examples/07-signatures/sign-with-long-term-validation.ts`](https://github.com/LibPDF-js/core/tree/main/examples/07-signatures/sign-with-long-term-validation.ts)

```typescript
import { HttpTimestampAuthority } from "@libpdf/core";

const tsa = new HttpTimestampAuthority("https://freetsa.org/tsr");
const signed = await pdf.sign({
  signer,
  level: "pades-lta", // or "pades-lt"
  timestampAuthority: tsa,
});
```

### Sign with CryptoKey (browser)
**When:** Signing in browser with Web Crypto API
**Example:** [`examples/07-signatures/sign-with-cryptokey.ts`](https://github.com/LibPDF-js/core/tree/main/examples/07-signatures/sign-with-cryptokey.ts)

```typescript
import { CryptoKeySigner } from "@libpdf/core";

const signer = await CryptoKeySigner.create(privateKey, certificateBytes);
const signed = await pdf.sign({ signer });
```

### Multiple signatures
**When:** Multi-party approval workflow
**Example:** [`examples/07-signatures/multiple-signatures.ts`](https://github.com/LibPDF-js/core/tree/main/examples/07-signatures/multiple-signatures.ts)

```typescript
const firstSign = await pdf.sign({ signer: signer1 });
const pdf2 = await PDF.load(firstSign);
const secondSign = await pdf2.sign({ signer: signer2 });
```

---

## 5. Encryption & Security

### Encrypt PDF with password
**When:** Protecting sensitive documents
**Example:** [`examples/10-security/set-password.ts`](https://github.com/LibPDF-js/core/tree/main/examples/10-security/set-password.ts)

```typescript
const encrypted = await pdf.save({
  protection: {
    userPassword: "user123", // Required to open
    ownerPassword: "owner456", // Required to change permissions
  },
});
```

### Set permissions
**When:** Allow viewing but restrict editing/printing
**Example:** [`examples/10-security/set-permissions.ts`](https://github.com/LibPDF-js/core/tree/main/examples/10-security/set-permissions.ts)

```typescript
pdf.setProtection({
  userPassword: "view",
  ownerPassword: "admin",
  permissions: {
    printing: "high-resolution", // or "low-resolution" or false
    modifying: false,
    copying: false,
    annotating: true,
  },
});
```

### Check encryption status
**When:** Verify security settings
**Example:** [`examples/10-security/check-encryption.ts`](https://github.com/LibPDF-js/core/tree/main/examples/10-security/check-encryption.ts)

```typescript
const security = pdf.getSecurityInfo();
console.log(security.isEncrypted); // true/false
console.log(security.algorithm); // "aes-256", "aes-128", "rc4", null
console.log(security.permissions); // { printing, modifying, ... }
```

### Remove password
**When:** Decrypting for editing
**Example:** [`examples/10-security/remove-password.ts`](https://github.com/LibPDF-js/core/tree/main/examples/10-security/remove-password.ts)

```typescript
const pdf = await PDF.load(encryptedBytes, { credentials: "password" });
pdf.removeProtection();
const decrypted = await pdf.save();
```

---

## 6. Merge & Split

### Merge PDFs
**When:** Combining multiple documents
**Example:** [`examples/09-merging-and-splitting/merge-pdfs.ts`](https://github.com/LibPDF-js/core/tree/main/examples/09-merging-and-splitting/merge-pdfs.ts)

```typescript
const merged = await PDF.merge([pdf1Bytes, pdf2Bytes, pdf3Bytes]);
```

### Merge with metadata
**When:** Setting properties on merged document
**Example:** [`examples/09-merging-and-splitting/merge-with-metadata.ts`](https://github.com/LibPDF-js/core/tree/main/examples/09-merging-and-splitting/merge-with-metadata.ts)

```typescript
const merged = await PDF.merge([pdf1Bytes, pdf2Bytes], {
  title: "Combined Report",
  author: "Jane Doe",
  subject: "Annual Summary",
});
```

### Extract pages
**When:** Creating subset of document
**Example:** [`examples/09-merging-and-splitting/extract-pages.ts`](https://github.com/LibPDF-js/core/tree/main/examples/09-merging-and-splitting/extract-pages.ts)

```typescript
const subset = await pdf.extractPages([0, 2, 5]); // Pages 1, 3, 6
```

### Copy pages between PDFs
**When:** Building new PDF from multiple sources
**Example:** [`examples/02-pages/copy-pages.ts`](https://github.com/LibPDF-js/core/tree/main/examples/02-pages/copy-pages.ts)

```typescript
const targetPdf = PDF.create();
const [page1, page2] = await targetPdf.copyPagesFrom(sourcePdf, [0, 1]);
```

### Split into individual pages
**When:** One PDF per page
**Example:** [`examples/09-merging-and-splitting/split-pages.ts`](https://github.com/LibPDF-js/core/tree/main/examples/09-merging-and-splitting/split-pages.ts)

```typescript
for (let i = 0; i < pdf.getPageCount(); i++) {
  const singlePage = await pdf.extractPages([i]);
  await writeFile(`page-${i + 1}.pdf`, singlePage);
}
```

---

## 7. Text Extraction & Search

### Extract all text from page
**When:** Indexing, analysis, content extraction
**Example:** [`examples/12-text-extraction/extract-text.ts`](https://github.com/LibPDF-js/core/tree/main/examples/12-text-extraction/extract-text.ts)

```typescript
const page = pdf.getPage(0);
const text = page.extractText();
console.log(text);
```

### Search text with regex
**When:** Finding specific patterns (invoice numbers, dates)
**Example:** [`examples/12-text-extraction/search-text.ts`](https://github.com/LibPDF-js/core/tree/main/examples/12-text-extraction/search-text.ts)

```typescript
const results = page.findText(/invoice #\d+/gi);
for (const result of results) {
  console.log(result.text); // Matched text
  console.log(result.rect); // { x, y, width, height }
}
```

### Highlight search results
**When:** Visual markup of found text
**Example:** [`examples/12-text-extraction/highlight-template-tags.ts`](https://github.com/LibPDF-js/core/tree/main/examples/12-text-extraction/highlight-template-tags.ts)

```typescript
const results = page.findText(/{{[^}]+}}/g);
for (const result of results) {
  page.addAnnotation({
    type: "highlight",
    rect: result.rect,
    color: rgb(1, 1, 0),
  });
}
```

### Extract text from all pages
**When:** Full document text extraction
**Example:** [`examples/12-text-extraction/extract-text.ts`](https://github.com/LibPDF-js/core/tree/main/examples/12-text-extraction/extract-text.ts)

```typescript
const allText = pdf.getPages().map((page) => page.extractText()).join("\n\n");
```

---

## 8. Drawing (Text, Images, Shapes)

### Draw text
**When:** Adding content to pages
**Example:** [`examples/04-drawing/draw-text.ts`](https://github.com/LibPDF-js/core/tree/main/examples/04-drawing/draw-text.ts)

```typescript
import { rgb } from "@libpdf/core";

page.drawText("Hello, World!", {
  x: 50,
  y: 700,
  fontSize: 24,
  color: rgb(0, 0, 0),
});
```

### Draw with custom font
**When:** Brand consistency, special characters
**Example:** [`examples/05-images-and-fonts/embed-font.ts`](https://github.com/LibPDF-js/core/tree/main/examples/05-images-and-fonts/embed-font.ts)

```typescript
const font = await pdf.embedFont(ttfBytes);
page.drawText("Custom font", {
  x: 50,
  y: 600,
  font,
  fontSize: 18,
});
```

### Draw shapes
**When:** Visual elements, diagrams, highlights
**Example:** [`examples/04-drawing/draw-shapes.ts`](https://github.com/LibPDF-js/core/tree/main/examples/04-drawing/draw-shapes.ts)

```typescript
// Rectangle
page.drawRectangle({
  x: 50,
  y: 500,
  width: 200,
  height: 100,
  color: rgb(0.9, 0.9, 0.9),
  borderColor: rgb(0, 0, 0),
  borderWidth: 2,
});

// Circle
page.drawCircle({
  x: 300,
  y: 400,
  radius: 50,
  color: rgb(1, 0, 0),
});

// Line
page.drawLine({
  start: { x: 50, y: 300 },
  end: { x: 250, y: 300 },
  thickness: 3,
});
```

### Embed and draw images
**When:** Logos, photos, diagrams
**Example:** [`examples/05-images-and-fonts/embed-image.ts`](https://github.com/LibPDF-js/core/tree/main/examples/05-images-and-fonts/embed-image.ts)

```typescript
const image = await pdf.embedImage(jpegBytes, "jpeg");
page.drawImage(image, {
  x: 100,
  y: 400,
  width: 200,
  height: 150,
});
```

### Draw SVG path
**When:** Complex vector graphics
**Example:** [`examples/04-drawing/draw-svg-path.ts`](https://github.com/LibPDF-js/core/tree/main/examples/04-drawing/draw-svg-path.ts)

```typescript
page.drawSvgPath("M 0 0 L 100 100 L 200 0 Z", {
  x: 100,
  y: 200,
  color: rgb(0, 0.5, 0),
});
```

### Rotated text
**When:** Diagonal labels, watermarks
**Example:** [`examples/04-drawing/draw-text.ts`](https://github.com/LibPDF-js/core/tree/main/examples/04-drawing/draw-text.ts)

```typescript
page.drawText("Rotated", {
  x: 300,
  y: 400,
  fontSize: 24,
  rotate: { angle: 45 }, // degrees
});
```

---

## 9. Attachments

### Get attachments
**When:** Extracting embedded files (invoices, receipts)
**Example:** [`examples/08-attachments/extract-attachments.ts`](https://github.com/LibPDF-js/core/tree/main/examples/08-attachments/extract-attachments.ts)

```typescript
const attachments = pdf.getAttachments();
for (const attachment of attachments) {
  console.log(attachment.name);
  const bytes = await attachment.getData();
  await writeFile(attachment.name, bytes);
}
```

### Add attachment
**When:** Embedding related files
**Example:** [`examples/08-attachments/add-attachment.ts`](https://github.com/LibPDF-js/core/tree/main/examples/08-attachments/add-attachment.ts)

```typescript
pdf.addAttachment(fileBytes, {
  name: "invoice.pdf",
  description: "Original invoice",
  mimeType: "application/pdf",
});
```

### Remove attachment
**When:** Cleaning up embedded files
**Example:** [`examples/08-attachments/remove-attachment.ts`](https://github.com/LibPDF-js/core/tree/main/examples/08-attachments/remove-attachment.ts)

```typescript
const attachments = pdf.getAttachments();
pdf.removeAttachment(attachments[0]);
```

---

## 10. Annotations

### Add text annotation
**When:** Comments, notes, review marks
**Example:** [`examples/11-advanced/add-annotations.ts`](https://github.com/LibPDF-js/core/tree/main/examples/11-advanced/add-annotations.ts)

```typescript
page.addAnnotation({
  type: "text",
  rect: { x: 100, y: 500, width: 20, height: 20 },
  contents: "Review this section",
  color: rgb(1, 1, 0),
});
```

### Add link annotation
**When:** Hyperlinks to URLs or internal pages
**Example:** [`examples/11-advanced/add-annotations.ts`](https://github.com/LibPDF-js/core/tree/main/examples/11-advanced/add-annotations.ts)

```typescript
page.addAnnotation({
  type: "link",
  rect: { x: 100, y: 400, width: 200, height: 30 },
  url: "https://example.com",
});
```

### Highlight text
**When:** Emphasizing content
**Example:** [`examples/12-text-extraction/highlight-template-tags.ts`](https://github.com/LibPDF-js/core/tree/main/examples/12-text-extraction/highlight-template-tags.ts)

```typescript
page.addAnnotation({
  type: "highlight",
  rect: { x: 100, y: 500, width: 200, height: 20 },
  color: rgb(1, 1, 0),
});
```

### Flatten annotations
**When:** Making annotations permanent
**Example:** [`examples/11-advanced/flatten-annotations.ts`](https://github.com/LibPDF-js/core/tree/main/examples/11-advanced/flatten-annotations.ts)

```typescript
await pdf.flattenAnnotations();
const flattened = await pdf.save();
```

---

## 11. Page Manipulation

### Add blank page
**When:** Inserting pages
**Example:** [`examples/02-pages/add-page.ts`](https://github.com/LibPDF-js/core/tree/main/examples/02-pages/add-page.ts)

```typescript
const page = pdf.addPage({ size: "letter" });
```

### Remove page
**When:** Deleting unwanted pages
**Example:** [`examples/02-pages/remove-page.ts`](https://github.com/LibPDF-js/core/tree/main/examples/02-pages/remove-page.ts)

```typescript
pdf.removePage(2); // Remove page 3 (0-indexed)
```

### Rotate page
**When:** Fixing orientation
**Example:** [`examples/02-pages/rotate-page.ts`](https://github.com/LibPDF-js/core/tree/main/examples/02-pages/rotate-page.ts)

```typescript
const page = pdf.getPage(0);
page.setRotation(90); // 0, 90, 180, 270
```

### Get page dimensions
**When:** Positioning content
**Example:** [`examples/02-pages/get-dimensions.ts`](https://github.com/LibPDF-js/core/tree/main/examples/02-pages/get-dimensions.ts)

```typescript
const { width, height } = page.getSize();
const rotation = page.getRotation();
```

### Embed page as overlay
**When:** Watermarks, letterhead
**Example:** [`examples/11-advanced/embed-page-as-xobject.ts`](https://github.com/LibPDF-js/core/tree/main/examples/11-advanced/embed-page-as-xobject.ts)

```typescript
const embeddedPage = await pdf.embedPage(sourcePdf, 0);
page.drawField(embeddedPage, {
  x: 0,
  y: 0,
  width: page.getSize().width,
  height: page.getSize().height,
  opacity: 0.3,
});
```

---

## 12. Incremental Saves

### Save with incremental update
**When:** Preserving digital signatures
**Example:** [`examples/01-basic/save-incremental.ts`](https://github.com/LibPDF-js/core/tree/main/examples/01-basic/save-incremental.ts)

```typescript
const modified = await pdf.save({ incremental: true });
```

**Critical:** Always use `incremental: true` when:
- Modifying signed PDFs
- Preserving existing signatures
- Appending changes without rewriting

### Detect if incremental save needed
**When:** Checking for signatures
**Example:** [`examples/11-advanced/detect-signatures.ts`](https://github.com/LibPDF-js/core/tree/main/examples/11-advanced/detect-signatures.ts)

```typescript
const hasSignatures = pdf.context.catalog.has("Perms");
const saveOptions = hasSignatures ? { incremental: true } : {};
const bytes = await pdf.save(saveOptions);
```

---

## Advanced Patterns

### Low-level object access
**When:** Direct PDF structure manipulation
**Example:** [`examples/11-advanced/low-level-object-access.ts`](https://github.com/LibPDF-js/core/tree/main/examples/11-advanced/low-level-object-access.ts)

```typescript
import { PdfDict, PdfName } from "@libpdf/core";

const catalog = pdf.context.catalog;
const pagesDict = catalog.get("Pages") as PdfDict;
const count = pagesDict.get("Count");
```

### Flatten everything (forms + annotations + layers)
**When:** Preparing for archival or signing
**Example:** [`examples/11-advanced/flatten-all.ts`](https://github.com/LibPDF-js/core/tree/main/examples/11-advanced/flatten-all.ts)

```typescript
await pdf.flattenAll();
const archival = await pdf.save();
```

### Inspect content streams
**When:** Debugging drawing operations
**Example:** [`examples/11-advanced/content-stream-inspection.ts`](https://github.com/LibPDF-js/core/tree/main/examples/11-advanced/content-stream-inspection.ts)

```typescript
const page = pdf.getPage(0);
const contentStream = page.getContentStream();
console.log(contentStream.toString());
```

---

For complete working implementations of these patterns, browse the [examples directory](https://github.com/LibPDF-js/core/tree/main/examples) organized by category.
