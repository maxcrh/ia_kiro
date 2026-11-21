# Implementation Plan: Dark Mode Full Coverage

## Overview

Convert the feature design into a series of prompts for a code-generation LLM
that will implement each step with incremental progress. Make sure that each
prompt builds on the previous prompts, and ends with wiring things together.
There should be no hanging or orphaned code that isn't integrated into a
previous step. Focus ONLY on tasks that involve writing, modifying, or testing
code.

This is a **pure styling effort**. Every task below is an **additive
`className` edit**: for each light-only utility class in scope, add the mapped
`dark:` variant from the design's palette mapping table on the same element,
and **never remove or replace an existing light class** (preserve the light
appearance exactly — Req 2.6 / Property 5). No new components, props, state,
storage keys, effects, or toggles are introduced, and no theme-infrastructure
or config files are touched.

**Palette mapping** (the single source of truth) is the "Dark palette mapping"
table in the design's Components and Interfaces section. Every edit must follow
it so identical semantic roles look identical everywhere (Req 3.2 / Property 3).

**Environment verification constraints (must be honored throughout):**
- Primary build gate after every batch of edits:
  `npx tsc --noEmit -p tsconfig.app.json`.
- Run single test files with no file parallelism:
  `npx vitest --run --no-file-parallelism path/to/file.test.tsx`.
- Do **NOT** run the full vitest suite (`npm test`) — it wedges the terminal in
  this environment. Validate file-by-file instead.

## Tasks

- [x] 1. Style the two pages (ModuleDetailPage, SettingsPage)
  - [x] 1.1 Add dark variants to `src/pages/ModuleDetailPage.tsx`
    - Outer wrapper `bg-gray-50` → add `dark:bg-gray-900`.
    - `LoadingState` skeleton blocks `bg-gray-100` / `bg-gray-200` → add `dark:bg-gray-700`.
    - `NotFoundState`: heading `text-gray-700` → `dark:text-gray-200`; body `text-gray-500` → `dark:text-gray-400`; svg `text-gray-300` → `dark:text-gray-600`.
    - Collapse/Expand buttons `text-gray-600 hover:text-gray-900` → `dark:text-gray-300 dark:hover:text-gray-100` (keep `focus:outline-none` + `focus:ring-*`).
    - Empty-section message `text-gray-400` → `dark:text-gray-500`.
    - Inline delete-error box `border-red-200 bg-red-50 text-red-700` → `dark:border-red-800 dark:bg-red-900/30 dark:text-red-300`.
    - Inline `ConfirmDialog` message `<strong class="text-gray-900">` → `dark:text-gray-100`.
    - Follow the palette mapping; add only, never replace light classes.
    - _Requirements: 2.1, 2.6, 3.1, 3.5, 5.1, 5.2_

  - [x] 1.2 Add dark variants to `src/pages/SettingsPage.tsx`
    - Outer wrapper `bg-gray-50` → `dark:bg-gray-900`.
    - `h1 text-gray-900` → `dark:text-gray-100`.
    - Each of the three section cards `border-gray-200 bg-white` → `dark:border-gray-700 dark:bg-gray-800`.
    - Each card `h2 text-gray-900` → `dark:text-gray-100`.
    - _Requirements: 2.2, 2.6, 3.1, 3.2_

- [x] 2. Checkpoint - type check after pages
  - Run `npx tsc --noEmit -p tsconfig.app.json` and confirm it is clean.
  - Ensure all tests pass, ask the user if questions arise.

