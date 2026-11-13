---
name: test-summary-report
description: "Generate a summary table of executed test cases (XTCs) in a MatrixReq folder or project, showing ID and title, full Tester and Reviewer names, Test Date, and the display text of the Test Run Result. Use when the user asks for a test summary report, XTC summary table, or test execution overview."
type: workflow
lifecycle: active
---

# Test Summary Report

Produce a markdown table summarizing XTC items. Read-only; never modify items.

## Inputs required

1. Folder ID (e.g. `F-XTC-109`) or project short label. Default project `PHYSCADE_V2`.
2. Token — use the one already in the conversation; do not store it. Ask if missing.

## Defaults (PHYSCADE_V2)

| Item | Value |
|------|-------|
| REST base | `https://nexus.matrixreq.com/rest/1` |
| Auth | `Authorization: Token <token>` |
| Tester | field id `7830` (dropdown, optionSetting `dd_tester_list`) |
| Reviewer | field id `7863` (dropdown, optionSetting `dd_tester_list`) |
| Test Date | field id `7829` |
| Test Run Result | field id `7831` (type `test_result`) |

## Result code mapping

Map the stored code to display text (unknown codes are shown as-is):

| Code | Display text |
|------|--------------|
| `p`  | Passed |
| `f`  | Failed |
| `pd` | Passed with Deviation |
| `pf` | Passed with Exception |
| `d`  | Deviation |
| `c`  | Completed |
| `r`  | Not executed |

## Procedure

1. **Resolve items**  
   - Folder: walk the project tree (`GET /{project}/tree`) and collect every `XTC-*` item directly in the named folder.  
   - If no folder is given, collect all XTCs in the project (or ask for clarification).

2. **Load name mapping**  
   `GET /{project}` and locate the setting whose key is `dd_tester_list`.  
   Parse its JSON value and build a map from option `id` → option `label` (e.g. `MP` → `Pear, M`, `DF` → `Fallon, D.`).  
   Use this map for both Tester and Reviewer. Unknown or empty ids stay as-is (or `—` if empty).

3. **Fetch fields**  
   For each XTC:  
   `GET /{project}/item/{XTC-nnn}`  
   Extract title, and the four fields above.  
   - Tester / Reviewer: look up the stored id in the `dd_tester_list` map and show the full label.  
   - Test Run Result: map the stored code through the result code table above and show the display text.  
   - Test Date: clean HTML tags and excess whitespace.  
   Show `—` for empty fields.

4. **Report**  
   Output a markdown table with columns:  
   **ID** | **Title** | **Tester** | **Reviewer** | **Test Date** | **Test Run Result**

5. **Stop**  
   Do not write any updates.

## Error handling

| Situation | Action |
|-----------|--------|
| Item not found | Skip and note it |
| Missing token | Ask the user; never store the token |
| Folder empty | Report that no XTCs were found |
| `dd_tester_list` missing | Fall back to raw ids for Tester/Reviewer |

Do not store tokens, passwords, or item payloads in the skill, memory, or artifacts.
