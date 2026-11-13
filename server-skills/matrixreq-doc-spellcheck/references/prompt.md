# How to run this skill

In a later chat, paste one of the templates below, or ask:

- Show me the prompt examples for the Matrix spellcheck skill.
- How do I run the DOC, requirements, hazard, or risk spellcheck?

Replace PROJECT_LABEL with the project short name (example PHYSCADE_TEST). Send an API token if that conversation does not already have one. Tokens are not stored in this skill.

Default behavior for every mode: scan, list only items with issues, explain the exact characters that change, offer approve options, and do not write to Matrix until you approve.

---

## Documents (DOC Text Blocks)

Spellcheck DOC Text Blocks in project PROJECT_LABEL.
Select items whose titles contain "V2.0".
Show the exact characters that change, offer approve options, and do not write until I approve.

Spellcheck DOC Text Blocks in project PROJECT_LABEL for items in folder FOLDER.
Review first.

Spellcheck DOC Text Blocks in project PROJECT_LABEL for all DOC items.
List only documents that have errors.

---

## Requirements (Title and Description)

Spellcheck Title and Description on UN, PRS, SRS, and CSR items in project PROJECT_LABEL.
List only items with issues. Explain each change and let me approve individually.

Spellcheck Title and Description for SRS items in project PROJECT_LABEL in folder FOLDER_NAME.

---

## Hazards (HA Hazard and Cause)

This mode scans only the richtext Hazard and Cause fields on HA items. It does not scan Harm or Severity dropdowns, Risks links, Labels, or the item title unless you ask.

Spellcheck Hazard and Cause on HA items in project PROJECT_LABEL.
List only items with issues. Explain each change and let me approve individually.

Spellcheck Hazard and Cause on HA items in project PROJECT_LABEL in folder FOLDER_NAME.

Spellcheck Hazard and Cause on HA-53, HA-54 in project PROJECT_LABEL.

---

## Risks (RISK and CSRA risk2 richtext)

This mode scans only richtext subfields inside the field whose type is risk2.

RISK subfields: Hazard, Sequence of Events, Root Cause of Hazard, Mitigations.
CSRA subfields: Threat Description, Vulnerability Involved, Affected Assets, Mitigation Description.

It does not scan Harm dropdowns, likelihood or impact scores, mitigation links, Justification / Comments, or other sibling fields unless you ask.

It does flag spelling, product-name capitalization (Physcade vs PhysCade), broken stock phrases (fails to for acquisition, waits the required ... is collected), and clear grammar that changes meaning. It keeps domain tokens (SAM, IFU, CRC16, MCU, UWF, horsetail, featureless, coMAP, coPILOT).

Full category scan

Spellcheck risk2 richtext subfields on RISK and CSRA items in project PROJECT_LABEL.
Only Hazard, Sequence of Events, Root Cause, and Mitigations on RISK, and Threat Description, Vulnerability Involved, Affected Assets, and Mitigation Description on CSRA.
List only items with issues. Explain the exact characters that change and let me approve individually.

Shorter

Spellcheck RISK and CSRA richtext risk fields in project PROJECT_LABEL.
Review first with approve options.

One category or folder

Spellcheck risk2 richtext on RISK items in project PROJECT_LABEL.

Spellcheck risk2 richtext on RISK items in project PROJECT_LABEL in folder FOLDER_NAME.

Named items

Spellcheck risk2 richtext on CSRA-6, CSRA-7, CSRA-10 in project PROJECT_LABEL.

---

## Scan only (any mode)

Scan-only spellcheck of RISK and CSRA risk2 richtext in PROJECT_LABEL.
Do not update Matrix.

Scan-only spellcheck of UN/PRS/SRS/CSR Title and Description in PROJECT_LABEL.
Do not update Matrix.

Scan-only spellcheck of HA Hazard and Cause in PROJECT_LABEL.
Do not update Matrix.