- [x] 3. Style the Detail components
  - [x] 3.1 Add dark variants to `src/components/ModuleHeader.tsx`
    - Back/Delete controls' gray text (ghost treatment); module title `text-gray-900`; chevron/edit icon `text-gray-400`; Tags label `text-gray-700`.
    - Edit input: normal branch (`text-gray-900` / `border-indigo-400` + bg) and error branch (`border-red-400 bg-red-50 text-red-800`) per input + invalid-input mapping.
    - "Press Enter…" hint `text-gray-400` → `dark:text-gray-500`; field error `text-red-600` → `dark:text-red-400`.
    - Preserve focus utilities; add `dark:focus:ring-offset-*` where an offset ring sits on a dark surface.
    - _Requirements: 2.3, 2.6, 3.1, 3.2, 3.3, 3.4, 5.1, 5.3, 5.4_

  - [x] 3.2 Add dark variants to `src/components/SectionAccordion.tsx`
    - Card `border-gray-200`; header strip `bg-gray-50` → `dark:bg-gray-800`; toggle `hover:bg-gray-100` → `dark:hover:bg-gray-800`; title `text-gray-900`; progress summary `text-gray-500`; chevron `text-gray-400`.
    - Rename input: normal branch + error branch (`border-red-400 bg-red-50 text-red-800`).
    - Save (primary, keep) / Cancel (secondary) buttons; action icon buttons (ghost + ghost-danger).
    - Inline delete-error strip `bg-red-50 border-red-200 text-red-700`; `ConfirmDialog` `<strong>` `text-gray-900`.
    - _Requirements: 2.3, 2.6, 3.1, 3.2, 3.3, 3.4, 3.6, 5.1, 5.3, 5.4_

  - [x] 3.3 Add dark variants to `src/components/AddSectionForm.tsx`
    - DB-error banner (error box mapping); input (normal + error branch); field error `text-red-600`.
    - Add button (primary, keep; add `dark:focus:ring-offset-gray-900`).
    - _Requirements: 2.3, 2.6, 3.1, 3.3, 3.4, 3.6, 5.1, 5.3, 5.4_

  - [x] 3.4 Add dark variants to `src/components/SectionContent.tsx`
    - "Loading…" / "No content items yet." `text-gray-400` → `dark:text-gray-500`.
    - "Search the web" disclosure link: indigo link treatment + `hover:bg-indigo-50` → `dark:hover:bg-indigo-900/40`.
    - _Requirements: 2.3, 2.6, 3.1, 3.2_

  - [x] 3.5 Add dark variants to `src/components/ContentItemRow.tsx`
    - Row dividers `border-gray-200`; row hover `hover:bg-gray-50` → `dark:hover:bg-gray-800`; title link `text-indigo-600 hover:text-indigo-800`; external-link icon `text-gray-400`; metadata `text-gray-500`.
    - Edit/delete icon buttons (ghost + ghost-danger incl. `hover:bg-indigo-50` / `hover:bg-red-50`).
    - All edit-mode inputs (title, URL, rating — normal + error branches); edit-mode DB-error box; field errors; Save/Cancel buttons; `ConfirmDialog` `<strong>`.
    - _Requirements: 2.3, 2.6, 3.1, 3.2, 3.3, 3.4, 3.6, 5.1, 5.3, 5.4_

  - [x] 3.6 Add dark variants to `src/components/AddContentItemForm.tsx`
    - DB-error box; Title/URL labels `text-gray-700`; both inputs (normal + error branches); field errors; Add button (primary).
    - _Requirements: 2.3, 2.6, 3.1, 3.3, 3.4, 3.6, 5.1, 5.3, 5.4_

  - [x] 3.7 Add dark variants to `src/components/ContentItemNote.tsx`
    - Notes toggle `text-gray-600 hover:text-indigo-700 hover:bg-gray-100` → `dark:text-gray-300 dark:hover:text-indigo-300 dark:hover:bg-gray-800`; error `text-red-600` → `dark:text-red-400`.
    - _Requirements: 2.3, 2.6, 3.1, 3.4, 5.1_

  - [x] 3.8 Add dark variants to `src/components/MarkdownEditor.tsx`
    - Wrapper `border-gray-200`; tablist border; `ACTIVE_TAB_STYLE` (`bg-white text-indigo-700 border-indigo-500` → `dark:bg-gray-800 dark:text-indigo-300`) and `INACTIVE_TAB_STYLE` (`bg-gray-50 text-gray-600 hover:bg-gray-100` → `dark:bg-gray-900 dark:text-gray-400 dark:hover:bg-gray-800`).
    - Textarea (`border-gray-300` + bg + text + placeholder → `dark:border-gray-600 dark:bg-gray-900 dark:text-gray-100 dark:placeholder-gray-500`); footer border; char counter `text-gray-500`; Save button (primary).
    - _Requirements: 2.3, 2.6, 3.1, 3.2, 3.3, 3.6, 5.1, 5.3, 5.4_

  - [x] 3.9 Add dark variants to `src/components/MarkdownPreview.tsx`
    - Add `dark:text-gray-200` to the prose container and `dark:` variants to any hard-coded gray text so rendered notes are readable.
    - _Requirements: 2.3, 2.6, 3.1, 4.1_

  - [x] 3.10 Add dark variants to `src/components/TagInput.tsx`
    - Input normal, error, and disabled/at-limit branches (`bg-gray-100 text-gray-400` → `dark:bg-gray-800 dark:text-gray-500 dark:border-gray-700`); error `text-red-600`; limit hint `text-gray-500`.
    - _Requirements: 2.3, 2.6, 3.1, 3.3, 3.4, 5.1, 5.3, 5.4_

  - [x] 3.11 Add dark variants to `src/components/TagBadge.tsx`
    - `bg-blue-100 text-blue-800` → `dark:bg-blue-900 dark:text-blue-200`; remove × `text-blue-600 hover:text-blue-900` → `dark:text-blue-300 dark:hover:text-blue-100`.
    - _Requirements: 2.3, 2.6, 3.1, 3.7, 4.3_

  - [x] 3.12 Add dark variants to `src/components/QualityBadge.tsx`
    - Empty star `text-gray-300` → `dark:text-gray-600`; filled star `text-yellow-400` unchanged.
    - _Requirements: 2.3, 2.6, 3.1, 3.7, 4.3_

  - [x] 3.13 Add dark variants to the remaining Detail-rendered components
    - For `src/components/ContentThumbnail.tsx`, `src/components/ProgressControl.tsx`, `src/components/ContentItemProgress.tsx`, `src/components/SourceBadge.tsx`, and `src/components/WebSearchPanel.tsx`: apply the palette mapping to any light-only surface/text/border/badge/button classes they carry.
    - Leave already-theme-neutral elements (pure colored dots, images, backdrops) unchanged.
    - _Requirements: 2.3, 2.6, 3.1, 3.2, 3.6, 3.7, 5.1_

