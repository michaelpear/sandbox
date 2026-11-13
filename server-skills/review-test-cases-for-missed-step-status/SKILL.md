---
name: review-test-cases-for-missed-step-status
description: "Review a MatrixReq XTC for test steps that have no Passed/Failed (or other) result status recorded, report them, and after approval append a note to Reviewer Comments. Use when the user asks to review test cases for missed step status, find steps without a result, or check for empty result fields on XTC-nnn."
type: workflow
lifecycle: active
---

# Review Test Cases for Missed Step Status

Review an XTC item (or all XTCs in a folder) and report every test step that has no result status recorded (empty or missing `result` value). After showing the findings, wait for explicit user approval, then append a note listing the affected steps to the Reviewer Comments field.

## Inputs required

1. XTC item reference (e.g. `XTC-140`) or a folder ID (e.g. `F-XTC-109`).
2. Project short label (default `PHYSCADE_V2` if not given).
3. Token — use the one already in the conversation; do not store it. Ask if missing.

## Defaults (PHYSCADE_V2)

| Item | Value |
|------|-------|
| REST base | `https://nexus.matrixreq.com/rest/1` |
| Auth | `Authorization: Token <token>` |
| Test Case Steps field | id `7839`, type `test_steps_result` |
| Reviewer Comments field | id `7864`, type `richtext` |

A step has a **missed status** when its `result` value is missing, the empty string, or consists only of whitespace.

## Procedure

1. **Resolve items**  
   - Single XTC: use the given item ID.  
   - Folder: walk the project tree (or `GET /{project}/tree`) to collect every `XTC-*` item directly in that folder. Skip items the user explicitly excludes (e.g. “not executed yet”).

2. **Fetch and identify**  
   For each item:  
   `GET /{project}/item/{XTC-nnn}`  
   Extract field `7839`. Collect every 1-based step where `result` is missing or empty after stripping whitespace.

3. **Clean comments for display**  
   - Replace `<img ...>` tags with `[IMAGE]`  
   - Strip remaining HTML tags  
   - Convert `&nbsp;` to spaces  
   - Collapse whitespace  
   If the cleaned text is empty, show `(empty)`.

4. **Report and ask for approval**  
   Output a markdown table (for a single item: **Step** | **Comment**; for a folder: **XTC** | **Step** | **Comment**).  
   If no missed-status steps exist, state that and stop.  
   Otherwise, show the exact note(s) that will be written and **stop and wait for explicit user approval** before any write.  
   Example note: `Missing result status for steps 18, 19, 20, 24`

5. **Append Reviewer Comments note (after approval)**  
   For each approved item that has missed-status steps:  
   - Re-GET to obtain current `maxVersion` and existing Reviewer Comments (`7864`).  
   - Build the note text: `Missing result status for steps <comma-separated numbers>`  
   - Wrap in `<p>...</p>`.  
   - If the field already has content, append the new `<p>`; otherwise set it as the sole content.  
   - `PUT /{project}/item/{XTC-nnn}` with JSON body:
     ```json
     {
       "reason": "Add reviewer comment noting steps with missing result status",
       "currentVersion": <maxVersion>,
       "onlyThoseFields": 1,
       "itemProperties": {
         "fx7864": "<combined HTML>"
       }
     }
     ```

6. **Confirm**  
   Report the final version of each updated item and the exact note that was appended.

## Error handling

| Situation | Action |
|-----------|--------|
| 403 “This item has been modified” | Re-GET `maxVersion` and retry that PUT only |
| Item not found | Report the error and continue with remaining items |
| No steps field | Note it and skip that item |
| Missing token | Ask the user; never store the token |
| User excludes an item | Skip it |

Do not store tokens, passwords, or item payloads in the skill, memory, or artifacts.
