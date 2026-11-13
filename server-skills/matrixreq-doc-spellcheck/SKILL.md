---
name: matrixreq-doc-spellcheck
description: Spellcheck MatrixReq DOC Text Blocks, requirement Title plus Description (UN, PRS, SRS, CSR), HA Hazard plus Cause, and risk2 richtext subfields on RISK and CSRA items, including clear word-form and agreement errors, with review before commit. Use when the user asks to spellcheck Matrix documents, requirements, hazards, HA, risks, CSRA, Text Blocks, or to reuse the PHYSCADE spellcheck tasks on another project or folder.
---

# MatrixReq spellcheck

Four modes, same review-then-commit rule. Never write to Matrix until the user approves. For large scans, list only items that have proposed fixes. Always explain the exact characters that change, then offer `ask_user_question` multi-select approval.

Instance defaults — override if the user names another host.

- REST base `https://nexus.matrixreq.com/rest/1`
- Docs `https://docs.matrixreq.com/en/`
- Swagger 2.5 `https://app.swaggerhub.com/apis/matrixreq/MatrixALM_QMS/2.5`
- Auth header `Authorization: Token <token>`
- Do not store tokens or passwords in skills, memory, or artifacts. Ask for a token if the current conversation does not already have one.

## Inputs if missing

1. Project short label (example `PHYSCADE_TEST`).
2. Mode — Documents, Requirements, Hazards, or Risks.
3. Selection rule — all items in those categories, title contains a string, folder id/title, or explicit IDs.
4. Review before commit (default yes).

## Mode A — DOC Text Blocks

1. `GET /{project}/cat/DOC` and filter.
2. Text Blocks are `fieldType=dhf` whose JSON `type` is `richtext`. Parse `value` as JSON. Only edit `fieldValue` HTML.
3. Skip empty sections, item tables, audit trails, document options, labels, attachments.

PUT the **full dhf JSON string** in `itemProperties.fx{fieldId}`.

## Mode B — requirement Title and Description

Categories UN, PRS, SRS, CSR.

1. `GET /{project}/cat/{CAT}` and walk `folder.itemList`.
2. Resolve Description field id from that category `fieldList` by label `Description`. PHYSCADE_TEST ids were UN `10382`, PRS `10392`, SRS `10407`, CSR `10417`.
3. Description `value` is raw HTML. Title is `item.title`.

PUT title via top-level `title`. PUT Description via `itemProperties.fx{id}` = HTML string.

## Mode C — RISK and CSRA risk2 richtext

Categories RISK and CSRA (project labels may differ; match `fieldType=risk2`).

1. `GET /{project}/cat/RISK` and `GET /{project}/cat/CSRA`. Find the `risk2` field id from `fieldList` (PHYSCADE_TEST RISK `10428` label Risk, CSRA `10438` label Cyber Risk).
2. Item `value` is a JSON object with `factors[]`, `mitigations[]` (item links — do not spellcheck), and `postWeights` (scores — do not spellcheck).
3. Spellcheck only factors whose `inputType` is `richtext`. On PHYSCADE_TEST that is
   - RISK — Hazard (`hazard`), Sequence of Events (`events`), Root Cause of Hazard (`sequence`), Mitigations (`mitigation`)
   - CSRA — Threat Description (`threat`), Vulnerability Involved (`vulnerability`), Affected Assets (`harm`), Mitigation Description (`mitigations`)
4. Skip RISK Harm when `inputType` is `select`. Skip Justification / Comments and other sibling item fields unless the user asks.
5. Edit only `factors[i].value` (HTML or plain text). PUT the **entire risk2 JSON** as a string in `itemProperties.fx{riskFieldId}`.

Risk-content nuances