- [x] 4. Checkpoint - type check after Detail components
  - Run `npx tsc --noEmit -p tsconfig.app.json` and confirm it is clean.
  - Ensure all tests pass, ask the user if questions arise.

- [x] 5. Style the Settings components
  - [x] 5.1 Add dark variants to `src/components/ApiKeysSettings.tsx`
    - Credential label `text-gray-900`; description `text-gray-500`; password input (`bg-white text-gray-900 border-gray-300` → input mapping).
    - Save (primary) / Clear (secondary) buttons; confirmation `text-green-700` (success mapping); `StatusIndicator` "Set" chip (`text-green-700 bg-green-50 border-green-200`) and "Not set" chip (`text-gray-500 bg-gray-50 border-gray-200`) and their dots (`bg-gray-400` → `dark:bg-gray-500`; `bg-green-500` unchanged).
    - _Requirements: 2.4, 2.6, 3.1, 3.2, 3.3, 3.6, 3.7, 4.3, 5.1, 5.3, 5.4_

  - [x] 5.2 Add dark variants to `src/components/ExportImportSettings.tsx`
    - Section headings `text-gray-900`; helper text `text-gray-500` / `text-gray-700`; Export/Import/Confirm/Cancel buttons (primary + secondary).
    - Pending-import panel `border-gray-200 bg-gray-50` → `dark:border-gray-700 dark:bg-gray-800`; import-section top border `border-gray-100` → `dark:border-gray-700`; success and error status boxes.
    - _Requirements: 2.4, 2.6, 3.1, 3.2, 3.4, 3.6, 5.1, 5.3, 5.4_

  - [x] 5.3 Add dark variants to `src/components/ModuleSelectList.tsx`
    - "no modules" `text-gray-500`; Select all/none secondary buttons; list `border-gray-200 divide-gray-200` → `dark:border-gray-700 dark:divide-gray-700`; row `hover:bg-gray-50` → `dark:hover:bg-gray-800`; row label `text-gray-800`; checkbox `border-gray-300` → `dark:border-gray-600 dark:bg-gray-900`.
    - _Requirements: 2.4, 2.6, 3.1, 3.2, 3.6, 5.1, 5.3, 5.4_

  - [x] 5.4 Add dark variants to `src/components/DriveControls.tsx` (and verify `src/components/DriveSettings.tsx`)
    - Secondary/ghost buttons; progress (indigo `text-indigo-700 bg-indigo-50 border-indigo-200`), success (green), error (red) status boxes per mapping.
    - Nested `ConfirmRestoreModal`: `bg-white` card → `dark:bg-gray-800`, `text-gray-900` heading → `dark:text-gray-100`, `text-gray-600` body → `dark:text-gray-300`, buttons — same shared-dialog treatment as `ConfirmDialog`.
    - Confirm `DriveSettings.tsx` is a styling-free wrapper; if it carries any light-only classes, apply the mapping, otherwise leave unchanged.
    - _Requirements: 2.4, 2.6, 3.1, 3.2, 3.4, 3.6, 5.1, 5.3, 5.4_

