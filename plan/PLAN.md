# TRUENDO GTM Tag Template — Planning

## Current Initiative: Region-Aware Consent Defaults (Opt-in / Opt-out)

### Background & Goal
New Google requirements mean consent defaults should no longer be `denied`
globally. Instead:
- **Opt-in regions** (GDPR-style jurisdictions) → default `denied`
- **Opt-out regions** (everywhere else) → default `granted`

This only applies to the built-in fallback path — when the user has NOT
configured any rows in the `defaultSettings` table. Users who configure
custom default rows keep full control and are unaffected.

### Technical Constraint
GTM template parameters are **declarative only** — the editor UI is rendered
entirely by Google from `___TEMPLATE_PARAMETERS___`. Custom widgets, onclick
handlers, or injected JS/CSS in the configuration UI are **not possible**
(the sandboxed JS only runs on the client's site at runtime). A "chips with
close icons" UI is therefore not feasible; the closest native equivalent is a
**single-column PARAM_TABLE** (visible list, add-row, per-row delete), which
was the chosen approach.

### Proposed Changes

#### 1. New template parameter
Add a collapsible `GROUP` ("Opt-in Regions", `groupStyle: ZIPPY_CLOSED`)
**inside** the existing "TRUENDO Consent Mode Settings" group, containing
the country table. The zippy keeps the UI clean — the 32 pre-populated rows
stay collapsed until the user expands the section (same accordion mechanism
the template already uses for its top-level groups).

> Fallback: if nested groups don't render correctly in the editor, make
> "Opt-in Regions" a third top-level `ZIPPY_CLOSED` group instead
> (guaranteed to work). Verify during implementation.

The group contains a single parameter:

| Property | Value |
|---|---|
| type | `PARAM_TABLE` |
| name | `opt_in_regions` |
| displayName | `Opt-in Regions` |
| columns | single column: `country` (TEXT, displayName "Country Code") |
| column validator | `REGEX`: `^[A-Za-z]{2}$` (2-letter ISO code), `valueHint`: `DE` |
| column `isUnique` | `true` (prevents duplicate country rows) |
| defaultValue | pre-populated with 32 rows: AT, BE, BG, HR, CY, CZ, DK, EE, FI, FR, DE, GR, HU, IS, IE, IT, LV, LI, LT, LU, MT, NL, NO, PL, PT, RO, SK, SI, ES, SE, GB, CH |
| help | Explains: countries listed here default to `denied`, all others to `granted`. Only applies when no rows exist in "Default settings" above. If all rows are deleted, the built-in opt-in list (EEA + UK + CH) is used. |

Why this UI: each country is a visible row with a native delete action and
an "Add Row" button — functionally the closest GTM-native equivalent to the
requested chips/cards UI. The table value is the sole source of truth.

Default opt-in list = 27 EU members + IS, LI, NO (EEA) + GB + CH (32 codes).
**Confirmed** with stakeholder. (Adjust if legal/compliance later adds other
jurisdictions, e.g. BR.)

Behavior note: since the table is pre-populated, an **empty** table is
treated as "use built-in default list" (documented in help text) — this
prevents accidentally granting consent everywhere.

#### 2. Code change (sandboxed JS)
Add a constant and modify the `else` branch of the defaults logic:

```js
const DEFAULT_OPT_IN_REGIONS = ['AT', 'BE', 'BG', 'HR', 'CY', 'CZ', 'DK',
  'EE', 'FI', 'FR', 'DE', 'GR', 'HU', 'IS', 'IE', 'IT', 'LV', 'LI', 'LT',
  'LU', 'MT', 'NL', 'NO', 'PL', 'PT', 'RO', 'SK', 'SI', 'ES', 'SE', 'GB',
  'CH'];
```

```js
} else {
  // Build opt-in region list from the param table; fall back to the
  // built-in list if the table is empty
  let optInRegions = [];
  if (data.opt_in_regions && data.opt_in_regions.length > 0) {
    optInRegions = data.opt_in_regions
      .map(row => (row.country || '').trim().toUpperCase())
      .filter(code => code.length === 2);
  }
  if (optInRegions.length === 0) {
    optInRegions = DEFAULT_OPT_IN_REGIONS;
  }

  // Opt-out default: granted everywhere (regionless)
  setDefaultConsentState({
    'ad_storage': 'granted',
    'analytics_storage': 'granted',
    'functionality_storage': 'granted',
    'personalization_storage': 'granted',
    'security_storage': 'granted',
    'ad_user_data': 'granted',
    'ad_personalization': 'granted',
    'wait_for_update': 500,
  });

  // Opt-in regions: denied — region-scoped defaults override the
  // regionless default for visitors from these regions
  setDefaultConsentState({
    'ad_storage': 'denied',
    'analytics_storage': 'denied',
    'functionality_storage': 'denied',
    'personalization_storage': 'denied',
    'security_storage': 'granted',
    'ad_user_data': 'denied',
    'ad_personalization': 'denied',
    'region': optInRegions,
    'wait_for_update': 500,
  });
}
```

Notes:
- `security_storage` stays `granted` in both calls (matches current behavior).
- The `defaults.length > 0` branch (custom table) is **unchanged** —
  **confirmed**: custom "Default settings" rows fully override the new
  opt-in/opt-out fallback.
- Codes are trimmed and uppercased so `de` and ` DE ` both work.

#### 3. Permissions
No new permissions required — `access_consent` write is already granted for
all seven consent types.

#### 4. Tests (`___TESTS___`, currently empty)
Add scenarios:
- Default (pre-populated) table → two `setDefaultConsentState` calls; second
  has the 32-code region list.
- Custom rows (e.g. DE, AT) → denied scoped to exactly those.
- Lowercase / whitespace input (`de`, ` at `) → normalized to `DE`, `AT`.
- Empty table → falls back to built-in 32-code list.
- Rows in `defaultSettings` → old single-call-per-row behavior, no
  opt-in/opt-out fallback calls.

#### 5. Documentation
Update `readme.md`:
- Document the new **Opt-in Regions** table and the opt-in/opt-out default
  behavior.
- Note it is ignored when custom "Default settings" rows exist, and that
  deleting all rows restores the built-in EEA + UK + CH list.

#### 6. Release
1. Implement + test in a GTM test container (Preview/debug mode).
2. Commit with a clear message.
3. Add new entry at top of `metadata.yaml` (new commit SHA + change notes).

### Open Decision Points
- [x] Default opt-in country list → **EEA + UK + CH (32 codes)**.
- [x] Interaction with custom table → **custom rows fully override** the
      new fallback.
- [x] Field UI → **single-column PARAM_TABLE** (chips UI not possible in
      GTM templates).
- [ ] Keep `wait_for_update: 500` in both calls (recommended, matches
      current behavior).
- [ ] Verify nested `ZIPPY_CLOSED` group renders correctly in the GTM
      template editor; fall back to top-level group if not.
- [ ] Verify PARAM_TABLE `defaultValue` pre-population renders correctly
      in the GTM template editor (verify during implementation).

### Manual Test Plan (GTM Preview)
- Fresh visitor, blank field → global `granted`, denied only for opt-in list.
- Fresh visitor, `DE,AT` in field → denied only for DE/AT.
- Fresh visitor with custom table rows → behavior identical to current release.
- Returning visitor with `truendo_cc` cookie → `updateConsentState` still
  fires correctly after defaults.

---

## Project Overview
This repo contains the Google Tag Manager Community Gallery template for the
TRUENDO Consent Management Platform (CMP). The template supports Google
Consent Mode v2 and can both inject the TRUENDO banner script and map
TRUENDO consent categories to Google consent types.

## Repo Structure
| File | Purpose |
|---|---|
| `template.tpl` | The GTM tag template (parameters, sandboxed JS, permissions) |
| `metadata.yaml` | Gallery version history (commit SHA + change notes) |
| `readme.md` | User-facing setup instructions (Basic & Advanced Consent Mode) |
| `LICENSE` | License terms |

## Current Template Behavior
- **Parameters (Inject TRUENDO group):** `truendo_inject`, `site_id`,
  `transparency`, `accessibility`, `nofont`, `lang_conf`/`lang_id`,
  `enable_auto_block`, `enable_event_triggers`.
- **Parameters (Consent Mode v2 group):** `defaultSettings` table (per-region
  defaults for all 7 consent types), `ads_data_redaction`, `url_passthrough`.
- **Script flow (`main`):**
  1. `gtagSet` for ads_data_redaction, url_passthrough, developer_id.
  2. Set default consent state (from table or all-denied fallback),
     `wait_for_update: 500`.
  3. Read consent from `truendo_cc` cookie (preferred) or `truendo_cmp` cookie;
     if found, call `onUserConsent`.
  4. Register a listener on `window.truConsentListeners` for live consent
     changes.
  5. Inject `https://cdn.priv.center/pc/truendo_cmp.pid.js` (if
     `truendo_inject` enabled).
- **Consent mapping:** marketing → ad_storage/ad_user_data/ad_personalization;
  statistics → analytics_storage; add_features → functionality_storage/
  personalization_storage; security_storage always granted.
- **Event triggers (optional):** pushes `truendo_cc_marketing`,
  `truendo_cc_statistics`, `truendo_cc_preferences`,
  `truendo_cc_social_sharing`, `truendo_cc_social_content`,
  `truendo_cc_add_features`, `truendo_initialized` to the dataLayer.

## Release Process
1. Make changes to `template.tpl`.
2. Test in GTM (import template into a test container; use Preview/debug mode).
3. Commit with a clear message.
4. Add a new entry at the top of `metadata.yaml` with the new commit SHA and
   change notes (this publishes the update to the Gallery).

## Known Issues / Observations
- `wait_for_update` is set twice in the custom-defaults branch (harmless
  duplication).
- `TruendoCookieControl` and `TruendoAddConsentListener` globals are declared
  in permissions but not used by the current script.
- `decode` (decodeUriComponent) is required but never used.
- Cookie value from `truendo_cmp` may be URL-encoded — worth verifying parse
  robustness.
- No test scenarios defined in `___TESTS___` (`scenarios: []`).
- `truendo_cc_add_features` event is pushed but not documented in the readme's
  trigger list.
- Logging (`logToConsole`) is active for all runs — confirm it is restricted
  to debug environments only (permission is set to `debug`, good).

## Backlog / Ideas
- [ ] Add test scenarios in `___TESTS___`.
- [ ] Clean up unused requires/permissions if confirmed unused.
- [ ] Verify URL-decoding of `truendo_cmp` cookie values.
- [ ] Sync readme trigger list with `truendo_cc_add_features` event.
- [ ] Consider configurable `wait_for_update` value.