- Expect short, templated sentences. A small hit list after a full-category scan is normal; say so rather than inventing style nits.
- Flag real spelling (`occurence`, `acqusition`, `commmunications`, `requring`, `Adminstrator`).
- Flag broken stock phrases used on several items (`fails to for acquisition`, `waits the required 4-sec EGM data is collected` → `waits until the required…`).
- Flag product-name capitalization `Physcade` → `PhysCade`.
- Flag clear grammar (word-form and agreement) — see Scan rules. Examples: `confusion results` → `confusing results`; `a USB drives` → `a USB drive`; `keystrokes that modifies` → `keystrokes that modify`.
- Keep domain tokens — SAM, IFU, CRC16, MCU, UWF, RLD, EGM, ECG, PHI, RubberDucky, Rufus, horsetail, featureless, coMAP, coPILOT, VM1000, DB44.
- Do not rewrite risk judgments, scores, harm dropdown values, or mitigation link titles.

## Mode D — HA Hazard and Cause

Category HA (project label may differ; match `fieldList` labels `Hazard` and `Cause` with `fieldType=richtext`).

1. `GET /{project}/cat/HA` and walk `folder.itemList`.
2. Resolve field ids from that category `fieldList` by label. PHYSCADE_V2 ids were Hazard `7843`, Cause `7865`.
3. Both values are raw HTML, same write shape as a requirement Description.
4. Skip Harm and Severity dropdowns, Risks links, and Labels. Do not spellcheck the HA item title unless the user asks.

PUT each edited field via `itemProperties.fx{id}` = HTML string.

## Scan rules (all modes)

Flag spelling and clear punctuation/grammar defects only. Do not flag optional style (hyphenating front end / back end, greyed vs grayed, British vs American, serial comma, tone).

### General spellcheck (required)

Do **not** rely only on a fixed pattern list. Use a dictionary-based spellchecker on every scanned field.

1. Install if missing: `pip install pyspellchecker`
2. Strip HTML, extract tokens (words), and check each against the dictionary.
3. Skip:
   - Domain / product tokens (allowlist below)
   - Numbers, units, identifiers (RA, LA, LL, V1, RLD, USB-C, DB44, CRC16, etc.)
   - Proper nouns already correctly capitalized (PhysCade, coPILOT, coMAP)
4. For any unknown token, propose the top dictionary correction if the edit distance is small (≤2) and the suggestion is unambiguous.
5. Always include the known project patterns (occurence, Physcade, teh, etc.) as a supplement, not a replacement.

Domain allowlist (extend as needed):  
SAM, IFU, CRC16, MCU, UWF, RLD, EGM, ECG, PHI, RubberDucky, Rufus, horsetail, featureless, coMAP, coPILOT, VM1000, DB44, PhysCade, RA, LA, LL, V1, LR, USB-C, DC-Offset, IRB, FW, MCU

Example that must be caught: `dtect` → `detect`.

### Clear grammar — word form and agreement

Flag only high-confidence, low-ambiguity errors that use the wrong form of a word or violate basic agreement. Do not rewrite for style or preference.

| Type | Before | After |
|------|--------|-------|
| Noun used where adjective is required | confusion results | confusing results |
| Number agreement | a USB drives | a USB drive |
| Subject-verb agreement | keystrokes that modifies | keystrokes that modify |
| Missing object / broken stock phrase | connect and ECG leads | connect the ECG leads |
| Missing conjunction / verb form | channels encode as | channels and encode as |

Apply both the dictionary spellcheck and the grammar rules in every mode. Present the exact before/after characters and a one-line reason.

Present item id, title, field/subfield, before snippet, after snippet, and a one-line reason. Then multi-select approve options (Apply all plus one option per item or finding).

## Commit after approval

`PUT /{project}/item/{item}` with `reason`, `onlyThoseFields=1`, `failOnCleanup=0`, `currentVersion` from GET `maxVersion`, and `itemProperties.fx{fieldId}`. Use `itemProperties`, not a top-level `fx` key. Re-GET to confirm.

Details in `references/api.md`. User-facing how-to and copy-paste prompts (including the Risks mode) are in `references/prompt.md`. If the user asks how to run a mode later, quote that file.
