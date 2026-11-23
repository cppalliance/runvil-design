---
name: Restart Vendor Report
overview: Back up the 590 KB version to cabinet/_output, restart from the 169 KB original, fix its wording with the deslop rulebook, add a terms list and first-use explanations, and keep the result under 210 KB.
todos:
  - id: working-copy
    content: Copy the 169 KB original to cabinet/_scratch/deslop-vendor-report/report.md and edit only that file
    status: completed
  - id: names-and-corrections
    content: Apply the term sheet names and the 43 evidence corrections, and update the totals they move
    status: completed
  - id: prose-and-context
    content: Fix the rulebook patterns in the summary and sections 1, 2, 4 to 11; add the terms section, first-use explanations and the table legend
    status: completed
  - id: tables-and-links
    content: Shorten Table 15, link the 26 evidence files from section 12, fix the five moved research paths, drop the evidence-files decision
    status: completed
  - id: checks
    content: Run the row comparison, URL and commit check, rulebook searches, size check, and one fresh-reader round on the summary and section 1
    status: completed
  - id: backup-long
    content: When the sweep ends, copy the long version to cabinet/_output/conduct-research-vendor-feature-mapping-long-version.md
    status: completed
  - id: swap-commit
    content: Copy the working file over the report, move the cabinet evidence copies to cabinet/_trash, commit and push on master
    status: in_progress
isProject: false
---

# Restart the vendor feature report from the original

## Files

- Original, 169 KB and 821 lines: [cabinet/_scratch/conduct-research-openrouter/report-before-rewrite.md](cabinet/_scratch/conduct-research-openrouter/report-before-rewrite.md)
- Long version, 594 KB, published at commit `1099650`: [runvil-design/research/2026-11-22-openrouter-universal-api-deep-dive.md](runvil-design/research/2026-11-22-openrouter-universal-api-deep-dive.md)
- Rulebook to apply: [tools-public/rulebooks/deslop-rulebook.md](tools-public/rulebooks/deslop-rulebook.md)
- Old name to plain name list, already written: [cabinet/_scratch/conduct-research-openrouter/term-sheet.md](cabinet/_scratch/conduct-research-openrouter/term-sheet.md)
- The 43 evidence corrections, each with its quoted evidence item: [cabinet/_scratch/conduct-research-openrouter/rewrite/reports3/C-CORRECTIONS.md](cabinet/_scratch/conduct-research-openrouter/rewrite/reports3/C-CORRECTIONS.md)

## Backup

- The sweep keeps running, as you said. It writes the long version once more when it ends.
- At that point the long version is copied, byte for byte, to `cabinet/_output/conduct-research-vendor-feature-mapping-long-version.md`. That name is free in `cabinet/_output`.
- The restart works in its own file, `cabinet/_scratch/deslop-vendor-report/report.md`, so it starts now and the two jobs touch different files.

## What changes in the original

One editor makes one pass, in this order.

1. **Names.** Apply the term sheet: "the slot" becomes "the opaque field", the codes L0 to L5 become the six plain option names, and "ladder", "matrix", "roll-up", "OR-doc", "CC", "MSG" and "RESP" become plain words. This covers table cells too.
2. **Known errors.** Apply the 43 corrections to the matching sentences and cells, then update the totals they move: 75 features needed and 30 unused, confidence 55 high, 43 medium and 7 low, and 16 features that need a new setting. The six option totals stay at 65, 25, 10, 0, 5 and 0.
3. **Prose** in the summary and sections 1, 2 and 4 to 11. Fix each pattern in the rulebook: negation as referent, sentences about the report's own rows, slogans, shorthand lists, leads that announce a count, process narration, and confidence justified by the report itself. Headings state what their sections say. The title and opening name the mapping as the subject. Where the long version already says a thing plainly, reuse its sentence.
4. **Context.** Add a terms section near the top with one line per term. Explain each term at its first use in the summary and at its first use in each top-level section. Add a short legend above the feature tables.
5. **Table 15.** Cut its cells to short phrases and move the long reasons into a list under the table.
6. **Evidence and paths.** Section 12 links each of the 26 evidence files at `evidence/<file>`. The decision about moving them comes off the list, which leaves ten decisions. The five citations of moved research files point at `research/2026-09/` and `research/2026-10/`.

## What stays as it is

- **The ten feature tables keep their twelve columns and their original cells.** They change only by steps 1 and 2 and by rewording cells that give a table row or a level as their reason. The per-row notes of the long version stay out.
- Every number, quotation, classification, evidence key and link, apart from the 43 corrections.
- The section order and the file name.

## Limits

- **Size:** 210 KB at most, which is the original plus about 25 percent.
- **Fact-checking:** the 43 known corrections and nothing further.
- **Reader check:** one fresh reader on the summary and section 1, one round. Fix what matters and stop.
- **Time:** 90 minutes of editing. At that mark the editor stops and reports what is left.

## Checks before the swap

- All 105 features are present. Each one's option, confidence, setting and evidence keys match the original, apart from the listed corrections. A script compares against [cabinet/_scratch/conduct-research-openrouter/rewrite/orig-rows.json](cabinet/_scratch/conduct-research-openrouter/rewrite/orig-rows.json).
- Every URL and commit id of the original is present.
- The rulebook's search block is run and each hit is read.
- The size is under the limit.

## Finish

- Copy the working file over the report in `runvil-design`.
- Move the cabinet copies of the 26 evidence files to `cabinet/_trash/`. The copies in `runvil-design/research/evidence/` are byte-identical.
- One commit on `master` with the report and the evidence files, then push. The long version stays in git history at `1099650` and in `cabinet/_output`.
- If a check fails, nothing is committed and I report why.
