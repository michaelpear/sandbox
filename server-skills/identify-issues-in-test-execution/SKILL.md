---
name: identify-issues-in-test-execution
description: "Review a MatrixReq XTC for test steps whose result status is present but not passed (not exactly p), and output a table of step number, result status, and comment. Use when the user asks to identify issues in test execution, find non-passed steps, or review XTC results. Empty results are handled by the missed-step-status skill."
type: workflow
lifecycle: active
---

# Identify Issues in Test Execution

Review an XTC item and report every test step whose result status is present but not passed. Produce a markdown table of step number, result status, and comment. This skill is read-only; never modify items.

Empty or missing results are **not** reported here; use the `review-test-cases-for-missed-step-status` skill for those.

## Inputs required

1. XTC item reference (e.g. `XTC-140`).
2. Project short label (default `PHYSCADE_V2` if not given).
3. Token — use the one already in the conversation; do not store it. Ask if missing.

## Defaults (PHYSCADE_V2)

| Item | Value |
|------|-------|
| REST base | `https://nexus.matrixreq.com/rest/1` |
| Auth | `Authorization: Token <token>` |
| Test Case Steps field | id `7839`, type `test_steps_result` |

A step is reported as an **issue** only when its `result` value is non-empty (after stripping whitespace) **and** is not exactly `p` (case-sensitive).  

Result codes are expanded for display using this mapping (unknown codes are shown as-is):

| Code | Display name |
|------|--------------|
| `f`  | failed       |
| `d`  | deviation    |
| `c`  | completed    |
| `r`  | not executed |
| `p`  | passed (excluded) |

Empty results are excluded.

## Procedure

1. **Fetch the item**  
   `GET /{project}/item/{XTC-nnn}`  
   Note the title, itemRef, and maxVersion. Extract field `7839`.

2. **Identify non-passed steps with a status**  
   Parse the JSON array of steps.  
   Collect every step (1-based index) where `result` is non-empty after stripping whitespace and is not exactly `p`.

3. **Clean comments for display**  
   - Extract every `<img src="URL">` and turn each into a markdown hyperlink `[Image N](URL)` (N starts at 1 per step).  
   - Strip remaining HTML tags.  
   - Convert `&nbsp;` to spaces and `&` to `&`.  
   - Collapse whitespace.  
   Prepend the image links (if any) to the cleaned text.  
   If both are empty, show `(empty)`.

4. **Report**  
   Output:
   - Item title, itemRef, and version (or folder summary)
   - Total step count
   - A markdown table with columns: **XTC** | **Step** | **Result** | **Comment**  
     (Result uses the expanded name)
   - If no such issues exist, state that clearly.

5. **Stop**  
   Do not write any updates. Offer to examine another XTC if useful.

## Error handling

| Situation | Action |
|-----------|--------|
| Item not found | Report the error and stop |
| No steps field | Report that the item has no test steps |
| Missing token | Ask the user; never store the token |

Do not store tokens, passwords, or item payloads in the skill, memory, or artifacts.
