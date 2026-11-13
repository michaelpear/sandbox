---
name: update-risk2-field
description: "Update a richtext factor (Mitigation Description, Threat Description, etc.) inside a MatrixReq CSRA or RISK risk2 field. Use when the user asks to update a mitigation description, edit a risk factor, change risk2 text, or revise threat/vulnerability/asset wording on a CSRA or RISK item."
type: workflow
lifecycle: active
---

# Update risk2 Field

Update one or more richtext factors inside a CSRA or RISK `risk2` field without clearing other content.

## Critical constraints

- **Always show the exact proposed text and wait for user approval** before writing.
- Use the JSON body format below. A form-urlencoded PUT with the wrong field key clears `risk2`.
- Never omit `failOnCleanup=0` and `onlyThoseFields=1`.
- After the write, re-GET and confirm the factor text and that other factors are intact.

## Procedure

1. `GET /{project}/item/{id}` and note `maxVersion`.
2. Locate field id **7846** (CSRA risk2 on PHYSCADE_V2; confirm for other projects). Parse its JSON value.
3. Update the target factor’s `value` (match on `label`, e.g. `"Mitigation Description"`).
4. Show the user the exact new HTML and wait for approval.
5. After approval, PUT with **application/json**:

```json
{
  "reason": "Update mitigation description",
  "onlyThoseFields": 1,
  "failOnCleanup": 0,
  "currentVersion": <maxVersion>,
  "itemProperties": {
    "fx7846": "<entire risk2 JSON string with updated factor>"
  }
}
```

6. `GET` the item again and confirm:
   - the factor text matches the approved HTML
   - other factors and the `mitigations` array are unchanged
   - `fieldValList` still contains field 7846

## Recovery

If a bad PUT empties `fieldValList`, recover the last good risk2 JSON from an earlier version (`GET /{project}/item/{id}-vN`), re-apply the approved change, and PUT again with the correct payload.

## Applies to

- CSRA items (risk2 field id 7846 on PHYSCADE_V2)
- RISK items (confirm risk2 field id on the target project)

## Do not

- Use form-urlencoded with a top-level `field_7846`, `7846`, or `fields` key — these are ignored and can clear content
- Omit `itemProperties.fx{id}`
- Write without explicit user approval of the exact text
