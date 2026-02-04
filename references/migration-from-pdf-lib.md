# Migrating from pdf-lib to @libpdf/core

This guide shows side-by-side comparisons for migrating from pdf-lib to @libpdf/core.

## Key Differences

| Feature | pdf-lib | @libpdf/core |
|---------|---------|--------------|
| **Malformed PDFs** | Strict parsing, often fails | Lenient with fallback recovery |
| **Incremental saves** | Not supported | `save({ incremental: true })` |
| **Digital signatures** | Not supported | Full PAdES support (B-B, B-T, B-LT, B-LTA) |
| **Text extraction** | Limited | Full with search capabilities |
| **API style** | Promise-based, fluent | Promise-based, similar patterns |
| **TypeScript** | Full support | Full support |
| **Runtime** | Universal | Universal |

## Installation

```bash
# Remove pdf-lib
npm uninstall pdf-lib

# Install @libpdf/core
npm install @libpdf/core
```

---

## Loading PDFs

### pdf-lib
```typescript
import { PDFDocument } from "pdf-lib";

const pdfDoc = await PDFDocument.load(bytes);
const pdfDoc = await PDFDocument.load(bytes, {
  ignoreEncryption: true,
});
```

### @libpdf/core
```typescript
import { PDF } from "@libpdf/core";

const pdf = await PDF.load(bytes);
const pdf = await PDF.load(bytes, {
  credentials: "password", // or { user: "pwd" } or { owner: "pwd" }
});
```

**Notes:**
- `PDFDocument` → `PDF`
- `ignoreEncryption` not needed; use `credentials` for encrypted PDFs
- @libpdf/core handles malformed PDFs automatically

---

## Creating PDFs

### pdf-lib
```typescript
const pdfDoc = await PDFDocument.create();
```

### @libpdf/core
```typescript
const pdf = PDF.create();
```

**Notes:**
- Synchronous in @libpdf/core (no await needed)

---

## Adding Pages

### pdf-lib
```typescript
import { PageSizes } from "pdf-lib";

const page = pdfDoc.addPage();
const page = pdfDoc.addPage(PageSizes.A4);
const page = pdfDoc.addPage([600, 800]);
```

### @libpdf/core
```typescript
const page = pdf.addPage(); // US Letter by default
const page = pdf.addPage({ size: "a4" });
const page = pdf.addPage({ width: 600, height: 800 });
```

**Notes:**
- Options passed as object instead of positional arguments
- Size names: lowercase strings (`"a4"`, `"letter"`, `"legal"`)

---

## Removing Pages

### pdf-lib
```typescript
pdfDoc.removePage(2);
```

### @libpdf/core
```typescript
pdf.removePage(2);
```

**Notes:**
- Same API

---

## Copying Pages

### pdf-lib
```typescript
const [copiedPage] = await pdfDoc.copyPages(otherDoc, [0]);
pdfDoc.addPage(copiedPage);
```

### @libpdf/core
```typescript
const [copiedPage] = await pdf.copyPagesFrom(otherPdf, [0]);
// Page automatically added to pdf
```

**Notes:**
- `copyPages` → `copyPagesFrom`
- Pages automatically added; no need for separate `addPage`

---

## Getting Pages

### pdf-lib
```typescript
const pages = pdfDoc.getPages();
const page = pdfDoc.getPage(0);
const count = pdfDoc.getPageCount();
```

### @libpdf/core
```typescript
const pages = pdf.getPages();
const page = pdf.getPage(0);
const count = pdf.getPageCount();
```

**Notes:**
- Same API

---

## Page Dimensions

### pdf-lib
```typescript
const { width, height } = page.getSize();
page.setSize(600, 800);
```

### @libpdf/core
```typescript
const { width, height } = page.getSize();
page.setSize(600, 800);
```

**Notes:**
- Same API

---

## Drawing Text

### pdf-lib
```typescript
import { rgb } from "pdf-lib";

page.drawText("Hello, World!", {
  x: 50,
  y: 700,
  size: 24,
  color: rgb(0, 0, 0),
});
```

### @libpdf/core
```typescript
import { rgb } from "@libpdf/core";

page.drawText("Hello, World!", {
  x: 50,
  y: 700,
  fontSize: 24, // Changed from "size"
  color: rgb(0, 0, 0),
});
```

