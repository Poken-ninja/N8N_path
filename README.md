# n8n + Poppler PDF Automation Suite (Windows)

### **Automated Invoice Parsing • PDF → Text/Image Extraction • GST/HSN Validation**

This repository provides a complete, production-ready automation workflow using **n8n** + **Poppler** on **Windows**. It includes:

* Automatic PDF download from Gmail
* PDF → text extraction using `pdftotext`
* PDF → PNG/JPG conversion using `pdftoppm`
* Invoice field extraction (Email, Address, State, HSN Code, Place of Supply)
* HSN validation logic
* GST invoice workflow blueprint
* Regex-tuned extraction for messy PDFs
* Example n8n workflows & advanced automation modules

---

# 📦 Features

### ✔ Gmail → PDF ingestion (binary-safe)

### ✔ Poppler integration for Windows

### ✔ Text extraction via stdout (no temporary files)

### ✔ Structured invoice field extraction (JSON)

### ✔ PNG/JPG image conversion

### ✔ HSN Code validation

### ✔ GST logic blueprint

### ✔ Troubleshooting guide

### ✔ Advanced regex patterns

---

# 🚀 Quick Start

## **1. Install Poppler (Windows)**

Download Poppler:

```
https://github.com/oschwartz10612/poppler-windows/releases
```

Extract and note the path to:

```
pdftotext.exe
pdftoppm.exe
```

---

## **2. Prepare Folder (No Spaces!)**

Create:

```
C:/n8n_files/
```

This avoids Windows/Poppler path errors.

---

## **3. Gmail Trigger → Write Binary File**

Use:

```
Binary Property: attachment_0
File Path: C:/n8n_files/{{ $('Gmail Trigger').item.binary.attachment_0.fileName }}
```

---

# 🛠 PDF → TEXT (Poppler)

Use this in **Execute Command node**:

```
"C:/path/to/pdftotext.exe" "C:/n8n_files/{{ $('Gmail Trigger').item.binary.attachment_0.fileName }}" -
```

The final `-` prints text to stdout → perfect for next nodes.

---

# 🧠 Field Extraction (Function Node)

```js
const text = $('Execute Command').item.json.stdout || '';

function extract(rgx) {
  const m = text.match(rgx);
  return m ? m[1].trim() : '';
}

return [{
  json: {
    Email: extract(/Email\s*[:\-]?\s*(.*)/i),
    Address: extract(/Address\s*[:\-]?\s*(.*)/i),
    PlaceOfSupply: extract(/Place\s+of\s+Supply\s*[:\-]?\s*(.*)/i),
    HSNCode: extract(/HSN\s*Code\s*[:\-]?\s*(.*)/i),
    State: extract(/State\s*[:\-]?\s*(.*)/i),
  }
}];
```

---

# 📸 PDF → PNG/JPG Conversion

```
"C:/path/to/pdftoppm.exe" "C:/n8n_files/{{ $('Gmail Trigger').item.binary.attachment_0.fileName }}" "C:/n8n_files/output" -png
```

Or JPG:

```
... -jpeg
```

High DPI:

```
... -png -r 200
```

---

# 🧾 HSN Code Validation

```js
const hsn = $json.HSNCode || '';
return [{ json: { ...$json, HSNValid: /^\d{4,8}$/.test(hsn) }}];
```

---

# 📚 Recommended Repo Structure

```
n8n-poppler-automation/
 ┣ workflows/
 ┃ ┣ gmail_to_text.json
 ┃ ┗ pdf_to_png.json
 ┣ docs/
 ┃ ┣ basic_guide.md
 ┃ ┗ advanced_guide.md
 ┣ examples/
 ┃ ┣ invoice.pdf
 ┃ ┗ output.txt
 ┣ README.md
 ┗ LICENSE
```

---

# 🧬 GST Workflow Blueprint

```
PDF
 ↓
Text (pdftotext)
 ↓
Field Extraction
 ↓
HSN Validation
 ↓
IF → Invalid → Notify
 ↓
Else → Store in DB / Google Sheets
```

---

# 🧪 Troubleshooting

### ❗ Empty stdout? Use `-` as last argument. (This ensures the - prints the output to stdout so that we can pass it the llm for extraction)

### ❗ "Could not open file"? Path incorrect or folder has spaces.

### ❗ Wrong fields extracted? Adjust regex.

### ❗ PDF not written? Wrong binary property in Gmail node.

---

Just ask: **"Add more modules"**
<img width="1497" height="893" alt="image" src="https://github.com/user-attachments/assets/fc74ae4e-f39b-486a-ba67-304426bf9295" /># N8N_path
<img width="1477" height="694" alt="image" src="https://github.com/user-attachments/assets/a8578438-63b9-4247-8e8a-a7d08854bd72" />
<img width="708" height="407" alt="image" src="https://github.com/user-attachments/assets/0ed6cac9-0f06-4e65-807a-0940851b8f13" />
<img width="1720" height="912" alt="image" src="https://github.com/user-attachments/assets/de16972b-71c1-4b20-9277-d8d9374cbfca" />
<img width="1497" height="893" alt="image" src="https://github.com/user-attachments/assets/cf0cc8a8-3b77-48f0-b904-b5ec1a64797a" />
<img width="932" height="485" alt="image" src="https://github.com/user-attachments/assets/31168faf-4cf5-431a-b196-e518c012fdfd" />
<img width="1879" height="934" alt="image" src="https://github.com/user-attachments/assets/d98a5993-4c9d-46d3-8b35-93be4d70b875" />
<img width="1736" height="893" alt="image" src="https://github.com/user-attachments/assets/5ca9a41f-19fa-4247-8567-31f88cf717a0" />
<img width="1820" height="743" alt="image" src="https://github.com/user-attachments/assets/36be264d-dabc-46c9-96c1-6785957d88af" />
<img width="1781" height="1079" alt="image" src="https://github.com/user-attachments/assets/3efb4fcb-bd8b-4838-aa84-ab5fab564775" />
<img width="1856" height="970" alt="image" src="https://github.com/user-attachments/assets/e20dd666-1a2b-40a6-a2c2-586ffdee19b7" />
<img width="1564" height="823" alt="image" src="https://github.com/user-attachments/assets/05d0c716-24c7-4d53-a6a2-e0524c6c79a9" />
<img width="1162" height="638" alt="image" src="https://github.com/user-attachments/assets/11c9c12a-f471-4dc8-8265-8652d3223151" />


https://github.com/oschwartz10612/poppler-windows/releases/tag/v25.07.0-0 this is the link used to download poppler the pdf tool
