# Changelog

## 2026-10-21
- Initial skill creation
- Documented MatrixReq REST flow for creating a TC folder (if needed), creating a TC item, and writing the Test Steps field (id 7815) as a JSON array of {ReqID, action, expected}
- Validated by creating F-TC-20 / TC-93 with 10 sample steps in PHYSCADE_V2

## 2026-10-21 (update)
- Expanded to cover updates of existing items (Description id 7798, Test Setup id 7809, and step revisions)
- Added patterns for numbered-list actions and explicit file-name references in steps
- Added Test Setup pattern that includes a transfer instruction plus bulleted file list
- Validated on TC-93 “Confirm Direction Display for HD Grid” (direction steps + song file list)
