---
name: matrix-remove-empty-lines
description: Remove leading and trailing empty paragraphs from richtext fields in named MatrixReq categories and project, plus empty div-nbsp or p-nbsp spacers between content sections and leading or trailing empty p-nbsp inside table cells. Use when the user asks to remove empty lines, strip empty paragraphs, trim leading or trailing blank lines, remove div or p spacers between sections, clean empty paragraphs in table cells, or clean leading or trailing blank HTML in Matrix items. Triggers include remove empty lines from categories in project, trim empty paragraphs, and clean leading or trailing blank HTML in Matrix items.
---

# Matrix remove empty lines

Strip empty blocks that add blank lines in richtext fields. Do not rewrite wording. Do not touch the existing `matrixreq-doc-spellcheck` skill. Never write to Matrix until the user approves.

Instance defaults — override if the user names another host.

- REST base `https://nexus.matrixreq.com/rest/1`
- Docs `https://docs.matrixreq.com/en/`
- Swagger 2.5 `https://app.swaggerhub.com/apis/matrixreq/MatrixALM_QMS/2.5`
- Auth header `Authorization: Token <token>`
- Do not store tokens or passwords in skills, memory, or artifacts. Ask for a token if the current conversation does not already have one.

## Inputs if missing

1. Project short label (example `PHYSCADE_TEST` or `PHYSCADE_V2`).
2. One or more category labels (example `UN, PRS, SRS, CSR` or `DOC, HA, RISK`).
3. Optional selection — all items in those categories (default), title contains a string, folder id/title, or explicit item IDs (`SRS-61`, `HA-53`).
4. Review before commit (default yes).

## What to edit

Resolve fields from `GET /{project}/cat/{CAT}` `fieldList`. Edit only values that are HTML-bearing richtext.

Treat as richtext

- `fieldType=richtext` — raw HTML (requirement Description, HA Hazard, HA Cause).
- `fieldType=dhf` whose parsed JSON `type` is `richtext` — edit only `fieldValue` HTML (DOC Text Blocks).
- `fieldType=risk2` — edit only `factors[i].value` where `inputType` is `richtext`.

Skip

- Titles, dropdowns, selects, item links, labels, attachments, audit trails, document options, item tables.
- RISK/CSRA `mitigations` links and `postWeights`.
- HA Harm / Severity and Risks links.
- Empty fields (nothing to trim).
- Fields whose HTML has none of the removable constructs below.

Do not spellcheck or restyle content. If a field also has spelling issues, leave them; this skill only removes the empty constructs listed here.

## Empty block definition

An empty block is HTML that adds no visible text. Match case-insensitively, including nested or repeated forms

- `<p></p>`, `<p/>`
- `<p><br></p>`, `<p><br/></p>`, `<p><br /></p>`
- `<p>&nbsp;</p>`, `<p>&#160;</p>`, `<p>&nbsp;<br></p>`
- `<p> </p>` (whitespace only)
- `<p><strong></strong></p>`, `<p><!-- comment --></p>` (no visible text)
- lone `<br>`, `<br/>`, `<div><br></div>`, `<div></div>`
- `<div>&nbsp;</div>`, `<div>&#160;</div>`

Also treat a `p` or `div` as empty when it has attributes but its body is only whitespace, `&nbsp;`, empty spans/strong, HTML comments, or `br`.

An image, table with visible cell text, or non-empty list is visible content.

## What to remove

1. Field ends — remove all consecutive empty blocks from the **start** until the first remaining node has visible text, and from the **end** until the last remaining node has visible text (or the field becomes empty).

2. Between-section spacers — remove a sibling empty `<div>&nbsp;</div>` or empty `<p>&nbsp;</p>` (or empty-div / empty-p equivalent) that sits **between** two non-empty content sections (other `div`, `p`, `table`, `ul`/`ol`, headings). PHYSCADE_V2 examples were SRS-139, SRS-140 (div) and DOC-41/DOC-43/DOC-44 Protocol Overview (p after tables). Do not remove an empty `<p>` that sits between two non-empty paragraphs **inside the same table cell** (rule 3 still applies for cells).

3. Table cells — inside each `td`/`th`, remove leading and trailing empty `<p>&nbsp;</p>` (or empty-`p` equivalent) until the first/last remaining child in that cell has visible text. PHYSCADE_V2 examples were SRS-102 and SRS-123. Do not delete an empty `p` that sits between two non-empty blocks **in the same cell**. Do not drop a row or cell that still has other content. An entire row whose cells become empty is out of scope unless the user asks.

Normalize nothing else — keep existing tags, entities, and spacing in the kept HTML. After trim, if the value is unchanged, skip the item/field.

Show the user a short before/after head and tail (first and last ~120 characters of the HTML) so the dropped tags are obvious. Report counts per field — leading blocks, trailing blocks, between-section `div` or `p` spacers, and cell leading/trailing empty `p`.

## Scan

1. `GET /{project}/cat/{CAT}` for each requested category. Walk `folder.itemList` (and nested folders). Apply the selection rule.
2. For listed items, `GET /{project}/item/{ITEM}` using the id without `-vN` (example `SRS-61`).
3. Map each richtext-bearing field as above. Parse DHF and risk2 JSON when needed.
4. Compute trimmed HTML per field or subfield. Collect only items that would change.

Present item id, title, category, field/subfield, counts by construct, before/after snippets, and a one-line reason. Then `ask_user_question` multi-select (Apply all plus one option per item, or per field if an item has several).

For large scans, list only items with proposed trims.

## Commit after approval

`PUT /{project}/item/{item}` with `reason`, `onlyThoseFields=1`, `failOnCleanup=0`, `currentVersion` from GET `maxVersion`, and `itemProperties.fx{fieldId}`. Use `itemProperties`, not a top-level `fx` key.

Write shapes

- Raw HTML fields — string HTML in `itemProperties.fx{id}`.
- DHF Text Blocks — PUT the **full dhf JSON string** with updated `fieldValue`.
- risk2 — PUT the **entire risk2 JSON string** with updated richtext `factors[].value`.

Reason text example — `Remove empty richtext blocks (leading/trailing, between-section div or p spacers, table-cell empty p)`. Re-GET to confirm. On `403` This item has been modified, re-GET and retry that item only.

Details in `references/api.md`. Copy-paste prompts in `references/prompt.md`. If the user asks how to run this later, quote that file.
