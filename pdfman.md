# n8n + Poppler PDF Automation Guide (Windows)

A fully working, production-ready workflow for automating PDF extraction from Gmail, converting PDFs using Poppler on Windows, and extracting invoice fields via **regex OR LLM**.

---

# 📌 Overview

This workflow covers:

* Fetching Gmail PDF attachments
* Writing PDF to disk (Windows-safe)
* Running Poppler (`pdftotext`) correctly
* Extracting fields via **regex**
* OR extracting fields using an **LLM node** (DeepSeek/OpenAI/Gemini)
* Avoiding all common Windows path issues

---

# 🚀 1. Gmail Trigger → Binary Attachment Handling

Gmail Trigger must output a binary file.

Binary name is usually:

```
attachment_0
```

Verify under **Binary** tab in the Gmail node output.

EX:

<img width="1895" height="930" alt="image" src="https://github.com/user-attachments/assets/09e22bbe-3829-42a8-9c22-b706ee8c1e90" />

---

# 📁 2. Write PDF to Disk (NO SPACES)

Use a folder with no spaces. Example:

```
C:/n8n_files/
```

**Write Binary File Node:**

```
Binary Property: attachment_0
File Path: C:/n8n_files/{{ $('Gmail Trigger').item.binary.attachment_0.fileName }}
```

This produces:

```
C:/n8n_files/Invoice.pdf
```

Common mistakes:

```
❌ C:/n8n_files{{filename}}
✔ C:/n8n_files/{{filename}}
```
EX:

<img width="1888" height="894" alt="image" src="https://github.com/user-attachments/assets/c93bbee6-6a19-4860-aa7f-bf10263271f3" />

---

# 🛠️ 3. PDF → Text Using Poppler

Use stdout so n8n can read the extracted text directly.

```
"C:/path/to/pdftotext.exe" "C:/n8n_files/{{ $('Gmail Trigger').item.binary.attachment_0.fileName }}" -
```

### Why use `-`?

* It prints ALL text directly to n8n
* No file output needed
* Cleaner + faster

### Correct forms:

```
pdftotext input.pdf -
```

Incorrect:

```
pdftotext input.pdf output.txt -   ❌
```
Ex: 1. Here I used - and '-' is use to print the output to stdout so that it can be used as an input for regexx or llm chain

<img width="1903" height="897" alt="image" src="https://github.com/user-attachments/assets/5363fef1-ba02-4983-a095-7561b7d830b6" />


Ex: 2: if i used this instead, then I'd have generated an output.txt (here test.txt) use this case when we just need to etract the pdf text into local files for your personal uses.
<img width="1896" height="849" alt="image" src="https://github.com/user-attachments/assets/b5923c28-e75d-4056-ad1a-1c8f295bcc2b" />

This generates a .txt files for local use or whatever
<img width="750" height="33" alt="image" src="https://github.com/user-attachments/assets/adfcc686-250a-496a-b2fa-8752048c7cc8" />


---

# 🎯 4. Field Extraction Options (Regex OR LLM)

Two ways to extract invoice fields:

---

## **4A — Extract Using Regex (Free Method)**

Add a **Function** node after Execute Command:

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

Extracts:

* Email
* Address
* Place of Supply
* HSN Code
* State

---

## **4B — OR Extract Using LLM (DeepSeek / OpenAI / Gemini)**

Add a **Basic LLM Chain** node after Execute Command.

### Prompt:

```
Identify and extract 5 key fields:
1. "Email"
2. "Address"
3. "Place of supply"
4. "HSN Code"
5. "State"

Use the data from:
---
{{ $('Execute Command').item.json.stdout }}
---
```

### Example Output:

```
Email: finance@zomato.com
Address: Pioneer Square, Tower 1- Ground to 6th Floor and Tower 2- 1st and 2nd Floors, Near Golf Course Extension, Sector-62, Gurugram, Haryana - 122098
Place of supply: Telangana (36)
HSN Code: 999799
State: Haryana
```

### Pros:

* Handles messy / inconsistent PDFs
* Works even when regex breaks
* Great for multi-line fields

### Cons:

* Costs API tokens

EX: In this i used llm to extract text, one important thing to notice here is "use the data from:
"---
{{ $('Execute Command').item.json.stdout }}
---" in this line --- is used as a seperator to ensure the llm does'nt get confused and the {{...}} means its an expresion
Breakdown of expression:
{{ $('Execute Command').item.json.stdout }}

✔ {{ ... }}

This means Expression Mode (JavaScript inside n8n).

✔ $()

This is the node selector function in n8n.

✔ $('Execute Command') || Why Select execute node? since its the previous node , we use it as an input

This selects the node named Execute Command.

✔ .item

This selects the current item being processed.

✔ .json

This selects the JSON data of that item.

✔ .stdout

This selects the output text produced by Poppler.

<img width="1895" height="936" alt="image" src="https://github.com/user-attachments/assets/7123c329-b079-47e9-91ec-2e881deac75d" />

---

# 🧩 5. Troubleshooting (CRITICAL)

### ❗ PDF not writing?

* Wrong binary property
* Missing trailing slash
* Folder not created

### ❗ Poppler error: "Could not open file"

* Path has spaces
* File never saved
* Wrong filename

### ❗ Empty stdout

* You forgot `-`

Correct:

```
pdftotext input.pdf -
```

---

# 🏗️ Final Workflow Architecture

```
Gmail Trigger
      ↓
Write Binary File
      ↓
Execute Command (pdftotext)
      ↓
Regex Extraction  OR  LLM Extraction
      ↓
Structured JSON Output
```

here is the predicted output for the aboce steps:

<img width="1830" height="857" alt="image" src="https://github.com/user-attachments/assets/e5b40bcb-6447-409d-837e-58365e151eaa" />



---
