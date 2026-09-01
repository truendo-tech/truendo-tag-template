# TRUENDO GTM Tag Template — Change Ledger

A chronological log of work done on this repo. Newest entries on top.

| Date | Author | Change | Files Touched |
|---|---|---|---|
| 2026-09-01 | OpenCode | Plan refinement: opt-in regions table moved into a collapsible `ZIPPY_CLOSED` group ("Opt-in Regions") inside Consent Mode v2 Setup to keep UI clean; top-level zippy group as fallback if nesting unsupported. | `plan/PLAN.md`, `plan/LEDGER.md` |
| 2026-09-01 | OpenCode | Plan updated after review: opt-in regions field changed from TEXT to single-column PARAM_TABLE (chips UI confirmed not possible in GTM templates); decisions locked — EEA+UK+CH default list, custom table fully overrides fallback. Awaiting go-ahead to implement. | `plan/PLAN.md`, `plan/LEDGER.md` |
| 2026-09-01 | OpenCode | Planned region-aware consent defaults feature: new `opt_in_regions` TEXT param (default EEA+UK+CH), two-call `setDefaultConsentState` fallback (granted globally / denied in opt-in regions). Plan written, implementation pending decisions. | `plan/PLAN.md`, `plan/LEDGER.md` |
| 2026-09-01 | OpenCode | Created `plan/` folder with `PLAN.md` (planning) and `LEDGER.md` (this file) after repo familiarization. No functional changes. | `plan/PLAN.md`, `plan/LEDGER.md` |
