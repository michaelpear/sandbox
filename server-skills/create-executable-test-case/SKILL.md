---
name: create-executable-test-case
description: "Create MatrixReq XTC (execute test case) items from TC test cases in named folders, placing them in matching subfolders under a destination folder, copying test steps (ReqID/action/expected locked, result/comment open), Description, Test Setup, Test Method, Test Level, and Test Files, and setting uplinks. Use when the user asks to create executable test cases, XTC from TC, or generate test runs from test cases."
type: workflow
lifecycle: active
---

# Create Executable Test Cases (XTC from TC)

Create XTC items from TC items in one or more source folders. Place them in matching subfolders under a user-specified destination folder. Copy test steps (ReqID, action, expected) and add empty result/comment columns. Also copy Description, Test Setup, Test Method, Test Level, and Test Files. Set uplink from each XTC to its source TC.

## Inputs required

1. Project short label (default `PHYSCADE_V2` if not given).
2. One or more source TC folders — by ID (`F-TC-10`) or title.
3. Destination parent folder — by ID (`F-XTC-107`) or title.
4. Token — use the one already in the conversation; do not store it. Ask if missing.

Ignore any items or folders titled “Out of Scope”.

## Defaults (PHYSCADE_V2)

| Item | Value |
|------|-------|
| REST base | `https://nexus.matrixreq.com/rest/1` |
| Auth | `Authorization: Token <token>` |
| TC Steps field | id `7815`, type `test_steps` |
| TC Description | id `7798`, type `richtext` |
| TC Test Setup | id `7809`, type `richtext` |
| TC Test Method | id `7799`, type `dropdown` |
| TC Test Level | id `7806`, type `dropdown` |
| TC Test Files | id `7805`, type `fileManager` |
| XTC Test Case Steps | id `7839`, type `test_steps_result` |
| XTC Description | id `7838`, type `richtext` |
| XTC Test Input Files | id `9990`, type `fileManager` |
| XTC title pattern | `<TC title> (TC-xx)` |

Field IDs may differ on other projects — resolve from `GET /{project}/cat/TC` and `GET /{project}/cat/XTC` if the project is not PHYSCADE_V2.

## Procedure

1. **Resolve folders**  
   `GET /{project}/cat/TC` and walk `folder.itemList` to find the source folders by ID or exact title. Collect every non-folder `TC-*` item. Skip anything under an “Out of Scope” folder.

2. **Create matching subfolders** under the destination  
   For each source folder title:  
   `POST /{project}/folder` (form)  
   - `label` = source folder title  
   - `parent` = destination folder ID  
   - `reason` = short note  
   Response contains `serial` → new folder is `F-XTC-{serial}` (or the category prefix of the destination).

3. **Create XTC items**  
   For each TC:  
   `POST /{project}/item` (form)  
   - `title` = `{TC title} ({TC-xx})`  
   - `folder` = the matching new subfolder ID  
   - `reason` = `Create XTC from {TC-xx}`  
   Response contains `serial` → `XTC-{serial}`.

4. **Set uplink**  
   `PUT /{project}/item/{XTC}` (form)  
   - `reason` = `Link to source {TC-xx}`  
   - `linksUp` = the TC id (comma-delimited if more than one)  
   - `linksAreComplete` = `0` (append)  
   Verify with a GET that `upLinkList` contains the TC.

5. **Copy test steps**  
   - `GET /{project}/item/{TC}` and extract field id `7815` (or the `test_steps` field). Value is a JSON array of `{ReqID, action, expected, ...}`.  
   - Build XTC steps: keep `ReqID`, `action`, `expected`; add empty `result`, `human`, `render`, `comment`.  
   - `GET /{project}/item/{XTC}` for `maxVersion`.  
   - `PUT /{project}/item/{XTC}` (JSON) with `itemProperties.fx7839` = JSON string of the step array.  
   Skip items whose TC has no steps.

6. **Copy additional fields**  
   From the same TC GET, map:
   - TC Description (`7798`) → XTC Description (`7838`)
   - TC Test Setup (`7809`) → append to XTC Description under a `<p><strong>Test Setup</strong></p>` heading
   - TC Test Method (`7799`) and Test Level (`7806`) → append to XTC Description as a bold metadata line: `Test Method: … | Test Level: …`
   - TC Test Files (`7805`) → XTC Test Input Files (`9990`) only when the value is non-empty (not `[]` or blank)

   Build the Description HTML by concatenating the parts that exist, then PUT with `onlyThoseFields=1`, `failOnCleanup=0`, and current `maxVersion`. Include both `fx7838` and `fx9990` in the same PUT when both apply. If none of these fields have content, skip the update and note it.

7. **Report**  
   Summarize folders created, item counts, any TCs that had no steps, and any that had no extra fields to copy. Offer a verification GET of one sample XTC.

## Error handling

| Situation | Action |
|-----------|--------|
| 403 “This item has been modified” | Re-GET `maxVersion` and retry that item only |
| Folder already exists with same title | Reuse it; do not create a duplicate |
| TC has empty steps | Create the XTC and uplink, note “no steps” |
| TC has no Description/Setup/Method/Level/Files | Skip the extra-fields PUT, note “no extra fields” |
| Missing token | Ask the user; never store the token |

## API notes

See `references/api.md` for the exact request shapes used above.

Do not store tokens, passwords, or item payloads in the skill, memory, or artifacts.
