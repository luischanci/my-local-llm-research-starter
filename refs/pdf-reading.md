# Reading papers in master_supporting_docs/

Read when a task requires information from a PDF.

A PDF cannot be read whole: a 40-page paper exceeds the working context. Convert to text
and read only the pages needed.

## 1. Check the tool

```powershell
Get-Command pdftotext, pdfinfo
```
If not found, stop and ask the PI (poppler must be installed while the machine is online).

## 2. Size and structure

```powershell
pdfinfo master_supporting_docs/paper.pdf | Select-String "Pages"
pdftotext -layout -f 1 -l 2 master_supporting_docs/paper.pdf tmp/paper_p1-2.txt
```
Read the first two pages (abstract, introduction) to locate the relevant section.

## 3. Extract only what is needed

```powershell
pdftotext -layout -f 12 -l 15 master_supporting_docs/paper.pdf tmp/paper_p12-15.txt
```
At most about five pages per extraction. Say which pages you read.

## Rules

- Quote page numbers for every fact taken from a paper.
- Equations and tables often extract badly; if a formula matters, say it needs checking against the PDF.
- Scanned (image-only) PDFs produce empty text; report this rather than guessing.
- Extracted text goes in `tmp/`, never into the repository's reports.
