---
name: fill-empty-xtc-comments
description: "Fill empty Comments in MatrixReq XTC test steps with None and record the changed step numbers in Reviewer Comments. Use when the user asks to fill empty comments for test XTC-nnn or similar."
type: workflow
lifecycle: active
---

# Fill Empty XTC Comments

Ensure every test step in an XTC has a non-empty Comments field. Empty comments are set to the literal string `None`. After the change, append a note listing the affected step numbers to the Reviewer Comments field.

Always show the list of steps that will be changed and wait for explicit user approval before writing any update. Never edit without approval.

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
| Reviewer Comments field | id `7864`, type `richtext` |

Field IDs may differ on other projects — resolve from `GET /{project}` category list if the project is not PHYSCADE_V2.

## Procedure

1. **Fetch the item**  
   `GET /{project}/item/{XTC-nnn}`  
   Note `maxVersion` and extract field `7839` (JSON array of steps).

2. **Identify empty comments**  
   A comment is empty when its value is missing, the empty string, or consists only of whitespace.  
   Collect the 1-based step numbers that are empty.

3. **Show and ask for approval**  
   Report the list of step numbers that will be set to `None`.  
   Stop and wait for the user to approve the change for this item.  
   If there are no empty comments, report that and stop.

4. **Update steps (after approval)**  
   Set `comment` to the string `None` on every empty step.  
   `PUT /{project}/item/{XTC-nnn}` with JSON body:
   ```json
   {
     "reason": "Fill empty test step Comments with None",
     "currentVersion": <maxVersion>,
     "onlyThoseFields": 1,
     "itemProperties": {
       "fx7839": "<JSON string of the updated steps array>"
     }
   }
   ```
   Re-GET the item and confirm the new `maxVersion`.

5. **Append Reviewer Comments note**  
   Build the note text exactly as:  
   `Empty comments in steps <comma-separated numbers> set to None`  
   (example: `Empty comments in steps 2, 3, 5, 6, 9, 19, 22, 23, 24 set to None`).  
   Wrap it in `<p>...</p>`.  
   If the current Reviewer Comments field (`7864`) already contains content, append the new `<p>` after the existing HTML; otherwise set it as the sole content.  
   `PUT` again with the new `maxVersion`, `onlyThoseFields=1`, and `fx7864` set to the combined HTML.  
   Use a short reason such as `Add reviewer comment noting empty step comments set to None`.

6. **Confirm**  
   Report the final version and the exact note that was written to Reviewer Comments.

## Error handling

| Situation | Action |
|-----------|--------|
| 403 “This item has been modified” | Re-GET `maxVersion` and retry that PUT only |
| No empty comments | Report and stop; do not write anything |
| Missing token | Ask the user; never store the token |
| Field IDs differ | Resolve from project config before writing |

Do not store tokens, passwords, or item payloads in the skill, memory, or artifacts.