- [x] 6. Checkpoint - type check after Settings components
  - Run `npx tsc --noEmit -p tsconfig.app.json` and confirm it is clean.
  - Ensure all tests pass, ask the user if questions arise.

- [x] 7. Style the Shared components
  - [x] 7.1 Add dark variants to `src/components/ConfirmDialog.tsx`
    - Card `bg-white` → `dark:bg-gray-800`; title `text-gray-900`; body `text-gray-600`; Cancel (secondary) button; confirm button offset ring `dark:focus:ring-offset-gray-800`.
    - Keep destructive/indigo fills and the `bg-black/50` backdrop unchanged.
    - _Requirements: 2.5, 2.6, 3.1, 3.2, 3.6, 5.1, 5.3, 5.4_

  - [x] 7.2 Add dark variants to `src/components/CreateModuleModal.tsx`
    - Card `bg-white`; title `text-gray-900`; close button (ghost); labels `text-gray-700`; required `*` `text-red-500` (keep), optional hint `text-gray-400`.
    - Name input (normal + error branch); field error; char counter `text-gray-400`; Cancel/Create buttons.
    - _Requirements: 2.5, 2.6, 3.1, 3.2, 3.3, 3.4, 3.6, 5.1, 5.3, 5.4_

  - [x] 7.3 Add dark variants to `src/components/GeneratedModulePreview.tsx`
    - Panel `border-gray-200 bg-white`; headings `text-gray-900`; helper `text-gray-500` / `text-gray-700`; name input; name error.
    - Section headings and counts; "Select all" labels `text-gray-600`; checkboxes `border-gray-300`; result dividers `divide-gray-200`; result links `text-indigo-600`; metadata `text-gray-500`.
    - Save (primary) / Discard (secondary) buttons; empty-selection error `text-red-600`.
    - _Requirements: 2.5, 2.6, 3.1, 3.2, 3.3, 3.4, 3.6, 5.1, 5.3, 5.4_

  - [x] 7.4 Add dark variants to `src/components/PwaInstallBanner.tsx`
    - Banner `bg-white border-indigo-200` → `dark:bg-gray-800 dark:border-indigo-900`; message `text-gray-800` → `dark:text-gray-100`; Install (primary) / Dismiss (secondary) buttons; keep the indigo icon.
    - _Requirements: 2.5, 2.6, 3.1, 3.2, 3.6, 5.1_

