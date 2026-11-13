---
name: create-test-case
description: "Create or update a MatrixReq TC test case item: create in a named folder (creating the folder if needed), set Description and Test Setup, and define or revise test steps (including numbered-list actions and file references). Use when the user asks to create a test case, new TC, add or replace steps, update Description or Test Setup, or define test case steps from ReqID / Action / Expected Result descriptions."
type: workflow
lifecycle: active
---

# Create or Update Test Case

Create a new TC item (or update an existing one) in PHYSCADE_V2 and populate Description, Test Setup, and Test Steps from user descriptions.

## Inputs required

1. Project short label (default `PHYSCADE_V2` if not given).
2. Destination folder — by ID (`F-TC-20`) or title (e.g. `Experiment`). Create the folder if it does not exist.
3. Title for the new TC (or existing item ID such as `TC-93` when updating).
4. Optional: Description text.
5. Optional: Test Setup text (may include file lists).
6. Steps: list of `{ReqID, Action, Expected Result}`. ReqID may be blank. Action and Expected may be plain text or HTML (numbered lists, file names, etc.).
7. Token — use the one already in the conversation; do not store it. Ask if missing.

## Defaults (PHYSCADE_V2)

| Item | Value |
|------|-------|
| REST base | `https://nexus.matrixreq.com/rest/1` |
| Auth | `Authorization: Token <token>` |
| TC category | `TC` |
| Description field | id `7798`, type `richtext` |
| Test Setup field | id `7809`, type `richtext` |
| Test Steps field | id `7815`, type `test_steps` |
| Root TC folder | `F-TC-1` |

Field IDs may differ on other projects — resolve by inspecting an existing TC item (`GET /{project}/item/TC-xx`) if the project is not PHYSCADE_V2.

## Procedure

1. **Resolve or create destination folder** (new items only)  
   - `GET /{project}/cat/TC` (or `/tree`) and walk folders to find an exact title match or ID.  
   - If missing, `POST /{project}/folder` (form):  
     - `label` = folder title  
     - `parent` = `F-TC-1` (or other parent)  
     - `reason` = short note  
   - Response `serial` → new folder is `F-TC-{serial}`.

2. **Create the TC item** (if new)  
   `POST /{project}/item` (form):  
   - `title` = given title  
   - `folder` = destination folder ID  
   - `reason` = `Create test case`  
   Response `serial` → `TC-{serial}`.

3. **Set or update Description** (field `7798`)  
   - Wrap plain text in `<p>...</p>`.  
   - `GET` for `maxVersion`, then `PUT` JSON with `onlyThoseFields=1`, `failOnCleanup=0`, and `itemProperties.fx7798`.

4. **Set or update Test Setup** (field `7809`)  
   - Build HTML. For a file list, use a leading `<p>` instruction followed by `<ul><li>filename</li>...</ul>`.  
   - Append to existing content if the user says “add” rather than “replace”.  
   - Same GET-then-PUT pattern as Description.

5. **Build and set steps** (field `7815`)  
   - Construct a JSON array of objects:  
     `{"ReqID": "<id or empty>", "action": "<html>", "expected": "<html>"}`  
   - For multi-action steps, use a numbered list:  
     `<ol><li>Load the song named NG_dir_000_player_mV.hex for direction 0</li><li>Analyze to acquire and process the data</li></ol>`  
   - Wrap simple text in `<p>...</p>`.  
   - When replacing or appending, `GET` the current steps, modify the array, then `PUT` with the full new array in `itemProperties.fx7815`.  
   - Always re-GET `maxVersion` immediately before the PUT.

6. **Verify**  
   `GET /{project}/item/{TC-xx}` and confirm the fields contain the expected content.

7. **Report**  
   Return the item ID, folder, title, step count, and which fields were updated. Offer further edits.

## Error handling

| Situation | Action |
|-----------|--------|
| 403 “This item has been modified” | Re-GET `maxVersion` and retry that PUT only |
| Folder already exists with same title | Reuse it; do not create a duplicate |
| Missing token | Ask the user; never store the token |
| Steps / field ID unknown | Inspect an existing TC via GET and locate the field |

## API notes

- Form endpoints (`POST /folder`, `POST /item`) use multipart form data (`-F`).  
- Field updates use JSON body with `itemProperties.fx{fieldId}`.  
- Do not store tokens, passwords, or item payloads in the skill, memory, or artifacts.

## Example patterns

**Simple step**  
```json
{"ReqID": "", "action": "<p>Action for Step 1</p>", "expected": "<p>Expecting Step 1 action completed</p>"}
```

**Numbered-list action with file reference**  
```json
{"ReqID": "", "action": "<ol><li>Load the song named NG_dir_000_player_mV.hex for direction 0</li><li>Analyze to acquire and process the data</li></ol>", "expected": "<p>Summary view displays with corresponding direction bump and \"8\" segments in the corresponding summary circle for direction 0.</p>"}
```

**Test Setup with file list**  
```html
<p>Transfer files to system with Player USB attached.</p>
<ul>
  <li>NG_dir_000_player_mV.hex</li>
  <li>NG_dir_045_player_mV.hex</li>
  ...
</ul>
```
