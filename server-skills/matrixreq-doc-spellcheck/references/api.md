# MatrixReq REST notes for spellcheck

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
| `GET /{project}/cat/{CAT}` | Category tree plus `fieldList` (labels and ids) |
| `GET /{project}/item/{item}` | Full item. Use `DOC-28` / `SRS-61` / `RISK-77`, not the `-vN` suffix |

`cat` payload includes `folder.itemList` and `fieldList[{id,label,fieldType,fieldParam}]`.

## DOC Text Block write

DHF `value` is a JSON string. PUT the whole object string in `itemProperties.fx{id}`.

## Requirement Title / Description write

Description is raw HTML in `itemProperties.fx{id}`. Title is top-level `title`.

PHYSCADE_TEST Description ids — UN 10382, PRS 10392, SRS 10407, CSR 10417.

## HA Hazard / Cause write

Hazard and Cause are raw HTML in `itemProperties.fx{id}`. Resolve ids from `GET /{project}/cat/HA` `fieldList` by label.

PHYSCADE_V2 ids — Hazard 7843, Cause 7865.

Skip Harm and Severity dropdowns and the Risks links field.

## RISK / CSRA risk2 write

`value` is JSON with `factors`, `mitigations` (links), `postWeights`. Edit only richtext `factors[].value`. PUT the full object string.

```json
{
  "reason": "Spellcheck risk2 richtext subfields",
  "onlyThoseFields": 1,
  "failOnCleanup": 0,
  "currentVersion": 1,
  "itemProperties": {
    "fx10428": "{\"factors\":[...],\"mitigations\":[...],\"postWeights\":[...]}"
  }
}
```

PHYSCADE_TEST risk2 ids — RISK 10428, CSRA 10438.

403 This item has been modified means currentVersion is stale.
