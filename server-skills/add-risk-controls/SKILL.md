---
name: add-risk-controls
description: "Add CSR risk-control downlinks to a CSRA or RISK item without clearing risk2 or other field content. Use when the user asks to add risk controls, link CSRs as mitigations, or attach controls to a CSRA or RISK item."
type: workflow
lifecycle: active
---

# Add Risk Controls

Add one or more CSR items as downlinks (risk controls) to a CSRA or RISK item without clearing the risk2 field or any other field content.

## Critical constraint

A form-urlencoded PUT that updates `linksDown` **clears risk2 content** unless `failOnCleanup=0` and `onlyThoseFields=1` are included. Always use the pattern below.

## Procedure

1. `GET /{project}/item/{id}` and note `maxVersion` and current `downLinkList`.
2. Build the new downlink list (existing + new CSR IDs, deduplicated).
3. PUT with **form-urlencoded** body:

```
reason=Add risk controls
linksDown=<comma-separated list>
linksAreComplete=0
failOnCleanup=0
onlyThoseFields=1
currentVersion=<maxVersion>
```

4. `GET` the item again and confirm:
   - the new CSR IDs appear in `downLinkList`
   - `fieldValList` still contains the risk2 field (id 7846 for CSRA; confirm id for RISK) with non-empty content

## Example

```
PUT /PHYSCADE_V2/item/CSRA-5
Content-Type: application/x-www-form-urlencoded

reason=Add CSR-47 as risk control&linksDown=CSR-5,CSR-21,CSR-24,CSR-47,CSR-48&linksAreComplete=0&failOnCleanup=0&onlyThoseFields=1&currentVersion=27
```

## Applies to

- CSRA items (risk2 field id 7846)
- RISK items (confirm risk2 field id on the target project)

## Do not

- Omit `failOnCleanup=0` or `onlyThoseFields=1`
- Use `linksAreComplete=1` for a partial link update
- Rely on version bump alone as proof of success — always re-GET and check field content
