This is the advanced step of pdfman.md
there all we did was to generate the output, here we focus on "Generating an ouptut in a specific format that is in 
json and then use that output to update in google sheets as google sheets use json as input

# 📌 Step 6 — Converting Extracted PDF Text Into Structured JSON

Once you have raw text extracted using Poppler (`pdftotext`), you now convert it into **clean JSON** so it can be used by Google Sheets, databases, APIs, etc.

This step uses:

* **Basic LLM Chain**
* **Structured Output Parser**
* **DeepSeek / OpenAI model (JSON Mode)**

---

## ✅ 6.1 Create a Structured Output Schema

Add a **Structured Output Parser** node.

Use **Define Using JSON Schema** and paste this:

```json
{
  "type": "object",
  "properties": {
    "Email":       { "type": "string" },
    "Address":     { "type": "string" },
    "PlaceOfSupply": { "type": "string" },
    "HSNCode":     { "type": "string" },
    "State":       { "type": "string" }
  },
  "required": ["Email", "Address", "PlaceOfSupply", "HSNCode", "State"]
}
```

This guarantees the LLM **must** return JSON and ONLY JSON.

---

## ✅ 6.2 Connect the Parser to the Basic LLM Chain

In **Basic LLM Chain**:

* **Model →** connect to your DeepSeek Chat Model
* **Output Parser →** connect to Structured Output Parser
* Result = clean validated JSON

⚠️ The UI now shows dotted lines (new n8n update) — this is NORMAL.
As long as the node appears under **INPUT → Basic LLM Chain → output**, the connection is correct.

---

## ✅ 6.3 Add Your Prompt (With PDF Text)

Set **Prompt Source: Define below**

Use this prompt:

```
Your task is to extract ONLY the following fields from the PDF text:

- Email
- Address
- PlaceOfSupply
- HSNCode
- State

Return ONLY valid JSON following this exact schema:

{
  "Email": "",
  "Address": "",
  "PlaceOfSupply": "",
  "HSNCode": "",
  "State": ""
}

PDF TEXT:
---
{{ $('Execute Command').item.json.stdout }}
---
```

Enable:

✔️ Require Specific Output Format
✔️ Model in JSON mode (DeepSeek → Response Format: JSON)

---

# 📌 Step 7 — Using The JSON Output to Update Google Sheets

Now that the LLM Chain produces:

```json
{
  "Email": "finance@zomato.com",
  "Address": "Pioneer Square...",
  "PlaceOfSupply": "Telangana (36)",
  "HSNCode": "999799",
  "State": "Haryana"
}
```

We will send this to Google Sheets.

---

## ✅ 7.1 Add "Append Row in Sheet" Node

Set:

* **Resource:** Sheet Within Document
* **Operation:** Append Row
* **Mapping Column Mode:** Map Each Column Manually

---

## ✅ 7.2 SWITCH EACH FIELD TO “EXPRESSION”

This is **critical.**

Click the tiny button:
`Fixed → Expression`

Then add:

### **Email**

```
{{ $json["Email"] }}
```

### **Address**

```
{{ $json["Address"] }}
```

### **PlaceOfSupply**

```
{{ $json["PlaceOfSupply"] }}
```

### **HSNCode**

```
{{ $json["HSNCode"] }}
```

### **State**

```
{{ $json["State"] }}
```

You will know it's correct if the **left INPUT panel** shows:

```
Basic LLM Chain
 └── output
      ├── Email
      ├── Address
      ├── PlaceOfSupply
      ├── HSNCode
      └── State
```

---

# 📌 Step 8 — Final Workflow Order

This is your finalized pipeline:

```
Gmail Trigger
   ↓
Read/Write From Disk
   ↓
Execute Command (pdftotext stdout)
   ↓
Basic LLM Chain (with JSON Mode)
   ↓
Structured Output Parser
   ↓
Google Sheets (Append Row)
```

---

# 📌 Step 9 — Common Errors & Fixes

### ❌ Wrong: Fixed mode

Sheets will show:

```
{{ $json["Email"] }}
```

### ✅ Correct: Expression mode

Sheets resolves actual values:

```
finance@zomato.com
```

---

# 📌 Step 10 — Your Automation Is Now Complete

You now have:

✔ Automatic email → PDF → text
✔ Automatic text → clean structured JSON
✔ Automatic JSON → Google Sheets row
✔ End-to-end automated data pipeline

---

# 📌 Step 11 — Send Confirmation Email (“Updated to Sheets”)

After the Google Sheets node successfully appends the row, we send an automated email notifying that the invoice has been processed and added to the sheet.

This helps you keep track, debug faster, and confirm your pipeline is running end-to-end.

---

## ✅ 11.1 Add the “Send Email” Node

1. Drag **Gmail → Send Email** node
2. Connect it **after Google Sheets**

Workflow:

```
Google Sheets (Append Row)
       ↓
Send Email
```

---

## ✅ 11.2 Configure the Email Node

Set these fields:

### **To**

Your email or any tracking email:

```
yourEmail@gmail.com
```

### **Subject**

```
Invoice Updated to Google Sheets
```

### **Message (HTML / Text)**

Use a dynamic message so you can see exactly what was added.

```
The invoice extraction and sheet update were completed successfully.

Details added:

Email: {{ $('Basic LLM Chain').item.json.output.Email }}
Address: {{ $('Basic LLM Chain').item.json.output.Address }}
Place of Supply: {{ $('Basic LLM Chain').item.json.output.PlaceOfSupply }}
HSN Code: {{ $('Basic LLM Chain').item.json.output.HSNCode }}
State: {{ $('Basic LLM Chain').item.json.output.State }}

✔ Successfully added to Google Sheet: "Invoice"
```

This gives you a beautiful status email every time the automation finishes.

---

## 🧠 11.3 Dynamic Success Message (Optional)

If you want the message to include timestamp:

```
Completed at: {{ $now }}
```

If you want the PDF filename included:

```
PDF File: {{ $('Read/Write Files from Disk').item.binary.attachment_0.fileName }}
```

---

# 📌 Final Updated Pipeline

```
Gmail Trigger
    ↓
Read/Write Files From Disk
    ↓
Execute Command (pdftotext)
    ↓
Basic LLM Chain (JSON)
    ↓
Structured Output Parser
    ↓
Google Sheets (Append Row)
    ↓
Send Email (“Updated to Sheets”)
```

---

# 📌 Final Confirmation Email Example

```
Subject: Invoice Updated to Google Sheets

The invoice has been processed successfully.

Email: finance@zomato.com
Address: Pioneer Square...
Place Of Supply: Telangana (36)
HSN Code: 999799
State: Haryana

✔ Successfully updated in Sheet1 (Invoice)
```

---