**Notes:**
- `size` → `fontSize` (more descriptive)
- Same color helpers: `rgb()`, `cmyk()`, `grayscale()`

---

## Drawing Rectangles

### pdf-lib
```typescript
page.drawRectangle({
  x: 50,
  y: 500,
  width: 200,
  height: 100,
  color: rgb(0.9, 0.9, 0.9),
  borderColor: rgb(0, 0, 0),
  borderWidth: 2,
});
```

### @libpdf/core
```typescript
page.drawRectangle({
  x: 50,
  y: 500,
  width: 200,
  height: 100,
  color: rgb(0.9, 0.9, 0.9),
  borderColor: rgb(0, 0, 0),
  borderWidth: 2,
});
```

**Notes:**
- Same API

---

## Drawing Circles

### pdf-lib
```typescript
page.drawCircle({
  x: 300,
  y: 400,
  size: 50,
  color: rgb(1, 0, 0),
});
```

### @libpdf/core
```typescript
page.drawCircle({
  x: 300,
  y: 400,
  radius: 50, // Changed from "size"
  color: rgb(1, 0, 0),
});
```

**Notes:**
- `size` → `radius` (more accurate)

---

## Drawing Lines

### pdf-lib
```typescript
page.drawLine({
  start: { x: 50, y: 300 },
  end: { x: 250, y: 300 },
  thickness: 3,
  color: rgb(0, 0, 1),
});
```

### @libpdf/core
```typescript
page.drawLine({
  start: { x: 50, y: 300 },
  end: { x: 250, y: 300 },
  thickness: 3,
  color: rgb(0, 0, 1),
});
```

**Notes:**
- Same API

---

## Embedding Fonts

### pdf-lib
```typescript
import { StandardFonts } from "pdf-lib";

const helvetica = await pdfDoc.embedFont(StandardFonts.Helvetica);
const customFont = await pdfDoc.embedFont(fontBytes);
```

### @libpdf/core
```typescript
import { StandardFonts } from "@libpdf/core";

const helvetica = StandardFonts.Helvetica; // No embed needed for standard fonts
const customFont = await pdf.embedFont(fontBytes);
```

**Notes:**
- Standard fonts don't require embedding
- Custom fonts use `embedFont(bytes)`

---

## Embedding Images

### pdf-lib
```typescript
const jpgImage = await pdfDoc.embedJpg(jpgBytes);
const pngImage = await pdfDoc.embedPng(pngBytes);

page.drawImage(jpgImage, {
  x: 100,
  y: 400,
  width: 200,
  height: 150,
});
```

### @libpdf/core
```typescript
const jpgImage = await pdf.embedImage(jpgBytes, "jpeg");
const pngImage = await pdf.embedImage(pngBytes, "png");

page.drawImage(jpgImage, {
  x: 100,
  y: 400,
  width: 200,
  height: 150,
});
```

**Notes:**
- `embedJpg` / `embedPng` → `embedImage(bytes, format)`
- Same `drawImage` API

---

## Form Filling

### pdf-lib
```typescript
const form = pdfDoc.getForm();
const nameField = form.getTextField("name");
nameField.setText("Jane Doe");

const checkbox = form.getCheckBox("agreed");
checkbox.check();
```

### @libpdf/core
```typescript
const form = pdf.getForm();

// Option 1: Batch fill
form.fill({
  name: "Jane Doe",
  agreed: true,
});

// Option 2: Individual fields
const nameField = form.getTextField("name");
nameField.setValue("Jane Doe");

const checkbox = form.getCheckbox("agreed");
checkbox.check();
```

**Notes:**
- `setText` → `setValue` (consistent naming)
- `getCheckBox` → `getCheckbox` (lowercase 'b')
- Batch filling with `form.fill()` is more convenient

---

## Flattening Forms

### pdf-lib
```typescript
const form = pdfDoc.getForm();
form.flatten();
```

### @libpdf/core
```typescript
const form = pdf.getForm();
await form.flatten();
```

**Notes:**
- Async in @libpdf/core (requires await)
- More robust appearance stream generation

---

## Saving PDFs

### pdf-lib
```typescript
const bytes = await pdfDoc.save();
```

