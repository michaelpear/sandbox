# How to run this skill

In a later chat, paste one of the templates below, or ask to remove empty lines from named categories in a named project.

Replace PROJECT_LABEL with the project short name (example PHYSCADE_TEST). Send an API token if that conversation does not already have one. Tokens are not stored in this skill.

Default behavior — scan, list only fields that would change, show the HTML head/tail that would drop, offer approve options, and do not write to Matrix until you approve.

Removes

- leading and trailing empty paragraphs or empty divs at the field ends
- empty `div` `&nbsp;` spacers between content sections
- leading and trailing empty `p` `&nbsp;` inside table cells

Does not remove an empty `p` between two non-empty paragraphs outside a table cell. Does not change wording.

---

## Named categories

Remove empty lines from categories UN, PRS, SRS, CSR in project PROJECT_LABEL.

Remove empty lines from categories DOC, HA in project PROJECT_LABEL.
Review first.

Remove empty lines from categories RISK, CSRA in project PROJECT_LABEL.
Only the empty-block constructs in this skill. Do not change wording.

---

## Folder or title filter

Remove empty lines from categories SRS in project PROJECT_LABEL in folder FOLDER_NAME.

Remove empty lines from categories DOC in project PROJECT_LABEL for items whose titles contain "V2.0".

---

## Named items

Remove empty lines from categories HA in project PROJECT_LABEL for HA-53, HA-54.

Remove empty lines on SRS-61 in project PROJECT_LABEL.

---

## Scan only

Scan-only remove empty lines from categories UN, PRS, SRS, CSR in project PROJECT_LABEL.
Do not update Matrix.
