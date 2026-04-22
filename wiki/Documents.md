# Document Parsing

emprise can read and parse common document formats directly through the `read_file` tool. When you ask the LLM to read a file, it automatically detects the format by extension and returns structured text.

## Supported Formats

| Extension | Format | What You Get |
|-----------|--------|-------------|
| `.pdf` | PDF | Page-by-page plain text extraction |
| `.docx` | Word | Text with heading levels preserved as Markdown (`#`, `##`, `###`) |
| `.xlsx` / `.xls` | Excel | Markdown tables per sheet with row counts |
| `.pptx` | PowerPoint | Slide-by-slide text with slide numbers and titles |
| `.zip` | ZIP archive | File listing with sizes and compression ratios |
| `.ipynb` | Jupyter Notebook | Code cells with Python syntax highlighting, markdown cells preserved |
| `.csv` | CSV | Can be imported into SQLite for analysis via `csv_to_sqlite` |

All other text-based files (`.go`, `.py`, `.js`, `.yaml`, `.json`, `.md`, `.txt`, etc.) are read as plain text with line limits.

## How It Works

The `read_file` tool checks the file extension before reading. If it matches a supported document format, the specialized parser is used instead of plain text reading.

```
read_file(path: "report.pdf", limit: 200)
```

The `limit` parameter controls maximum lines of output (default: 200). For large documents, output is truncated with a note showing total pages/lines.

## PDF Parsing

Extracts plain text page by page:

```
--- Page 1 ---
Executive Summary
This report covers...

--- Page 2 ---
Section 1: Background
...

... truncated at 200 lines (15 pages total)
```

Uses the `github.com/ledongthuc/pdf` library. Works with text-based PDFs; scanned/image PDFs will return empty pages.

## DOCX Parsing

Extracts text from `word/document.xml` inside the DOCX zip container. Preserves heading structure:

```
# Main Title
## Section One
This is paragraph text from the document.
### Subsection
More content here.
```

Heading levels 1-3 are converted to Markdown headings. All other text is plain paragraphs.

## XLSX Parsing

Each sheet is rendered as a Markdown table:

```
## Sheet: Sales (150 rows)

| Region | Q1 | Q2 | Q3 | Q4 |
| --- | --- | --- | --- | --- |
| North | 1200 | 1350 | 1100 | 1500 |
| South | 900 | 1100 | 950 | 1200 |
...
```

Uses the `github.com/xuri/excelize/v2` library. Multi-sheet workbooks show all sheets. Pipe characters in cell values are escaped.

## PPTX Parsing

Extracts text from each slide's XML:

```
## Slide 1: Project Overview
  Our goals for Q4

## Slide 2: Architecture
  Microservices approach
  Event-driven design
  Kubernetes deployment
```

The first text element of each slide is treated as the title. Slides are sorted by number.

## ZIP Parsing

Lists archive contents with file sizes and compression ratios:

```
ZIP archive: 42 entries

  src/main.go                                          1234 bytes (45%)
  src/config.go                                         890 bytes (52%)
  README.md                                             456 bytes (38%)
```

Does not extract file contents; use a shell command for that.

## Jupyter Notebook Parsing

Parses the JSON structure to show cells with type labels:

```
## Cell 1 [code]
\`\`\`python
import pandas as pd
df = pd.read_csv("data.csv")
df.head()
\`\`\`

## Cell 2 [markdown]
# Data Analysis
This notebook explores the dataset...

## Cell 3 [code]
\`\`\`python
df.describe()
\`\`\`
```

Code cells are wrapped in Python code fences. Markdown cells are shown as-is.

## CSV Analysis

CSV files are not directly parsed by `read_file` (they are read as plain text). For analysis, use the `csv_to_sqlite` tool:

```
csv_to_sqlite(csv_path: "sales.csv", table: "sales")
```

This imports the CSV into a SQLite database (created alongside the CSV file) and registers it for use with `sql_query`. See [SQL Tools](SQL-Tools) for details.

## Line Limits

All parsers respect the `limit` parameter (default 200 lines). When truncated, a message indicates how much was cut:

```
... truncated at 200 lines (15 pages total)    # PDF
... truncated at 200 lines                      # DOCX, XLSX
... truncated at 200 entries                    # ZIP
```

## Examples

```
"Read the quarterly report"
→ read_file(path: "Q4-report.pdf")

"What's in this Excel file?"
→ read_file(path: "budget.xlsx")

"Summarize this presentation"
→ read_file(path: "deck.pptx")

"What's in this zip?"
→ read_file(path: "release.zip")

"Analyze this CSV data"
→ csv_to_sqlite(csv_path: "data.csv")
→ sql_query(db: "data", query: "SELECT * FROM data LIMIT 10")
```
