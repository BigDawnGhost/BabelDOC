# One-page reproduction: spaced TOC dot leaders

`pml-page-one.pdf` contains only the first page of Steven G. Johnson's
*Notes on Perfectly Matched Layers (PMLs)*, updated August 12, 2021.
Source: <https://arxiv.org/abs/2108.05348>.

The excerpt is shared to reproduce BabelDOC's TOC paragraph detection bug.
Copyright remains with the original author. No translated output, credentials,
local logs, embedded files, annotations, or other pages are included.

## Affected entries

The original first-page TOC contains spaced dot leaders. For example, entries
3.2 and 3.5 each have 13 leader dots. A rule requiring 20 consecutive dots
does not recognize them.

On BabelDOC v0.7.1 / upstream commit
`dd9adc352b957533d05b8cb94e88a7589cc56a35`, the Rust paragraph finder can
merge adjacent entries before translation. Styles/formulas processing then
combines the text across those line boundaries.

The proposed scoped fix recognizes at least 10 dots with internal horizontal
spacing while retaining the existing 20-dot threshold for contiguous dots.
It also prevents the alternating-line-number merge from rejoining recognized
directory entries across independent page-number paragraphs.

Expected independent subentries after both paragraph finding and style
processing: 3.1, 3.2, 3.3, 3.4, 3.5, 4.1, 4.2, 7.1, 7.2, 7.3.

## Reproduction

Using a local checkout with dependencies and BabelDOC assets installed:

```bash
uv run babeldoc \
  --openai \
  --openai-api-key ci-dummy-key \
  --files pml-page-one.pdf \
  --output /tmp/babeldoc-toc-repro \
  --debug \
  --skip-translation \
  --skip-scanned-detection
```

`--skip-translation` keeps layout analysis, paragraph finding, and style
processing enabled while skipping translation and glossary extraction. The
dummy key is not a credential and no translation request is required. Do not
substitute `--only-parse-generate-pdf`: it also bypasses paragraph finding
and would not exercise this bug.

Inspect the debug paragraph-finding and style-processing snapshots to check
the entry boundaries. This does not test translation quality, dot-leader
redrawing, or page-number alignment.

This reproduction lives on a separate fork branch so the upstream source
patch does not need to add a third-party PDF fixture.
