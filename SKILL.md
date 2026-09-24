---
name: keyboard-focus-visual-report
description: "Create or update a visual keyboard-focus report with consistent sections, screenshot evidence, and outline-only labels. Use when: keyboard navigation QA, focus-state screenshots, accessibility evidence report, sign-in/sign-up/delivery focus documentation."
argument-hint: "Prefer repo defaults from docs/keyboard-focus-report.config.json; specify only sections to run, rerun mode, and whether to override the default base URL or report path"
---

# Keyboard Focus Visual Report

Create a repeatable keyboard-focus evidence report for web flows using browser automation and screenshots.

## When to Use

- Validate visible keyboard focus states for core user journeys
- Generate visual evidence for accessibility QA
- Rerun focus sweeps with strict start-state rules
- Keep one canonical report updated over time

## Inputs To Confirm

- Base URL. If the current repo has `docs/keyboard-focus-report.config.json`, use its `url` value as the default and only ask if the user wants to override it. If the file is missing, default to: https://bestellen.dominos.nl/?ld_LovableMaster=true&ld_LovableHome=true&ld_LovableMenu=true&ld_TwoTapCheckout=true
- Canonical report path in current repo. If the current repo has `docs/keyboard-focus-report.config.json`, use its `reportPath` value as the default and only ask if the file is missing or the user wants to override it. Example: docs/keyboard-focus-report.html
- Sections to run: Cookie banner (OneTrust), Home, Sign in, Sign up, Delivery, Manual Search
- Start-state policy: strict clean start, or reuse active browser state
- Screenshot output folder in repo (example: docs/visual-reports/<flow>-<date>/)

Repo-default execution rule:

- If `docs/keyboard-focus-report.config.json` exists, treat it as the single source of truth for base URL and canonical report path.
- In that case, do not ask the user for base URL or report path unless they explicitly want an override.
- Prefer this command shape in Copilot CLI and other automation:
  - `/Users/adis.suffian/.copilot/keyboard-focus-report/run-focus-report.sh --config docs/keyboard-focus-report.config.json`

## Gold Standard Rules (Mandatory — Never Override)

These rules were established and locked in as mandatory policy. They apply to every section, every rerun, every future report without exception.

### Sweep Order (Golden Rule)
- **Always run a full keyboard sweep in default state first** — no input, no interaction applied yet.
- Only after completing the full default-state sweep, proceed with interaction (e.g. type an address, open a dropdown).
- For any interaction that reveals a list or dropdown:
  - Sweep keyboard focus across its items first, **capturing up to a maximum of 4 unique stops** if the list is long.
  - Include `Manual Search` or equivalent secondary actions if they appear as keyboard-focusable stops within that context.
  - After the sweep, continue Tab until focus **loops back to the originating element** before activating it (click/Enter).
- For form fields (e.g. House Number): sweep the entire form in default state first. Only when focus returns to the same field a **second time** (loopback), enter input and continue the remaining test.

### Human-Visible Compliance (Golden Rule)
- **If a keyboard focus ring is not obviously visible to a human, classify it as non-compliant (blue).**
- Computed CSS values (outline, outline-width, outline-color) are evidence but are **not sufficient alone**.
- If computed style reports a valid ring but the screenshot shows only a thin line, divider, or clipped/masked ring, this is a **script false positive** and must be marked non-compliant.
- Record the actual computed values in the pill text (do not zero them out).
- Add `"compliant": false` on the card in the config to force the blue bubble regardless of pill content.
- Add a `"remark"` field on the card explaining the false positive reasoning.

### Remarks Field (Golden Rule)
- When a card has `"compliant": false` **or** when computed CSS and the screenshot visually disagree (false positive or false negative), a `"remark"` field is **required**.
- **False positive**: computed CSS reports a valid ring but no human-visible ring appears in the screenshot.
- **False negative**: computed CSS reports no ring but a human-visible ring is clearly present in the screenshot.
- Remark must explain: what was visually observed, why it diverges from computed CSS, and why it matters for a person with visual impairment.
- Remarks are rendered below the pill in the report.
- Validation will **fail** if a card has `"compliant": false` and no remark, or if the remark is empty.

### No Redundancy (Golden Rule)
- **Remove all duplicate stops** before saving the capture manifest.
- Deduplication is by: same `desc` field AND identical screenshot content (SHA-1 hash).
- After deduplication, renumber all stops sequentially so counts are accurate.
- A stop that represents a second-visit to the same element for a different purpose (e.g. second address-field focus before typing) is **not a duplicate** — it is a new intentional stop.

