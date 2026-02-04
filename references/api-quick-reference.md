# API Quick Reference

Common patterns for @libpdf/core (v0.2.5). For complete examples, see `/examples/` directory.

## Table of Contents

- [Loading & Creating](#loading--creating)
- [Pages](#pages)
- [Forms](#forms)
- [Drawing](#drawing)
- [Text Operations](#text-operations)
- [Images & Fonts](#images--fonts)
- [Digital Signatures](#digital-signatures)
- [Encryption](#encryption)
- [Merge & Split](#merge--split)
- [Metadata](#metadata)
- [Attachments](#attachments)

---

## Loading & Creating

### Load existing PDF
```typescript
import { PDF } from "@libpdf/core";

const pdf = await PDF.load(bytes);
const pdf = await PDF.load(bytes, { credentials: "password" });
```

### Create new PDF
```typescript
const pdf = PDF.create();
```

### Save PDF
```typescript
const output = await pdf.save();
const output = await pdf.save({ incremental: true }); // Preserves signatures
```

---

## Pages

### Get pages
```typescript
const pages = pdf.getPages();
const page = pdf.getPage(0); // Zero-indexed
const count = pdf.getPageCount();
```

### Add pages
```typescript
const page = pdf.addPage(); // Default US Letter size
const page = pdf.addPage({ size: "a4" });
const page = pdf.addPage({ width: 600, height: 800 });
```

### Remove/copy pages
```typescript
pdf.removePage(2);
pdf.copyPages([0, 2, 5]); // Returns new pages in same PDF
```

### Get page dimensions
```typescript
const { width, height } = page.getSize();
const rotation = page.getRotation(); // 0, 90, 180, 270
```

---

## Forms

### Access form
```typescript
const form = pdf.getForm();
const hasForm = pdf.hasForm();
```

### Get fields
```typescript
const fields = form.getFields();
const field = form.getField("fieldName");
const textField = form.getTextField("name");
const checkbox = form.getCheckbox("agreed");
```

### Fill fields (batch)
```typescript
form.fill({
  name: "Jane Doe",
  email: "jane@example.com",
  agreed: true,
  country: "USA",
});
```

### Fill fields (individual)
```typescript
const textField = form.getTextField("name");
textField.setValue("Jane Doe");

const checkbox = form.getCheckbox("agreed");
checkbox.check();

const dropdown = form.getDropdown("country");
dropdown.setValue("USA");
```

### Flatten form
```typescript
await form.flatten(); // Bake fields into page content
await form.flatten({ fields: ["name", "email"] }); // Flatten specific fields
```

### Create fields
```typescript
form.createTextField("name", {
  page: 0,
  rect: { x: 100, y: 500, width: 200, height: 30 },
  value: "Default text",
});

form.createCheckbox("agreed", {
  page: 0,
  rect: { x: 100, y: 400, width: 20, height: 20 },
  checked: false,
});
```

---

## Drawing

All drawing methods are on `PDFPage`.

### Draw text
```typescript
import { rgb } from "@libpdf/core";

page.drawText("Hello, World!", {
  x: 50,
  y: 700,
  fontSize: 24,
  color: rgb(0, 0, 0),
});

page.drawText("Rotated text", {
  x: 200,
  y: 400,
  fontSize: 16,
  rotate: { angle: 45 },
});
```

### Draw shapes
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
  color: rgb(0, 0, 1),
});
```

### Draw SVG path
```typescript
page.drawSvgPath("M 0 0 L 100 100 L 200 0 Z", {
  x: 100,
  y: 200,
  color: rgb(0, 0.5, 0),
});
```

---

## Text Operations

### Extract text
```typescript
const text = page.extractText();
console.log(text); // All text on page
```

### Search text
```typescript
const results = page.findText(/invoice #\d+/gi);
for (const result of results) {
  console.log(result.text); // Matched text
  console.log(result.rect); // Bounding box { x, y, width, height }
}
```

---

## Images & Fonts

### Embed image
```typescript
const image = await pdf.embedImage(jpegBytes, "jpeg");
const image = await pdf.embedImage(pngBytes, "png");

page.drawImage(image, {
  x: 100,
  y: 400,
  width: 200,
  height: 150,
});
```

### Embed font
```typescript
const font = await pdf.embedFont(ttfBytes);

page.drawText("Custom font", {
  x: 100,
  y: 500,
  font,
  fontSize: 18,
});
```

### Use standard font
```typescript
import { StandardFonts } from "@libpdf/core";

const font = StandardFonts.Helvetica;

page.drawText("Standard font", {
  x: 100,
  y: 500,
  font,
  fontSize: 16,
});
```

---

## Digital Signatures

### Sign with P12 certificate
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
```typescript
import { HttpTimestampAuthority } from "@libpdf/core";

const tsa = new HttpTimestampAuthority("https://freetsa.org/tsr");

const signed = await pdf.sign({
  signer,
  level: "pades-lt", // or "pades-lta"
  timestampAuthority: tsa,
  reason: "Long-term signature",
});
```

### Sign with CryptoKey (browser)
```typescript
import { CryptoKeySigner } from "@libpdf/core";

const keyPair = await crypto.subtle.generateKey(
  { name: "RSASSA-PKCS1-v1_5", modulusLength: 2048, publicExponent: new Uint8Array([1, 0, 1]), hash: "SHA-256" },
  true,
  ["sign", "verify"]
);

const signer = await CryptoKeySigner.create(keyPair.privateKey, certificateBytes);

const signed = await pdf.sign({ signer });
```

---

## Encryption

### Encrypt PDF (on save)
```typescript
const encrypted = await pdf.save({
  protection: {
    userPassword: "user123",
    ownerPassword: "owner456",
    permissions: {
      printing: "high-resolution",
      modifying: false,
      copying: false,
      annotating: true,
    },
  },
});
```

### Set protection
```typescript
pdf.setProtection({
  userPassword: "user123",
  ownerPassword: "owner456",
  permissions: {
    printing: "high-resolution",
    modifying: false,
  },
});

const encrypted = await pdf.save();
```

### Remove protection
```typescript
pdf.removeProtection();
const decrypted = await pdf.save();
```

### Check encryption status
```typescript
const security = pdf.getSecurityInfo();
console.log(security.isEncrypted); // true/false
console.log(security.algorithm); // "aes-256", "aes-128", "rc4", null
console.log(security.permissions); // { printing, modifying, copying, ... }
```

---

## Merge & Split

### Merge PDFs
```typescript
const merged = await PDF.merge([pdf1Bytes, pdf2Bytes, pdf3Bytes]);
```

### Merge with options
```typescript
const merged = await PDF.merge([pdf1Bytes, pdf2Bytes], {
  title: "Combined Document",
  author: "Jane Doe",
});
```

### Extract pages
```typescript
const extracted = await pdf.extractPages([0, 2, 5]);
```

### Copy pages to another PDF
```typescript
const targetPdf = PDF.create();
const [page1, page2] = await targetPdf.copyPagesFrom(sourcePdf, [0, 1]);
```

---

## Metadata

### Get metadata
```typescript
const title = pdf.getTitle();
const author = pdf.getAuthor();
const subject = pdf.getSubject();
const keywords = pdf.getKeywords();
const creator = pdf.getCreator();
const producer = pdf.getProducer();
const creationDate = pdf.getCreationDate();
const modificationDate = pdf.getModificationDate();
```

### Set metadata
```typescript
pdf.setTitle("My Document");
pdf.setAuthor("Jane Doe");
pdf.setSubject("Annual Report");
pdf.setKeywords(["finance", "2024"]);
pdf.setCreator("My App");
pdf.setCreationDate(new Date());
```

---

## Attachments

### Get attachments
```typescript
const attachments = pdf.getAttachments();
for (const attachment of attachments) {
  console.log(attachment.name);
  console.log(attachment.description);
  const bytes = await attachment.getData();
}
```

### Add attachment
```typescript
pdf.addAttachment(fileBytes, {
  name: "invoice.pdf",
  description: "Original invoice",
  mimeType: "application/pdf",
});
```

### Remove attachment
```typescript
const attachments = pdf.getAttachments();
pdf.removeAttachment(attachments[0]);
```

---

## Advanced Patterns

### Incremental saves
```typescript
// Critical for signed PDFs
const modified = await pdf.save({ incremental: true });
```

### Low-level object access
```typescript
import { PdfDict, PdfName } from "@libpdf/core";

const catalog = pdf.context.catalog;
const pagesDict = catalog.get("Pages") as PdfDict;
const count = pagesDict.get("Count");
```

### Flatten all (forms + annotations + layers)
```typescript
await pdf.flattenAll(); // Prepare for signing/archival
```

---

For complete working examples, see `/examples/` directory organized by category.
