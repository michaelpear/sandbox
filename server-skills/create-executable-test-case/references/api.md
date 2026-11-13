# MatrixReq API shapes for XTC creation

Base: `https://nexus.matrixreq.com/rest/1`  
Auth header: `Authorization: Token <token>`

## Create folder

```
POST /{project}/folder
Content-Type: application/x-www-form-urlencoded

label=<folder title>
parent=<parent folder id, e.g. F-XTC-107>
reason=<short reason>
```

Response: `{"itemId":..., "serial":109}` → folder id `F-XTC-109`

## Create XTC item

```
POST /{project}/item
Content-Type: application/x-www-form-urlencoded

title=<TC title> (TC-xx)
folder=<new subfolder id>
reason=Create XTC from TC-xx
```

Response: `{"itemId":..., "serial":140}` → item id `XTC-140`

## Set uplink

```
PUT /{project}/item/XTC-140
Content-Type: application/x-www-form-urlencoded

reason=Link to source TC-22
linksUp=TC-22
linksAreComplete=0
```

`linksUp` is a comma-delimited list. `linksAreComplete=0` appends (default).  
Confirm with `GET /{project}/item/XTC-140` → `upLinkList` contains the TC.

## Populate test steps

TC Steps field (PHYSCADE_V2): id `7815`, type `test_steps`  
XTC Test Case Steps field: id `7839`, type `test_steps_result`

1. `GET /{project}/item/TC-xx` → extract `fieldValList.fieldVal` where `id=7815`. Value is JSON array:
   ```json
   [{"ReqID":"SRS-6","action":"<p>...</p>","expected":"<p>...</p>"}, ...]
   ```
2. Map to XTC shape (add empty execution fields):
   ```json
   [{"ReqID":"...","action":"...","expected":"...","result":"","human":"","render":"","comment":""}, ...]
   ```
3. `GET /{project}/item/XTC-xx` for `maxVersion`.
4. `PUT /{project}/item/XTC-xx` (JSON body):
   ```json
   {
     "reason": "Populate test steps from TC-xx",
     "onlyThoseFields": 1,
     "failOnCleanup": 0,
     "currentVersion": 1,
     "itemProperties": {
       "fx7839": "[{\"ReqID\":\"...\",...}]"
     }
   }
   ```
   The value of `fx7839` must be a JSON **string** containing the step array.

On `403 This item has been modified`, re-GET `maxVersion` and retry that item only.

## Copy additional fields

PHYSCADE_V2 field IDs:

| TC field | id | XTC field | id |
|----------|----|-----------|----|
| Description | 7798 | Description | 7838 |
| Test Setup | 7809 | (append to Description) | 7838 |
| Test Method | 7799 | (append to Description) | 7838 |
| Test Level | 7806 | (append to Description) | 7838 |
| Test Files | 7805 | Test Input Files | 9990 |

Build XTC Description HTML:

1. Start with TC Description HTML (if non-empty).
2. If Test Setup is non-empty, append `<p><strong>Test Setup</strong></p>` + setup HTML.
3. If Test Method or Test Level is present, append `<p><strong>Test Method: … &nbsp;|&nbsp; Test Level: …</strong></p>` (include only the parts that exist).

Copy Test Files only when the value is not `[]`, empty, or null.

PUT (JSON) with both fields when applicable:

```json
{
  "reason": "Copy Description/Setup/Method/Level from TC-xx",
  "onlyThoseFields": 1,
  "failOnCleanup": 0,
  "currentVersion": 1,
  "itemProperties": {
    "fx7838": "<p>…description…</p><p><strong>Test Setup</strong></p>…<p><strong>Test Method: Functional &nbsp;|&nbsp; Test Level: system</strong></p>",
    "fx9990": "[]"
  }
}
```

Omit `fx9990` when Test Files is empty. Omit the whole PUT if none of the source fields have content.
