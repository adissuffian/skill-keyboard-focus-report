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

- Base URL. If the current repo has `docs/keyboard-focus-report.config.json`, use its `url` value as the default and only ask if the file is missing or the user wants to override it. Example: http://localhost:8080/index.html?ld_LovableMaster=true&ld_LovableHome=true&ld_LovableMenu=true&ld_TwoTapCheckout=true
- Canonical report path in current repo. If the current repo has `docs/keyboard-focus-report.config.json`, use its `reportPath` value as the default and only ask if the file is missing or the user wants to override it. Example: docs/keyboard-focus-report.html
- Sections to run: Home, Sign in, Sign up, Delivery, Delivery > Manual search
- Start-state policy: strict clean start, or reuse active browser state
- Screenshot output folder in repo (example: docs/visual-reports/<flow>-<date>/)

Repo-default execution rule:

- If `docs/keyboard-focus-report.config.json` exists, treat it as the single source of truth for base URL and canonical report path.
- In that case, do not ask the user for base URL or report path unless they explicitly want an override.
- Prefer this command shape in Copilot CLI and other automation:
  - `/Users/adis.suffian/.copilot/keyboard-focus-report/run-focus-report.sh --config docs/keyboard-focus-report.config.json`

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
  - Tag: <ELEMENT_TAG>, Role: <role-or-none>, Aria: <aria-label-or-best-accessible-name>
- Never mix description styles within one report (for example avoid narrative text in one section and Tag/Role/Aria in another).
- Do not use Label: in descriptions; use Aria: consistently.
- Do not infer exact CSS outline values unless measured directly from computed style.
- If a visible ring/indicator is present, never label it as none.

Pill standard text:

- Outline <style+width>; Outline color: <color>; Outline offset: <value>

Preferred examples:

- Outline solid: 2px; Outline color: black; Outline offset: 2px
- Outline solid: 2px; Outline color: white; Outline offset: 2px
- Outline solid: 2px; Outline color: blue; Outline offset: unknown
- Outline none: 3px; Outline color: blue; Outline offset: unknown

## Required Report Sections

Use this section order unless user asks otherwise:

1. Home
2. Sign in
3. Sign up
4. Delivery
5. Delivery > Manual search

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
- Enter test keyword (example: 1a london dr) in the Delivery search field.
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
- Confirm every card description starts with Tag:, includes Role:, and includes Aria:.
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
