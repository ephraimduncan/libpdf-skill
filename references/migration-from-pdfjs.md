# Migrating from PDF.js to @libpdf/core

This guide explains when and how to migrate from Mozilla's PDF.js to @libpdf/core.

## Understanding the Difference

**PDF.js** and **@libpdf/core** are often **complementary**, not competing:

| Library | Purpose | Strengths |
|---------|---------|-----------|
| **PDF.js** | Rendering and viewing | Best-in-class browser rendering, annotation editing UI |
| **@libpdf/core** | Parsing, modification, generation | Best parsing (malformed PDFs), modification, signatures, forms |

## When to Migrate vs Use Both

### Migrate to @libpdf/core when:
- You need **server-side PDF processing** (Node.js/Bun without browser)
- You need to **modify or generate** PDFs (not just render them)
- You need **form filling**, **digital signatures**, or **encryption**
- You need **text extraction** without rendering
- PDF.js parsing fails on malformed documents

### Use both when:
- **PDF.js for rendering** (display to users)
- **@libpdf/core for editing** (fill forms, sign, merge)

**Example workflow:**
1. User uploads PDF → @libpdf/core parses and fills form
2. Modified PDF → PDF.js renders for preview
3. User approves → @libpdf/core signs and saves

## Key Differences

| Feature | PDF.js | @libpdf/core |
|---------|--------|--------------|
| **Rendering** | ✓ Best | ✗ Use PDF.js |
| **Parsing** | ✓ Good | ✓ Better (lenient) |
| **Text extraction** | ✓ Best | ✓ Good |
| **Modification** | Limited annotations | ✓ Full (forms, pages, content) |
| **Generation** | ✗ | ✓ |
| **Digital signatures** | Display only | ✓ Sign and verify (planned) |
| **Form filling** | Display only | ✓ |
| **Server-side** | Possible but heavy | ✓ Optimized |
| **Browser** | ✓ Optimized | ✓ Supported |

---

## Installation

```bash
# Keep PDF.js for rendering (if needed)
npm install pdfjs-dist

# Add @libpdf/core for modification
npm install @libpdf/core
```

---

## Loading PDFs

### PDF.js
```typescript
import * as pdfjsLib from "pdfjs-dist";

const loadingTask = pdfjsLib.getDocument(bytes);
const pdfDocument = await loadingTask.promise;
```

### @libpdf/core
```typescript
import { PDF } from "@libpdf/core";

const pdf = await PDF.load(bytes);
const pdf = await PDF.load(bytes, { credentials: "password" });
```

**Notes:**
- Simpler API in @libpdf/core
- @libpdf/core handles malformed PDFs more gracefully
- Both support encrypted PDFs

---

## Getting Page Count

### PDF.js
```typescript
const numPages = pdfDocument.numPages;
```

### @libpdf/core
```typescript
const numPages = pdf.getPageCount();
```

---

## Accessing Pages

### PDF.js
```typescript
const page = await pdfDocument.getPage(1); // 1-indexed
```

### @libpdf/core
```typescript
const page = pdf.getPage(0); // 0-indexed
```

**Notes:**
- **PDF.js pages are 1-indexed**
- **@libpdf/core pages are 0-indexed** (standard array indexing)

---

## Text Extraction

### PDF.js
```typescript
const page = await pdfDocument.getPage(1);
const textContent = await page.getTextContent();

let text = "";
for (const item of textContent.items) {
  text += item.str;
}
```

### @libpdf/core
```typescript
const page = pdf.getPage(0);
const text = page.extractText();
```

**Notes:**
- @libpdf/core provides simpler API
- PDF.js provides more granular position data
- Both preserve reading order

---

## Searching Text

### PDF.js
```typescript
// Manual search through textContent items
const textContent = await page.getTextContent();
for (const item of textContent.items) {
  if (/pattern/.test(item.str)) {
    // Found match
  }
}
```

### @libpdf/core
```typescript
const results = page.findText(/pattern/gi);
for (const result of results) {
  console.log(result.text); // Matched text
  console.log(result.rect); // Bounding box
}
```

**Notes:**
- @libpdf/core provides built-in search with bounding boxes

---

## Metadata

### PDF.js
```typescript
const metadata = await pdfDocument.getMetadata();
console.log(metadata.info.Title);
console.log(metadata.info.Author);
```

### @libpdf/core
```typescript
const title = pdf.getTitle();
const author = pdf.getAuthor();
```

**Notes:**
- @libpdf/core provides individual getter methods
- Both provide similar metadata access

---

## Modification (Not Supported in PDF.js)

PDF.js is **read-only** (except limited annotation editing). For modification, use @libpdf/core:

