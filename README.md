Convert structured, paper-based dental records into a searchable digital database.

---

## PHASE 1: PLANNING & SETUP

### 1.1 Document Assessment

* Identify types of structured forms (e.g. patient intake, insurance, treatment, X-rays).
* Determine key fields per form (e.g. Name, DOB, Procedure, Date, Notes).
* Check for **handwritten vs typed** text.
* Estimate number of pages and files.

### 1.2 Environment Setup

Install and configure the following tools:

| Tool                         | Purpose                      |
| ---------------------------- | ---------------------------- |
| **Tesseract OCR**            | Core OCR engine              |
| **OpenCV**                   | Image pre-processing         |
| **pdf2image**                | Convert PDF to image         |
| **pytesseract**              | Python wrapper for Tesseract |
| **SQLite** or **PostgreSQL** | Store structured data        |
| **Python**                   | Main scripting language      |

> Recommended: Use a virtual environment with Python 3.10+

```bash
pip install pytesseract opencv-python pdf2image Pillow sqlite3
```

### Hardware Requirements

* A **scanner** (300 DPI minimum) or multi-function printer.
* Computer with ≥ 8 GB RAM and ≥ 100 GB free storage.

---

## PHASE 2: DIGITIZATION & SCANNING

### 2.1 Scan Documents

* Scan in **300 DPI**, grayscale or color.
* Save as **PDFs** (one file per patient or form).
* Use naming convention: `Lastname_Firstname_DOB.pdf`.

### 2.2 Organize Files

* Create folders by year or type:

  ```
  digitized_records/
    ├── 2023/
    ├── 2024/
    └── insurance_forms/
  ```

---

## PHASE 3: IMAGE PROCESSING & OCR

### 🧪 3.1 Convert PDFs to Images

```python
from pdf2image import convert_from_path

images = convert_from_path("sample_form.pdf", dpi=300)
images[0].save("page1.png", "PNG")
```

### 3.2 Preprocess Images (OpenCV)

```python
import cv2
def preprocess(img_path):
    img = cv2.imread(img_path, cv2.IMREAD_GRAYSCALE)
    img = cv2.threshold(img, 150, 255, cv2.THRESH_BINARY_INV + cv2.THRESH_OTSU)[1]
    img = cv2.medianBlur(img, 3)
    return img
```

---

## PHASE 4: STRUCTURED OCR EXTRACTION

### 4.1 Run OCR with Layout Awareness

Use `pytesseract.image_to_data()` to extract **text + bounding boxes**.

```python
import pytesseract
import pandas as pd

def extract_text(img):
    data = pytesseract.image_to_data(img, output_type=pytesseract.Output.DATAFRAME)
    return data[data.text.notnull()]
```

### 4.2 Map Fields Using Coordinates (Template Matching)

* Choose a **template form**.
* Record pixel coordinates for fields like:

  * Name → x: 100-400, y: 80-120
  * Date → x: 500-650, y: 80-120

Filter data:

```python
def get_field(df, x1, x2, y1, y2):
    region = df[(df.left > x1) & (df.left < x2) & (df.top > y1) & (df.top < y2)]
    return " ".join(region.text.tolist())
```

---

## PHASE 5: STRUCTURED DATA STORAGE

### 5.1 Design a Database Schema

Example schema for `patients` table:

```sql
CREATE TABLE patients (
    id INTEGER PRIMARY KEY,
    name TEXT,
    dob TEXT,
    visit_date TEXT,
    procedures TEXT,
    notes TEXT
);
```

### 5.2 Populate Automatically

```python
import sqlite3

conn = sqlite3.connect('dental_records.db')
cursor = conn.cursor()

cursor.execute("""
INSERT INTO patients (name, dob, visit_date, procedures, notes)
VALUES (?, ?, ?, ?, ?)""", (name, dob, visit, procedures, notes))
conn.commit()
```

---

## PHASE 6: SEARCH & ACCESS

### 6.1 Query Interface

You can build a simple **Tkinter GUI** or **Flask web app** for:

* Searching by name/DOB
* Viewing visits and procedures
* Exporting to CSV

Or use a tool like:

* **DB Browser for SQLite** (GUI, free)
* **Apache Superset** (if using PostgreSQL)

---

## PHASE 7: QUALITY CONTROL & ERROR HANDLING

### 7.1 Manual Verification

* Randomly check 5-10% of digitized files.
* Compare scanned values vs extracted ones.

###  7.2 Error Logging

Log:

* Low confidence values (Tesseract returns confidences).
* Skewed scans
* Missing fields

---

## PHASE 8: SECURITY & BACKUP

### 8.1 HIPAA-Friendly Practices (Best-Effort)

* Store data locally or on HIPAA-compliant services.
* Encrypt `dental_records.db` using tools like **SQLCipher**.
* Password-protect access to files/scripts.

### 8.2 Backup Strategy

* Regularly back up files to an **external drive**.
* Optionally automate with `cron` jobs.

---

## OPTIONAL: HANDWRITING SUPPORT

For fields with handwriting (e.g. dentist notes):

* Try **TrOCR** (Transformers for OCR): [https://huggingface.co/docs/transformers/en/model_doc/trocr]
* Or use **Google Cloud Vision OCR** (free for small use)

---

## Timeline Estimate (1–2 Months)

| Week | Task                                       |
| ---- | ------------------------------------------ |
| 1    | Setup tools, scan test set, build template |
| 2-3  | Write and test OCR + field extraction      |
| 4-5  | Extract from bulk files, populate DB       |
| 6    | Build GUI/search, verify data              |
| 7    | QA, documentation, backup                  |


