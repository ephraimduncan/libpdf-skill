# Migrating from PDFKit to @libpdf/core

This guide explains when and how to migrate from PDFKit to @libpdf/core.

## Key Differences

| Feature | PDFKit | @libpdf/core |
|---------|--------|--------------|
| **Purpose** | Generation only | Parsing + modification + generation |
| **Parsing** | ✗ None | ✓ Full with lenient handling |
| **Modification** | ✗ No (create only) | ✓ Full (forms, pages, signing) |
| **Form filling** | ✗ Create only | ✓ Fill and flatten |
| **Digital signatures** | ✗ | ✓ PAdES B-B, B-T, B-LT, B-LTA |
| **Text extraction** | ✗ | ✓ With search |
| **Output** | Stream-based | Promise → Uint8Array |
| **API style** | Imperative (method chaining) | Object-oriented |
| **TypeScript** | .d.ts files | First-class TypeScript |

## When to Migrate

### Migrate when you need:
- **Parsing existing PDFs** (PDFKit can't read PDFs)
- **Modifying existing PDFs** (forms, pages, annotations)
- **Filling forms** (not just creating them)
- **Digital signatures**
- **Text extraction**
- **TypeScript-first development**

### Keep PDFKit if:
- You only generate PDFs from scratch
- Your workflow is heavily stream-based
- You're satisfied with PDFKit's generation features

---

## Installation

```bash
# Remove PDFKit
npm uninstall pdfkit

# Install @libpdf/core
npm install @libpdf/core
```

---

## Creating PDFs

### PDFKit
```javascript
import PDFDocument from "pdfkit";
import fs from "fs";

const doc = new PDFDocument();
doc.pipe(fs.createWriteStream("output.pdf"));

doc.text("Hello, World!", 100, 100);
doc.end();
```

### @libpdf/core
```typescript
import { PDF } from "@libpdf/core";
import { writeFile } from "fs/promises";

const pdf = PDF.create();
const page = pdf.addPage();

page.drawText("Hello, World!", {
  x: 100,
  y: 700, // Y-axis origin at bottom (PDF standard)
  fontSize: 12,
});

const bytes = await pdf.save();
await writeFile("output.pdf", bytes);
```

**Key differences:**
- PDFKit uses streams; @libpdf/core uses Promises
- PDFKit Y-axis from top; @libpdf/core from bottom (PDF standard)
- @libpdf/core requires explicit page creation
- @libpdf/core uses option objects for parameters

---

## Adding Pages

### PDFKit
```javascript
doc.addPage({ size: "A4", margin: 50 });
```

### @libpdf/core
```typescript
const page = pdf.addPage({ size: "a4" });
```

**Notes:**
- Lowercase size names in @libpdf/core
- No built-in margins in @libpdf/core (control with x/y coordinates)

---

## Drawing Text

### PDFKit
```javascript
doc.fontSize(24)
   .fillColor("blue")
   .text("Styled text", 100, 100);

doc.text("Multiline text", {
  width: 400,
  align: "center",
});
```

### @libpdf/core
```typescript
import { rgb } from "@libpdf/core";

page.drawText("Styled text", {
  x: 100,
  y: 700,
  fontSize: 24,
  color: rgb(0, 0, 1),
});

// Multiline with layoutText helper
import { layoutText } from "@libpdf/core";

const lines = layoutText("Multiline text", {
  maxWidth: 400,
  fontSize: 12,
});

for (const line of lines) {
  page.drawText(line.text, {
    x: 100,
    y: yPosition,
    fontSize: 12,
  });
  yPosition -= 15; // Line height
}
```

**Notes:**
- PDFKit uses method chaining; @libpdf/core uses option objects
- PDFKit has built-in text wrapping; @libpdf/core requires layout helpers
- Different Y-axis conventions

---

## Drawing Shapes

### PDFKit
```javascript
// Rectangle
doc.rect(100, 100, 200, 150)
   .fillColor("gray")
   .fill();

// Circle
doc.circle(300, 300, 50)
   .fillColor("red")
   .fill();

// Line
doc.moveTo(50, 200)
   .lineTo(250, 200)
   .stroke();
```

### @libpdf/core
```typescript
import { rgb } from "@libpdf/core";

// Rectangle
page.drawRectangle({
  x: 100,
  y: 500,
  width: 200,
  height: 150,
  color: rgb(0.5, 0.5, 0.5),
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
  thickness: 1,
  color: rgb(0, 0, 0),
});
```

**Notes:**
- PDFKit uses immediate-mode drawing with state
- @libpdf/core uses declarative option objects
- Different Y-axis (top vs bottom origin)

---

## Images

### PDFKit
```javascript
doc.image("image.jpg", 100, 100, { width: 200 });
```

### @libpdf/core
```typescript
import { readFile } from "fs/promises";

const imageBytes = await readFile("image.jpg");
const image = await pdf.embedImage(imageBytes, "jpeg");

page.drawImage(image, {
  x: 100,
  y: 400,
  width: 200,
  height: 150, // Must specify height
});
```

**Notes:**
- @libpdf/core requires embedding before drawing
- Must specify both width and height

---

## Fonts

### PDFKit
```javascript
doc.font("Helvetica-Bold")
   .text("Bold text");

doc.registerFont("CustomFont", "path/to/font.ttf");
doc.font("CustomFont")
   .text("Custom font text");
```

### @libpdf/core
```typescript
import { StandardFonts } from "@libpdf/core";
import { readFile } from "fs/promises";

// Standard font
page.drawText("Bold text", {
  x: 100,
  y: 700,
  font: StandardFonts.HelveticaBold,
  fontSize: 12,
});

// Custom font
const fontBytes = await readFile("path/to/font.ttf");
const customFont = await pdf.embedFont(fontBytes);

page.drawText("Custom font text", {
  x: 100,
  y: 650,
  font: customFont,
  fontSize: 12,
});
```

**Notes:**
- No global font state in @libpdf/core
- Custom fonts must be embedded
- Font specified per drawText call

---

## Output Handling

### PDFKit (Stream)
```javascript
const doc = new PDFDocument();

// Option 1: Pipe to file
doc.pipe(fs.createWriteStream("output.pdf"));
doc.text("Hello");
doc.end();

// Option 2: Collect to buffer
const chunks = [];
doc.on("data", (chunk) => chunks.push(chunk));
doc.on("end", () => {
  const pdfBuffer = Buffer.concat(chunks);
  // Use buffer
});
doc.text("Hello");
doc.end();
```

### @libpdf/core (Promise)
```typescript
const pdf = PDF.create();
const page = pdf.addPage();
page.drawText("Hello", { x: 100, y: 700, fontSize: 12 });

const bytes = await pdf.save(); // Uint8Array
await writeFile("output.pdf", bytes);
```

**Notes:**
- PDFKit streams data as it's created
- @libpdf/core returns complete Uint8Array
- @libpdf/core is simpler for most use cases
- PDFKit better for very large documents (streaming)

---

## Parsing PDFs (Not Supported in PDFKit)

PDFKit **cannot read or modify existing PDFs**. This is a major reason to migrate:

```typescript
import { PDF } from "@libpdf/core";

// Load existing PDF
const pdf = await PDF.load(existingBytes);

// Get information
console.log(`${pdf.getPageCount()} pages`);
console.log(`Title: ${pdf.getTitle()}`);

// Modify
const page = pdf.getPage(0);
page.drawText("Added text", { x: 50, y: 700, fontSize: 12 });

// Save
const modified = await pdf.save();
```

---

## Form Filling (Not Supported in PDFKit)

PDFKit can create form fields but **cannot fill existing forms**:

```typescript
import { PDF } from "@libpdf/core";

const pdf = await PDF.load(formBytes);
const form = pdf.getForm();

// Fill all fields
form.fill({
  name: "Jane Doe",
  email: "jane@example.com",
  agreed: true,
});

const filled = await pdf.save();
```

---

## Digital Signatures (Not Supported in PDFKit)

```typescript
import { P12Signer } from "@libpdf/core";

const pdf = await PDF.load(bytes);
const signer = await P12Signer.create(p12Bytes, "password");

const signed = await pdf.sign({
  signer,
  reason: "I approve this document",
  location: "San Francisco, CA",
});
```

---

## Merging PDFs (Not Supported in PDFKit)

```typescript
const merged = await PDF.merge([pdf1Bytes, pdf2Bytes, pdf3Bytes]);
```

---

## Text Extraction (Not Supported in PDFKit)

```typescript
const pdf = await PDF.load(bytes);
const page = pdf.getPage(0);

// Extract all text
const text = page.extractText();

// Search with regex
const results = page.findText(/invoice #\d+/gi);
```

---

## Migration Patterns

### Pattern: Generate invoice

**PDFKit:**
```javascript
const doc = new PDFDocument();
doc.pipe(fs.createWriteStream("invoice.pdf"));

doc.fontSize(20).text("Invoice", 50, 50);
doc.fontSize(12).text(`Date: ${new Date().toLocaleDateString()}`, 50, 100);
doc.text(`Amount: $1,234.56`, 50, 120);

doc.end();
```

**@libpdf/core:**
```typescript
const pdf = PDF.create();
const page = pdf.addPage();

page.drawText("Invoice", { x: 50, y: 750, fontSize: 20 });
page.drawText(`Date: ${new Date().toLocaleDateString()}`, { x: 50, y: 700, fontSize: 12 });
page.drawText(`Amount: $1,234.56`, { x: 50, y: 680, fontSize: 12 });

const bytes = await pdf.save();
await writeFile("invoice.pdf", bytes);
```

### Pattern: Certificate with background

**PDFKit:**
```javascript
const doc = new PDFDocument();
doc.pipe(fs.createWriteStream("cert.pdf"));

doc.image("background.jpg", 0, 0, { width: 612, height: 792 });
doc.fontSize(32).fillColor("blue").text("Certificate", 200, 300);

doc.end();
```

**@libpdf/core:**
```typescript
import { readFile } from "fs/promises";
import { rgb } from "@libpdf/core";

const pdf = PDF.create();
const page = pdf.addPage({ size: "letter" });

const bgBytes = await readFile("background.jpg");
const bgImage = await pdf.embedImage(bgBytes, "jpeg");
page.drawImage(bgImage, { x: 0, y: 0, width: 612, height: 792 });

page.drawText("Certificate", {
  x: 200,
  y: 492,
  fontSize: 32,
  color: rgb(0, 0, 1),
});

const bytes = await pdf.save();
await writeFile("cert.pdf", bytes);
```

---

## Y-Axis Coordinate Conversion

**Critical difference:** PDFKit Y-axis starts at **top**; @libpdf/core starts at **bottom** (PDF standard).

**Conversion:**
```typescript
// PDFKit Y (from top) to @libpdf/core Y (from bottom)
const pageHeight = 792; // Letter size height
const pdfkitY = 100;
const libpdfY = pageHeight - pdfkitY; // 692

// Example
// PDFKit: doc.text("Hello", 50, 100) - 100pt from top
// @libpdf/core: page.drawText("Hello", { x: 50, y: 692 }) - 100pt from top
```

---

## Migration Checklist

- [ ] Replace `new PDFDocument()` with `PDF.create()`
- [ ] Replace `doc.pipe()` with `await pdf.save()` and `writeFile()`
- [ ] Replace method chaining with option objects
- [ ] Convert Y coordinates (top → bottom origin)
- [ ] Replace `doc.text()` with `page.drawText()`
- [ ] Replace `doc.fontSize().fillColor()` with `fontSize` and `color` options
- [ ] Replace `doc.rect/circle` with `page.drawRectangle/Circle`
- [ ] Replace `doc.image()` with `embedImage()` + `drawImage()`
- [ ] Replace `doc.font()` with `font` option in `drawText()`
- [ ] Test new capabilities: parsing, form filling, signatures
- [ ] Update output handling from streams to Promises

---

## Advantages After Migration

- ✓ **Parse existing PDFs** (PDFKit can't)
- ✓ **Modify existing PDFs** (add to existing documents)
- ✓ **Fill forms** programmatically
- ✓ **Digital signatures** with PAdES support
- ✓ **Text extraction** and search
- ✓ **Merge PDFs**
- ✓ **Better TypeScript** support
- ✓ **Lenient parsing** (opens malformed PDFs)
- ✓ **Universal runtime** (Node.js, Bun, browsers)

---

## Need Help?

- Check the [examples directory](https://github.com/LibPDF-js/core/tree/main/examples) for complete working examples
- See `api-quick-reference.md` for common patterns
- File issues at https://github.com/LibPDF-js/core/issues