- [x] 8. Checkpoint - type check after Shared components
  - Run `npx tsc --noEmit -p tsconfig.app.json` and confirm it is clean.
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 9. Structural property and unit tests
  - [ ]* 9.1 Write structural test for Property 1 (no runtime theme logic)
    - **Property 1: No runtime theme logic**
    - **Validates: Requirements 1.1, 1.2, 1.3, 1.4, 6.4**
    - Loop over every in-scope file; assert the source contains no `useTheme`, no `'theme'` localStorage read / `THEME_STORAGE_KEY`, no `matchMedia`, no root `classList` theme inspection, and no `ThemeToggle` import or `toggleTheme`/`setTheme` call. Fail with the offending file name.
    - Run via `npx vitest --run --no-file-parallelism <file>`.

  - [ ]* 9.2 Write structural test for Property 2 (dark-variant coverage)
    - **Property 2: Dark-variant coverage of every UI element category**
    - **Validates: Requirements 2.1, 2.2, 2.3, 2.4, 2.5, 3.1, 3.3, 3.6, 3.7**
    - For each in-scope file, for each light class in the mapping table that the file uses, assert the paired `dark:` variant also appears. Add render-based spot checks: render `ModuleDetailPage` (loading, not-found, loaded + delete-error states) and `SettingsPage` and assert enumerated elements expose their mapped `dark:` classes.
    - Run via `npx vitest --run --no-file-parallelism <file>`.

  - [ ]* 9.3 Write structural test for Property 3 (consistent styling for identical roles)
    - **Property 3: Consistent dark styling for identical semantic roles**
    - **Validates: Requirements 3.2**
    - Group mapped occurrences by semantic role (secondary button, inline error box, tag chip, card surface, form input) and assert every occurrence uses the identical `dark:` class set.

  - [ ]* 9.4 Write structural test for Property 4 (error surfaces distinct)
    - **Property 4: Error surfaces stay visually distinct in dark mode**
    - **Validates: Requirements 3.4**
    - Enumerate error/alert boxes; assert each uses the red-family dark classes and that the family differs from the gray/indigo dark families on the same surface.

  - [ ]* 9.5 Write structural test for Property 5 (light theme preserved / additive-only)
    - **Property 5: Light theme is preserved (additive-only change)**
    - **Validates: Requirements 2.6, 5.2**
    - Render each surface with the root `dark` class absent and assert every original light class is still present on each element.

  - [ ]* 9.6 Write structural test for Property 6 (focus indicators preserved)
    - **Property 6: Focus indicators are preserved and visible in both themes**
    - **Validates: Requirements 5.1, 5.3, 5.4**
    - Enumerate interactive controls; assert each still carries `focus:outline-none` + a `focus:ring-*` class, and that controls with a ring offset on a dark surface include `dark:focus:ring-offset-gray-900`/`-gray-800`.

  - [ ]* 9.7 Write example/edge-case unit tests
    - Skeleton pair (3.5): assert `LoadingState` blocks carry `dark:bg-gray-700` within a `dark:bg-gray-900` container.
    - Invalid-input state: assert the error branch of each form input carries the mapped dark error classes in addition to the normal-branch dark classes.
    - Status chips (ApiKeys "Set"/"Not set"): assert both variants carry their mapped dark chip classes.
    - _Requirements: 3.3, 3.4, 3.5_

- [x] 10. Final checkpoint - verify and clean up
  - Run the primary gate `npx tsc --noEmit -p tsconfig.app.json` and confirm it is clean.
  - Run each added test file individually with `npx vitest --run --no-file-parallelism <file>`; do NOT run the full suite.
  - Remove any scratch/temporary artifacts created while validating; keep only the intended component edits and added `*.test.tsx` files.
  - Ensure all tests pass, ask the user if questions arise.

## Notes

- Tasks marked with `*` are optional and can be skipped for faster MVP.
- Every task is additive: add mapped `dark:` variants, never remove or replace
  existing light classes (Req 2.6 / Property 5).
- Each task references the specific requirement sub-items it satisfies.
- Contrast ratios (Req 4.1–4.3 and the ratio portions of 2.x/3.x/5.3) and the
  no-flash criteria (Req 6.1–6.3) are verified by the design-time palette audit
  and a manual browser pass, not by runtime tests — they are not jsdom-testable
  and have no coding task here.
- Checkpoints run the primary type-check gate; test files are validated
  individually with `--no-file-parallelism`. The full vitest suite is never run.

## Task Dependency Graph

```json
{
  "waves": [
    { "id": 0, "tasks": ["1.1", "1.2", "3.1", "3.2", "3.3", "3.4", "3.5", "3.6", "3.7", "3.8", "3.9", "3.10", "3.11", "3.12", "3.13", "5.1", "5.2", "5.3", "5.4", "7.1", "7.2", "7.3", "7.4"] },
    { "id": 1, "tasks": ["9.1", "9.2", "9.3", "9.4", "9.5", "9.6", "9.7"] }
  ]
}
```