### Add pages
```typescript
const pdf = await PDF.load(bytes);
pdf.addPage({ size: "letter" });
const modified = await pdf.save();
```

### Fill forms
```typescript
const form = pdf.getForm();
form.fill({
  name: "Jane Doe",
  email: "jane@example.com",
});
const filled = await pdf.save();
```

### Sign documents
```typescript
import { P12Signer } from "@libpdf/core";

const signer = await P12Signer.create(p12Bytes, "password");
const signed = await pdf.sign({ signer });
```

### Merge PDFs
```typescript
const merged = await PDF.merge([pdf1Bytes, pdf2Bytes]);
```

---

## Server-Side Processing

### PDF.js (Heavy)
```typescript
// Requires canvas polyfill and large dependencies
import * as pdfjsLib from "pdfjs-dist/legacy/build/pdf.mjs";

const pdfDocument = await pdfjsLib.getDocument(bytes).promise;
```

### @libpdf/core (Optimized)
```typescript
import { PDF } from "@libpdf/core";

const pdf = await PDF.load(bytes);
// No canvas or DOM dependencies
```

**Notes:**
- PDF.js designed for browsers; server use is heavy
- @libpdf/core optimized for server (Node.js/Bun)
- @libpdf/core has no rendering dependencies

---

## Browser Usage

Both libraries work in browsers:

### PDF.js
```typescript
// Optimized for rendering
const loadingTask = pdfjsLib.getDocument(url);
const pdfDocument = await loadingTask.promise;
const page = await pdfDocument.getPage(1);

const canvas = document.getElementById("pdf-canvas");
const context = canvas.getContext("2d");
const viewport = page.getViewport({ scale: 1.5 });

canvas.height = viewport.height;
canvas.width = viewport.width;

await page.render({ canvasContext: context, viewport }).promise;
```

### @libpdf/core
```typescript
// Optimized for modification
const pdf = await PDF.load(bytes);
const form = pdf.getForm();
form.fill({ name: "Jane" });
const modified = await pdf.save();
```

**Recommendation:** Use PDF.js for rendering, @libpdf/core for editing.

---

## Combined Workflow Example

Using both libraries together for a complete solution:

```typescript
import * as pdfjsLib from "pdfjs-dist";
import { PDF } from "@libpdf/core";

// 1. Load PDF for modification
const pdf = await PDF.load(originalBytes);

// 2. Fill form with @libpdf/core
const form = pdf.getForm();
form.fill({
  name: "Jane Doe",
  date: new Date().toISOString(),
});

// 3. Save modified PDF
const modifiedBytes = await pdf.save();

// 4. Render preview with PDF.js
const pdfDocument = await pdfjsLib.getDocument(modifiedBytes).promise;
const page = await pdfDocument.getPage(1);
const viewport = page.getViewport({ scale: 1.0 });

const canvas = document.getElementById("preview");
const context = canvas.getContext("2d");
canvas.height = viewport.height;
canvas.width = viewport.width;

await page.render({ canvasContext: context, viewport }).promise;

// 5. Sign with @libpdf/core
const signer = await P12Signer.create(p12Bytes, "password");
const signed = await pdf.sign({ signer });

// 6. Final render with PDF.js
const finalDocument = await pdfjsLib.getDocument(signed).promise;
// ... render signed PDF
```

---

## Migration Decision Matrix

| Your Use Case | Recommendation |
|---------------|----------------|
| **Only need to display PDFs** | Keep PDF.js |
| **Need to edit PDFs (server-side)** | Migrate to @libpdf/core |
| **Need to edit PDFs (browser)** | Use both: @libpdf/core for editing, PDF.js for preview |
| **Text extraction only** | Either works; @libpdf/core simpler API |
| **Form filling** | @libpdf/core (PDF.js is display-only) |
| **Digital signatures** | @libpdf/core (PDF.js can't sign) |
| **Malformed PDFs** | @libpdf/core (more lenient) |
| **PDF generation** | @libpdf/core (PDF.js can't generate) |

---

## Common Questions

### Can @libpdf/core render PDFs?
No. Use PDF.js for rendering. @libpdf/core focuses on parsing, modification, and generation.

### Can I use both in the same project?
Yes! They complement each other. Use @libpdf/core for editing and PDF.js for rendering.

### Which is better for text extraction?
Both work well. PDF.js has more granular position data; @libpdf/core has simpler API with search.

### Why choose @libpdf/core over PDF.js?
When you need modification (forms, signatures, merging), generation, or better malformed PDF handling.

---

## Need Help?

- Check the [examples directory](https://github.com/LibPDF-js/core/tree/main/examples) for complete @libpdf/core examples
- See `api-quick-reference.md` for common patterns
- File issues at https://github.com/LibPDF-js/core/issues
