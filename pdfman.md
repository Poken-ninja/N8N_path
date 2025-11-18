# n8n + Poppler PDF Automation Guide (Windows)

This guide documents the full working solution for:

* Fetching PDF attachments from Gmail
* Saving them to disk using n8n
* Converting PDF → text using Poppler (`pdftotext`)
* Extracting structured fields inside n8n
* Avoiding all common Windows path & Poppler issues

---

## 🚀 1. Gmail → n8n: Handling Binary Attachments

Ensure the Gmail Trigger outputs binary data.

The binary property is typically:

```
attachment_0
```

Check the exact name inside n8n under **Binary** tab of the Gmail Trigger output.

---

## 📁 2. Write PDF to Disk (IMPORTANT)

Use a folder path **without spaces** to avoid Poppler issues.

Example folder:

```
C:/n8n_files/
```

**Write Binary File Node:**

* Operation: **Write File to Disk**
* Binary Property:

```
attachment_0
```

* File Path:

```
C:/n8n_files/{{ $('Gmail Trigger').item.binary.attachment_0.fileName }}
```

This will correctly create:

```
C:/n8n_files/Invoice.pdf
```

Avoid missing slashes — the most common error:

```
❌ C:/n8n_files{{filename}}            (WRONG)
✔ C:/n8n_files/{{filename}}            (CORRECT)
```

---

## 🛠️ 3. Convert PDF → Text Using Poppler

Poppler's `pdftotext` command is used to extract raw text.

### ✔ Run `pdftotext` with stdout output

Using `-` prints text directly to stdout so n8n can use it:

```
"C:/path/to/pdftotext.exe" "C:/n8n_files/{{ $('Gmail Trigger').item.binary.attachment_0.fileName }}" -
```

### Why stdout?

* n8n can immediately use text output
* No extra file writing
* Cleaner parsing workflow

### Important rules:

* **Exactly one output argument**—either a file OR `-`, not both.

Correct:

```
pdftotext input.pdf -
```

Incorrect:

```
pdftotext input.pdf output.txt -     (❌ causes usage error)
```

---

## 📤 4. Extract Fields From PDF Text in n8n

Add a **Function** node after Execute Command.

Paste this:

```js
const text = $('Execute Command').item.json.stdout || '';

function extract(pattern) {
  const match = text.match(pattern);
  return match ? match[1].trim() : '';
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

This extracts 5 key invoice fields:

* Email
* Address
* Place of Supply
* HSN Code
* State

You can now store, send, or process this structured JSON however you like.

---

## 🧩 5. Troubleshooting (MOST IMPORTANT)

### ❗ PDF not writing to disk

Check for:

* Wrong binary property (attachment_0 vs data)
* Missing `/` before filename
* Folder doesn’t exist

### ❗ Poppler cannot open file

Usually caused by:

* Space in folder name
* PDF file not actually written
* Wrong path
* Wrong file name (case-sensitive)

### ❗ Execute Command shows empty stdout

This is normal **unless** using `-` at the end.

Use:

```
pdftotext input.pdf -
```

---

## 🎯 Final Architecture Flow

```
Gmail Trigger
    ↓
Write Binary File → C:/n8n_files/Invoice.pdf
    ↓
Execute Command (pdftotext → stdout)
    ↓
Function Node (Regex extraction)
    ↓
JSON Output with structured invoice data
```

---

