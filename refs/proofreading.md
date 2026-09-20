# Proofreading a manuscript

Read when the PI asks for proofreading or consistency checks on `.tex` files.

## Rule

**Propose; never apply.** Do not edit the manuscript until the PI approves specific changes.

## Scope, one section at a time

Do not read the whole manuscript. Work on one `\section` (or one file of a multi-file paper)
per task. Locate it first:
```powershell
Select-String -Path Edition_[PAPER]/[paper].tex -Pattern "\\section"
```
Then read only that line range.

## Checks

1. **Grammar** — agreement, articles, prepositions, tense consistency.
2. **Typos** — misspellings, duplicated words, search-and-replace damage.
3. **Notation** — symbols match `refs/notation.md` if it exists; same symbol, same meaning.
4. **Terminology** — the same concept has the same name throughout the section.
5. **Numbers** — every number in the text matches its table or report; flag any you cannot trace.
6. **Claims** — causal language on associational results; overstatement of intervals that span zero.
7. **LaTeX** — undefined references, overfull boxes (from the log), `\citet` for textual and
   `\citep` for parenthetical citations.
8. **Cross-references** — every `\ref` has a matching `\label`; table and figure numbers
   cited in the text point to the right exhibit; every `\input` path exists
   (`Test-Path` each one).

## Output

Write `quality_reports/proofread_<section>.md`:

| # | Line | Category | Current text | Proposed text |
|---|---|---|---|---|

In chat, give only the count by category.

## After approval

Apply only the approved rows. Recompile (see `refs/verification.md`) and confirm the PDF builds.
