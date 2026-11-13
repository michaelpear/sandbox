# MatrixReq REST notes for empty-block trim

Trims field-end empty blocks, between-section empty `div` or `p` spacers, and leading/trailing empty `p` inside `td`/`th`.

Base `https://nexus.matrixreq.com/rest/1`

```
Authorization: Token <token>
Accept: application/json
Content-Type: application/json
```

Do not persist the token.

## Reads

| Call | Use |
|---|---|
| `GET /{project}/cat/{CAT}` | Category tree plus `fieldList` (labels, ids, fieldType) |
| `GET /{project}/item/{item}` | Full item. Use `DOC-28` / `SRS-61` / `RISK-77`, not the `-vN` suffix |

`cat` payload includes `folder.itemList` and `fieldList[{id,label,fieldType,fieldParam}]`.

Resolve field ids per project from `fieldList`. Known PHYSCADE ids are hints only.

- PHYSCADE_TEST Description — UN 10382, PRS 10392, SRS 10407, CSR 10417
- PHYSCADE_TEST risk2 — RISK 10428, CSRA 10438
- PHYSCADE_V2 HA — Hazard 7843, Cause 7865
- PHYSCADE_V2 Description — UN 7775, PRS 7787, SRS 7818, CSR 7856

## Raw HTML write

Description, HA Hazard, HA Cause, and any other `fieldType=richtext` value is raw HTML.

```json
{
  "reason": "Remove empty richtext blocks (leading/trailing, between-section div or p spacers, table-cell empty p)",
  "onlyThoseFields": 1,
  "failOnCleanup": 0,
  "currentVersion": 1,
  "itemProperties": {
    "fx10392": "<p>Kept content.</p>"
  }
}
```

## DOC Text Block write

DHF `value` is a JSON string. Edit only entries whose `type` is `richtext`. Change `fieldValue` HTML. PUT the whole object string in `itemProperties.fx{id}`.

## RISK / CSRA risk2 write

`value` is JSON with `factors`, `mitigations` (links), `postWeights`. Edit only richtext `factors[].value`. PUT the full object string.

```json
{
  "reason": "Remove empty richtext blocks (leading/trailing, between-section div or p spacers, table-cell empty p)",
  "onlyThoseFields": 1,
  "failOnCleanup": 0,
  "currentVersion": 1,
  "itemProperties": {
    "fx10428": "{\"factors\":[...],\"mitigations\":[...],\"postWeights\":[...]}"
  }
}
```

403 This item has been modified means `currentVersion` is stale. Re-GET `maxVersion` and retry that item.
