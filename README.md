# PDF Key Field Extraction Pipeline

A document automation pipeline that extracts key fields from loan worksheet PDFs using PyMuPDF, regex, and OpenCV. Built as part of an AI/document automation externship.

---

## What It Does

- Uploads a PDF document via Google Colab
- Extracts all text and bounding box coordinates word by word
- Detects and extracts table data with cell-level bounding boxes
- Uses regex to locate and extract 8 key fields from the document
- Visualizes extracted fields by drawing bounding boxes on the PDF page

---

## Pipeline Architecture
PDF Upload
↓
extractor()
├── get_text('words', sort=True)  → words + bboxes
└── find_tables(strategy='text') → table data + cell bboxes
↓
regex_search()
├── Tier 1: search fulltext (plain text)
└── Tier 2: search celltext (table text)
↓
visualize_keyfields()
└── draw bounding boxes around label + value pairs

---

## Key Fields Extracted

| Field | Example Value |
|---|---|
| Applicants | John Q. Smith / Mary A. Smith |
| Application No | samplesmith |
| Date Prepared | 10/05/2015 |
| Prepared By | XYZ Lender |
| Total Loan Amount | $ 380,000 |
| Interest Rate | 4.250 % |
| Term/Due In | 360 / 360 mths |
| Total Monthly Payment | 2,308.95 |

---

## Tech Stack

- Python 3
- PyMuPDF (`pymupdf`) — PDF parsing, text + bbox extraction, table detection
- OpenCV (`cv2`) — bounding box visualization
- Pillow (`PIL`) — PDF to image conversion
- NumPy — image array handling
- `re` — regex field extraction
- Google Colab — runtime environment

---

## Setup

### Install dependencies

```bash
pip install pymupdf opencv-python pillow numpy
```

> **Important:** Only install `pymupdf`. Never install the separate `fitz` package — it is unrelated and will break the import.

### Google Colab specific

```python
from google.colab import files
from google.colab.patches import cv2_imshow
```

---

## How to Run

### 1. Upload your PDF

```python
doc_upload = files.upload()
pdf_filename = list(doc_upload.keys())[0]
```

### 2. Extract text and tables

```python
word_output, table_output = extractor(pdf_filename)
```

### 3. Run regex extraction

```python
extracted_fields = regex_search(word_output, table_output)
for field, value in extracted_fields.items():
    print(f'{field}: {value}')
```

### 4. Visualize with bounding boxes

```python
visualize_keyfields(pdf_filename, word_output, extracted_fields)
```

---

## File Structure
pdf-extraction-pipeline/
│
├── extraction_pipeline.ipynb   ← main Colab notebook
├── LenderFeesWorksheetNew.pdf  ← sample input document
└── README.md

---

## Functions

### `extractor(pdf_filename)`
Opens the PDF and returns two outputs:
- `word_output` — list of dicts, each containing `text` and `bbox` for every word on the page
- `table_output` — list of dicts, each containing `page`, `table index`, `bbox`, `data`, and `cell bbox` for every table detected

### `regex_search(word_output, table_output)`
Builds a searchable string from both word output and table output. Runs hardcoded regex patterns for each of the 8 key fields. Returns a `results` dictionary mapping field names to extracted values. Returns `'Not found'` for any field the regex cannot match.

### `find_phrase_bbox(phrase, word_output)`
Searches `word_output` for a multi-word phrase using a sliding window. Returns the combined bounding box `(x0, y0, x1, y1)` of the entire phrase, or `None` if not found.

### `visualize_keyfields(pdf_filename, word_output, results)`
Converts the PDF page to an image, finds the bounding box of each field label and its value, combines them into one rectangle, and draws it in green on the image.

---

## Known Limitations

- Hardcoded regex patterns — works specifically for Lender Fees Worksheet format. Different document layouts require new patterns.
- `find_tables(strategy='text')` produces messy results on borderless tables — table data is extracted but cell splitting is imprecise. All 8 key fields are successfully found via plain text extraction.
- Currently processes page 1 only. Multi-page support requires extending the page loop in `extractor()`.
- Name pattern assumes `First Middle. Last` format — edge cases like suffixes or hyphenated names may not match.

---

## Future Improvements

- Replace hardcoded regex with LLM-based extraction for truly generalized field detection across any document type
- Explore `camelot` or `pdfplumber` for more robust borderless table detection
- Add multi-page support
- Export extracted fields to CSV or JSON
- Build a Streamlit UI so non-technical users can upload and extract without touching code

---

## Lessons Learned

This pipeline was built incrementally with real debugging — key lessons documented during development:

- Always use `sort=True` with `get_text('words')` for correct visual reading order
- Never install both `pymupdf` and `fitz` — they conflict on the same namespace
- `find_tables()` default strategy requires visible borders — use `strategy='text'` for borderless tables
- Regex patterns must be verified against actual extracted text, not assumed document structure
- `with fitz.open()` is safer than manual `doc.close()` — auto-closes even on crash

---

## Author

Built by Varun — AI/Document Automation.

P.S: I welcome any suggestions and Pull requests. Thanks! :D
