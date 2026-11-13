---
name: draft-test-report-discussion
description: "Draft a Discussion section for a MatrixReq test report (DOC) by reviewing XTCs in a folder, separating documentation deviations from actual system issues, and preserving GitHub and attachment hyperlinks. Use when the user asks to draft a Discussion section, summarize deviations and failures for a test report, or review XTCs for documentation vs system issues."
type: workflow
lifecycle: active
---

# Draft Test Report Discussion

Review executed XTCs in a named folder and draft a Discussion section suitable for a MatrixReq DOC Test Result report. Separate documentation deviations from real system issues. Preserve all hyperlinks (GitHub issues, file attachments) as clickable `<a>` tags.

Read-only until the user approves the exact text for insertion. Never edit a DOC item without showing the draft and receiving explicit approval.

## Inputs required

1. Folder of XTCs (e.g. `F-XTC-109`).
2. Optional target DOC item (e.g. `DOC-47`) and field name (default “Discussion”).
3. Project short label (default `PHYSCADE_V2`).
4. Token — use the one already in the conversation; do not store it. Ask if missing.

## Defaults (PHYSCADE_V2)

| Item | Value |
|------|-------|
| REST base | `https://nexus.matrixreq.com/rest/1` |
| Auth | `Authorization: Token <token>` |
| Test Run Result | field id `7831` |
| Reviewer Comments | field id `7864` |
| Test Steps | field id `7839` |
| Result codes of interest | `pd` (Passed with Deviation), `pf` (Passed with Exception), `f` (Failed), and any other non-`p` |

## Procedure

1. **Resolve XTCs**  
   Walk `GET /{project}/tree`, find the named folder under the XTC category, and collect every direct `XTC-*` child.

2. **Collect findings**  
   For each XTC:  
   `GET /{project}/item/{XTC-nnn}`  
   - Extract Test Run Result (`7831`).  
   - Extract Reviewer Comments (`7864`) as raw HTML (do not strip `<a>` tags).  
   - Parse Test Steps (`7839`). For every step whose `result` is non-empty and not exactly `p`, record the step number, result code, and raw comment HTML.  
   Focus on items whose overall result is not `p`.

3. **Classify**  
   Using reviewer comments and step comments, place each non-passed item into one of two groups:  
   - **Documentation deviations** — system behaved as intended; discrepancy is due to outdated/incorrect expected results, images from a prior data set, missing step status, or test-case wording that does not match implemented behavior.  
   - **System issues** — observed behavior differs from intended requirements. Capture any GitHub issue links and attachment links exactly as they appear in the source HTML.

4. **Draft Discussion**  
   Produce HTML suitable for a richtext field:  
   - Short intro stating the purpose (document deviations needed to complete tests because of documentation errors; identify failures that point to real system problems).  
   - **Documentation deviations** — bullet list of XTCs with a concise explanation and the recommended test-case correction.  
   - **System issues** — bullet list of XTCs. For each GitHub issue or attachment, emit a real `<a href="..." target="_blank" rel="noopener">label</a>` using the original URL. Do not reduce links to plain text.  
   - Closing sentence that remaining tests in the folder passed without deviation or exception.

5. **Present for review**  
   Show the draft to the user. Stop and wait for approval or edits. Do not write to any DOC item yet.

6. **Insert (only after approval)**  
   If the user approves and names a DOC item and field:  
   - Re-GET the DOC for `maxVersion` and the current field JSON structure.  
   - Replace only the `fieldValue` with the approved HTML.  
   - `PUT` with `onlyThoseFields=1`, the correct `fx{fieldId}`, and a short reason.  
   - Re-GET and confirm the hyperlinks are present.

## Hyperlink rules

- Always preserve original `href` values.  
- Preferred form: `<a href="URL" target="_blank" rel="noopener">visible label</a>`.  
- Common sources: Reviewer Comments on the XTC, and step comments (especially for objective-evidence attachments).  
- If a link was stripped in an earlier draft, re-fetch the source XTC and restore it.

## Error handling

| Situation | Action |
|-----------|--------|
| Folder empty | Report that no XTCs were found and stop |
| Missing token | Ask the user; never store the token |
| No non-passed results | Draft a short note that all tests passed without deviation or exception |
| DOC field missing | List available richtext fields and ask which one to use |

Do not store tokens, passwords, or item payloads in the skill, memory, or artifacts.