### @libpdf/core
```typescript
const bytes = await pdf.save();

// For signed PDFs or preserving signatures
const bytes = await pdf.save({ incremental: true });
```

**Notes:**
- Same basic API
- **Critical:** Use `incremental: true` when modifying signed PDFs
- pdf-lib doesn't support incremental saves

---

## Metadata

### pdf-lib
```typescript
pdfDoc.setTitle("My Document");
pdfDoc.setAuthor("Jane Doe");
const title = pdfDoc.getTitle();
```

### @libpdf/core
```typescript
pdf.setTitle("My Document");
pdf.setAuthor("Jane Doe");
const title = pdf.getTitle();
```

**Notes:**
- Same API

---

## Digital Signatures (New Feature)

### pdf-lib
```typescript
// Not supported
```

### @libpdf/core
```typescript
import { P12Signer } from "@libpdf/core";

const signer = await P12Signer.create(p12Bytes, "password");
const signed = await pdf.sign({
  signer,
  reason: "I approve this document",
  location: "San Francisco, CA",
});
```

**Notes:**
- Major feature addition
- Supports PAdES B-B, B-T, B-LT, B-LTA
- Requires incremental saves (automatic)

---

## Text Extraction (Enhanced)

### pdf-lib
```typescript
// Limited text extraction
const page = pdfDoc.getPage(0);
// No built-in text extraction
```

### @libpdf/core
```typescript
const page = pdf.getPage(0);
const text = page.extractText();

// Search with bounding boxes
const results = page.findText(/invoice #\d+/gi);
for (const result of results) {
  console.log(result.text, result.rect);
}
```

**Notes:**
- Full text extraction with reading order
- Search capabilities with position information

---

## Migration Checklist

- [ ] Replace imports: `pdf-lib` → `@libpdf/core`
- [ ] Update class names: `PDFDocument` → `PDF`
- [ ] Change drawing options: `size` → `fontSize` (text), `radius` (circles)
- [ ] Update font embedding: remove `embedFont()` for standard fonts
- [ ] Update image embedding: `embedJpg`/`embedPng` → `embedImage(bytes, format)`
- [ ] Update form methods: `setText` → `setValue`, `getCheckBox` → `getCheckbox`
- [ ] Add `await` to `form.flatten()`
- [ ] Add `incremental: true` to `save()` for signed PDFs
- [ ] Consider adding digital signatures (new capability)
- [ ] Consider adding text extraction (enhanced capability)
- [ ] Test with malformed PDFs (improved handling)

---

## Common Migration Patterns

### Pattern: Create, modify, save
**pdf-lib:**
```typescript
const pdfDoc = await PDFDocument.create();
const page = pdfDoc.addPage();
page.drawText("Hello");
const bytes = await pdfDoc.save();
```

**@libpdf/core:**
```typescript
const pdf = PDF.create();
const page = pdf.addPage();
page.drawText("Hello", { x: 50, y: 700, fontSize: 12 });
const bytes = await pdf.save();
```

### Pattern: Load, fill form, save
**pdf-lib:**
```typescript
const pdfDoc = await PDFDocument.load(bytes);
const form = pdfDoc.getForm();
const field = form.getTextField("name");
field.setText("Jane");
const filled = await pdfDoc.save();
```

**@libpdf/core:**
```typescript
const pdf = await PDF.load(bytes);
const form = pdf.getForm();
form.fill({ name: "Jane" });
const filled = await pdf.save();
```

### Pattern: Merge documents
**pdf-lib:**
```typescript
const pdfDoc = await PDFDocument.create();
const doc1 = await PDFDocument.load(bytes1);
const doc2 = await PDFDocument.load(bytes2);
const [page1] = await pdfDoc.copyPages(doc1, [0]);
const [page2] = await pdfDoc.copyPages(doc2, [0]);
pdfDoc.addPage(page1);
pdfDoc.addPage(page2);
const merged = await pdfDoc.save();
```

**@libpdf/core:**
```typescript
const merged = await PDF.merge([bytes1, bytes2]);
```

---

## Need Help?

- Check the [examples directory](https://github.com/LibPDF-js/core/tree/main/examples) for complete working examples
- See `api-quick-reference.md` for common patterns
- File issues at https://github.com/LibPDF-js/core/issues
