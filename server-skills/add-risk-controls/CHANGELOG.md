# Changelog

## 2026-11-08
- Initial skill created.
- Documents the form-urlencoded link-update pattern that preserves risk2 content on CSRA and RISK items.
- Root cause: link updates clear risk2 unless `failOnCleanup=0` and `onlyThoseFields=1` are supplied.