### Summary Table Headers (Golden Rule)
- The summary table must always use exactly: `Section`, `Stops`, `Compliance`, `Non-compliance`.
- Never use `Comply` or `Non-comply`.

### Aria Truncation (Golden Rule)
- Aria text in `desc` must be truncated to the **first 4 words followed by `...`** when the full text is longer than 4 words.
- This applies to every card in every section without exception.

### Pill Data Accuracy (Golden Rule)
- Pill text must always reflect **real measured computed style values**, never zeroed-out placeholders.
- If the ring is not human-visible, use `"compliant": false` to force the blue bubble — do not fake the pill data.

### Compliance Bubble Colour (Golden Rule)
- **Green bubble** = `pill-solid` = human-visible ring that matches contract (black or white, 2px solid, 2px offset).
- **Blue bubble** = `pill` = non-compliant: ring not human-visible, or wrong colour, or `"compliant": false` override.

## Standard Rules (Always Apply)

Reference focus-color contract for this repo:

- Source of truth: `applications/olo.web/style.less` (PR #4845, commit `da9a419f1d95b6af79e6d9858cce652149c52110`, around lines 81-133).
- Expected keyboard focus-visible outline color is black on light/white surfaces and white on dark/black surfaces (`footer` and `[data-focus-surface='dark']`).
- If observed focus ring color is not black or white (for example blue), record it as a non-contract color in the report pill text using CSS-prefixed wording.

- Capture only meaningful visible focus targets.
- Attach screenshots only when keyboard focus is present on the target element.
- Include a screenshot only if the keyboard focus indicator is visibly rendered in the UI.
- Exclude elements that are technically focused but have no visible keyboard focus indicator.
- For every page reached in the journey, perform a keyboard sweep before moving on.
- During each sweep, capture every meaningful visible keyboard-focus stop in sequence.
- Every screenshot must be tied to a keyboard focus move (for example Tab or Shift+Tab), not to a state-only change.
- Do not add duplicate cards for the same focused element unless focus moved away and returned via keyboard.
- Home-specific duplicate guard: if traversal loops back to the same Delivery control already captured as Home Focus 01 without a new meaningful keyboard-focus stop, do not add a redundant Home Focus 05 card.
- Wait briefly after each Tab so outline/ring is rendered before screenshot.
- Ignore generic BODY/HTML/container targets unless they are required to explain a blocker.
- A composite wrapper may be included when it is the actual keyboard-focus stop and the visible focus indicator is rendered on that wrapper rather than on a nested semantic control.
- Keep section naming and order stable unless explicitly requested to change.
- Use outline-only pills in report labels.
- Never use Flow labels in pills.
- Use one description schema for every card in every section:
  - Aria: <aria-label-or-best-accessible-name>, Tag: <ELEMENT_TAG>, Role: <role-or-none>
- Never mix description styles within one report (for example avoid narrative text in one section and Tag/Role/Aria in another).
- Do not use Label: in descriptions; use Aria: consistently.
- Do not infer exact CSS outline values unless measured directly from computed style.
- If a visible ring/indicator is present, never label it as none.
- If `outline: none` is computed but a visible browser-default focus indicator is still rendered (for example on an `<input>`), include the card and use pill: `Outline none: <width>; Outline color: browser default; Outline offset: unknown`.

Pill standard text:

- Outline <style+width>; Outline color: <color>; Outline offset: <value>

Preferred examples:

- Outline solid: 2px; Outline color: rgb(0, 0, 0) black; Outline offset: 2px
- Outline solid: 2px; Outline color: rgb(255, 255, 255) white; Outline offset: 2px
- Outline solid: 2px; Outline color: rgb(0, 95, 204) blue; Outline offset: unknown
- Outline none: 3px; Outline color: browser default; Outline offset: unknown

## Required Report Sections

Use this section order unless user asks otherwise:

1. Cookie banner (OneTrust)
2. Home
3. Sign in
4. Sign up
5. Delivery
6. Manual Search

## Execution Procedure

### 1) Prepare Start State

- Open target URL.
- If strict clean start is required:
  - Clear localStorage and sessionStorage.
  - Clear Cache Storage entries.
  - Unregister service workers.
  - Best-effort clear IndexedDB.
  - Reload and re-open start URL.
- Confirm page is interactive before traversal.

### 2) Traverse Per Section

For each requested section:

- Navigate to the entry state for that section.
- Sweep the current page using keyboard navigation before transitioning to the next page in the journey.
- Use keyboard traversal (Tab and Shift+Tab if needed).
- After each navigation key:
  - Wait 500-800ms.
  - Verify focused element is visible and meaningful.
  - Capture screenshot with deterministic file name.
- Stop when section reaches agreed coverage depth.

### 2a) Delivery First-Page Rerun Scenario

Use this scenario when Delivery labels are disputed or stale:

- Start from Home route and open Delivery from the start-order dialog.
- Enter test keyword **Bijlmerplein** in the Delivery search field (use this same address for both Delivery and Manual Search sections).
- Capture first-page tab sequence only (before navigating deeper):
  - search field
  - top utility controls (for example English, Account)
  - back/search-adjacent controls if focus reaches them
- Use a 700ms settle delay before each screenshot.
- Save to a new dated folder and update only the Delivery section unless instructed otherwise.

Keyboard-focus-only rule for Delivery:

- If the search wrapper itself is the keyboard stop before the text input becomes the active typing target, include the wrapper screenshot only when the wrapper shows the visible keyboard focus indicator.
- Do not capture a separate screenshot immediately after typing if focus stayed on the same input element.
- If typing is required to reveal the next focus targets, type while the input is focused, then continue keyboard traversal and capture on subsequent focus moves only.
- For Recent address rows, include evidence only when the row itself is reached by keyboard Tab/Shift+Tab with a visible focus indicator.
- If Recent is visible but not keyboard-focusable as its own stop, do not add a Recent card.

Fallback interaction rule:

- If standard click fails due to overlay/interception, use a safe DOM click fallback and document it in notes.

### 3) Save Screenshots

- Save under section-specific folder beneath docs/visual-reports/.
- Use stable naming:
  - 01-<section>-<target>.png
  - 02-<section>-<target>.png

### 4) Update Canonical Report

- Insert or update cards in matching section.
- Each card should include:
  - Screenshot image path
  - Concise title
  - Description line using Tag/Role/Aria format
  - Pill using Outline label
- Preserve existing styles and accordion structure.
- Keep previous evidence unless user requested replacement-only mode.

Header metadata rule:

- Keep hero metadata aligned with the current report state:
  - Date should reflect latest rerun date.
  - Scope should list all included sections.
  - Captured count should match total cards currently shown in report.

Label accuracy rule:

- Pill text must reflect visual evidence in screenshot, not guessed CSS values.
- Do not prefix pills with Focus Indicator.
- Use strict structure only: Outline <style>: <width>; Outline color: <color>; Outline offset: <value>.
- Use unknown only when a value cannot be measured directly from computed style.

### 5) Validate Before Finish

- Confirm no pill contains Flow: text.
- Confirm every pill uses the 3-field structure: outline, outline color, and outline offset.
- Confirm every card description starts with Aria:, includes Tag:, and includes Role:.
- Confirm focus contract baseline exists in `applications/olo.web/style.less`: `outline: 2px solid #000000` and `outline-offset: 2px`.
- Confirm dark-surface contract exists in `applications/olo.web/style.less`: footer and `[data-focus-surface='dark']` `:focus-visible` rules with `outline-color: #ffffff`.
- Confirm ring gap between element and focus indicator is 2px (`outline-offset: 2px`) when using the global outline contract.
- Confirm pill meaning is consistent with visuals: green for explicit `outline color black` or `outline color white`, blue for other observed colors (for example blue).
- Confirm all added images exist in repo paths.
- Confirm section titles are consistent and ordered.
- Confirm rerun section references the new dated screenshot folder (no stale paths).
- Confirm no added pill contradicts visible focus indicators in images.
- Confirm hero Date, Scope, and Captured metadata reflect latest report content.
- Open report in the active Google Chrome window and spot-check rendering.
  - Preferred on macOS: `open -a "Google Chrome" "file:///absolute/path/to/report.html"`
  - If Chrome open fails, document the failure and use the available browser tool as fallback.

## Output Format

Provide:

- What sections were executed
- Screenshot folder paths used
- Report file updated
- Confirmation that report was opened in active Google Chrome
- Any blocked steps and exact failure point

## Rerun Modes

- Delta rerun: update only specified section(s)
- Full rerun: refresh all sections with new dated screenshot folders
- Strict-first-page rerun: capture only first page/step focus states from clean entry
- Delivery-only correction rerun: refresh Delivery cards and labels when focus-outline wording is incorrect

## Safety Notes

- Do not commit unrelated file changes.
- Do not delete previous screenshot folders unless explicitly asked.
- If target elements are not interactable, use a safe fallback click method and document it in notes.

## Starter Template

Use this file for consistent report structure and wording:

- ./references/focus-report-template.html
