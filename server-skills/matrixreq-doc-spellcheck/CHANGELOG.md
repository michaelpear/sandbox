# Changelog
## 2026-11-10
- Expanded grammar coverage: explicit rule for clear word-form and agreement errors (noun/adjective, number, subject-verb) applied to all modes, including HA Hazard/Cause.
- Added example table (confusion results → confusing results, a USB drives → a USB drive, etc.).
- Updated skill description to mention grammar.
- Replaced fixed-pattern-only approach with required dictionary-based spellcheck (pyspellchecker) plus domain allowlist. Pattern list is now supplemental. Explicitly requires catching arbitrary typos such as `dtect` → `detect`.
