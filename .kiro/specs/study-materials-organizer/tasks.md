# Implementation Plan: Study Materials Organizer

## Overview

Build a 100% client-side PWA with React 18, TypeScript, Tailwind CSS, Dexie.js (IndexedDB), and Vite. All tasks are incremental: each step wires up only what is needed and hands a working slice of the app to the next task. The implementation order is: project scaffold → database → routing → dashboard → tags → filtering → module detail → content items → progress → notes → search → export/import → Google Drive → PWA → property tests → responsive polish.

## Tasks

- [x] 1. Scaffold the project with Vite, React 18, TypeScript, Tailwind CSS, and core dependencies
  - [x] 1.1 Initialise a new Vite project with the `react-ts` template and install all production dependencies: `react`, `react-dom`, `react-router-dom`, `dexie`, `react-markdown`, `remark-gfm`, `tailwindcss`, `postcss`, `autoprefixer`, `vite-plugin-pwa`
    - Run `npm create vite@latest study-materials-organizer -- --template react-ts`
    - Install all listed production dependencies with pinned versions
    - Install dev dependencies: `vitest`, `@testing-library/react`, `@testing-library/jest-dom`, `fast-check`, `fake-indexeddb`, `msw`, `@vitejs/plugin-react`
    - Configure `tailwind.config.ts` with `content` glob covering `src/**/*.{ts,tsx}`
    - Add `@tailwind base/components/utilities` directives to `src/index.css`
    - Verify `vite.config.ts` includes `@vitejs/plugin-react`
    - _Requirements: 1.1, 1.2, 2.1_

  - [x] 1.2 Configure Vitest, testing-library, and fast-check
    - Add `test` section to `vite.config.ts` (environment: `jsdom`, `globals: true`, `setupFiles`)
    - Create `src/test/setup.ts` importing `@testing-library/jest-dom/vitest`
    - Add `fake-indexeddb/auto` import in setup to replace IndexedDB with in-memory version for tests
    - Verify a trivial `it('works', () => expect(true).toBe(true))` passes with `vitest --run`
    - _Requirements: 10.1_

- [x] 2. Define the Dexie database schema and TypeScript entity interfaces
  - [x] 2.1 Create TypeScript entity interfaces in `src/db/types.ts`
    - Define and export `Module`, `Tag`, `ModuleTagAssociation`, `Section`, `ContentItem`, `QualityMetadata`, `Note`, `ProgressStateRecord`, and `ProgressState` type exactly as specified in the design document
    - _Requirements: 10.1_

  - [x] 2.2 Implement `StudyOrganizerDB` class in `src/db/db.ts`
    - Extend `Dexie` with all 7 tables typed against the interfaces from 2.1
    - Define `version(1).stores(...)` with the exact index strings from the design: `++id, name, createdAt, updatedAt` for modules; `++id, &name` for tags; `++id, moduleId, tagId, [moduleId+tagId]` for moduleTagAssociations; `++id, moduleId, order` for sections; `++id, sectionId, order` for contentItems; `++id, &contentItemId` for notes and progressStates
    - Export a singleton `db` instance
    - _Requirements: 10.1_

  - [x] 2.3 Write property test for valid module name persistence (Property 1)
    - **Property 1: Valid Module Names Are Persisted**
    - **Validates: Requirements 3.6**
    - Use `fc.string({ minLength: 1, maxLength: 100 })` filtered to non-whitespace-only strings; assert that `db.modules.add(...)` resolves and the retrieved entity has the same name

  - [x] 2.4 Write property test for whitespace-only module name rejection (Property 2)
    - **Property 2: Whitespace-Only Module Names Are Rejected**
    - **Validates: Requirements 3.8**
    - Use `fc.stringOf(fc.constantFrom(' ', '\t', '\n'))` to generate whitespace-only strings; assert that the validation function rejects them and `db.modules.count()` is unchanged

- [x] 3. Set up routing with HashRouter
  - [x] 3.1 Create `src/App.tsx` with `HashRouter`, `Routes`, and two `Route` entries: `path="/"` → `DashboardPage` (placeholder) and `path="/modules/:moduleId"` → `ModuleDetailPage` (placeholder)
    - Create stub components `src/pages/DashboardPage.tsx` and `src/pages/ModuleDetailPage.tsx` that render a heading with the route name
    - Mount `<App />` in `src/main.tsx`
    - _Requirements: 6.1_

- [x] 4. Build the Dashboard page — Module grid, ModuleCard, empty state, and CreateModuleModal
  - [x] 4.1 Implement `ModuleCard` component in `src/components/ModuleCard.tsx`
    - Display module name truncated at 60 characters with CSS ellipsis
    - Display total ContentItem count and completed ContentItem count (pass as props for now)
    - Display up to 5 `TagBadge` components; if more than 5 tags exist show a `+N` overflow indicator
    - Display a visual "All Completed" badge when completedCount === totalCount and totalCount > 0
    - Navigate to `/modules/${module.id}` on click using `useNavigate`
    - _Requirements: 3.3, 3.4, 9.3, 9.5_

  - [x] 4.2 Implement `DashboardPage` with live data from Dexie
    - Use `useLiveQuery` to subscribe to all modules, tags, moduleTagAssociations, sections, contentItems, and progressStates
    - Render `ModuleGrid` (a CSS grid div) containing one `ModuleCard` per module with computed `totalCount` and `completedCount`
    - Render `EmptyState` component (message + "Create Module" CTA button) when modules array is empty
    - _Requirements: 3.1, 3.2, 3.7_

  - [x] 4.3 Implement `CreateModuleModal` and wire it to `DashboardPage`
    - Controlled form with a module name input (maxLength=100) and a `TagInput` placeholder (real TagInput wired in task 5)
    - On submit: validate name is 1–100 non-whitespace chars; if invalid show inline error and do not write to DB
    - On valid submit: `db.modules.add(...)` inside a `db.transaction('rw', [db.modules], ...)` block; close modal on success
    - On dismiss (Escape or cancel button): close modal without any DB write
    - _Requirements: 3.5, 3.6, 3.8, 3.9_

- [x] 5. Build the Tag system — TagBadge, TagInput, tag CRUD service layer
  - [x] 5.1 Implement `TagBadge` component in `src/components/TagBadge.tsx`
    - Render tag name in a pill with optional remove (×) button controlled by `onRemove` prop
    - _Requirements: 4.4_

  - [x] 5.2 Implement `TagInput` component in `src/components/TagInput.tsx`
    - Accept `selectedTags`, `onAdd`, `onRemove`, `maxTags` (default 10), and `suggestions` props matching the design's `TagInputProps` interface
    - Show autocomplete dropdown with up to 20 suggestions filtered from `suggestions` prop, starting from first character typed
    - Allow creating a new tag by entering a name not in suggestions and pressing Enter/comma; reject names > 30 chars with an inline error
    - Block adding when `selectedTags.length >= maxTags`; show an error message indicating the 10-tag limit
    - Render selected tags as `TagBadge` components with `onRemove` wired
    - _Requirements: 4.1, 4.2, 4.3_

  - [x] 5.3 Implement tag service functions in `src/db/tagService.ts`
    - `getOrCreateTag(name: string): Promise<Tag>` — looks up tag by name, creates if absent
    - `assignTagToModule(moduleId: number, tagId: number): Promise<void>` — inserts `ModuleTagAssociation` inside a transaction; no-op if association exists
    - `removeTagFromModule(moduleId: number, tagId: number): Promise<void>` — deletes the association; retains the Tag record
    - `getTagsForModule(moduleId: number): Promise<Tag[]>` — joins `moduleTagAssociations` and `tags`
    - `getAllTagsWithSuggestions(query: string): Promise<Tag[]>` — returns up to 20 tags whose name starts with `query` (case-insensitive)
    - _Requirements: 4.4, 4.5, 4.6_

  - [x] 5.4 Write property test for tag assignment round-trip (Property 3)
    - **Property 3: Tag Assignment Round-Trip**
    - **Validates: Requirements 4.4**
    - Generate arbitrary module and tag IDs; call `assignTagToModule` then `getTagsForModule`; assert result includes the assigned tag

  - [x] 5.5 Write property test for tag removal removes only the target tag (Property 4)
    - **Property 4: Tag Removal Removes Only the Target Tag**
    - **Validates: Requirements 4.5**
    - Seed a module with N ≥ 2 tags; call `removeTagFromModule` for one; assert removed tag absent and all others present

  - [x] 5.6 Write property test for tag count upper bound enforcement (Property 5)
    - **Property 5: Tag Count Upper Bound Is Enforced**
    - **Validates: Requirements 4.1**
    - Seed a module with exactly 10 tags; attempt to assign an 11th; assert rejection and count remains 10

  - [x] 5.7 Write property test for tag name length boundary (Property 6)
    - **Property 6: Tag Name Length Boundary**
    - **Validates: Requirements 4.3**
    - Use `fc.string({ minLength: 1, maxLength: 30 })` → assert creation succeeds; `fc.string({ minLength: 31 })` → assert rejection and no new DB record

  - [x] 5.8 Wire `TagInput` into `CreateModuleModal` and `ModuleHeader`
    - Pass `suggestions` from `getAllTagsWithSuggestions` (debounced query); handle `onAdd` calling `getOrCreateTag` + `assignTagToModule`; handle `onRemove` calling `removeTagFromModule`
    - _Requirements: 4.1, 4.2, 3.5_

- [x] 6. Implement Dashboard filtering — SearchBar, TagFilter, filterModules, no-results state
  - [x] 6.1 Implement `filterModules` pure function in `src/utils/filterModules.ts`
    - Signature: `filterModules(modules, tagAssociations, query, selectedTagIds): Module[]`
    - Name filter: `query.trim().toLowerCase()` contained in `module.name.toLowerCase()` (empty query matches all)
    - Tag filter: AND logic — module must have ALL `selectedTagIds` present in `tagAssociations`
    - _Requirements: 5.1, 5.3, 5.4_

  - [x] 6.2 Write property test for dashboard name filter — all results match query (Property 7)
    - **Property 7: Dashboard Name Filter — All Results Match the Query**
    - **Validates: Requirements 5.1**
    - Generate arbitrary `Module[]` and non-empty query string; assert every returned module name contains query (case-insensitive) and every non-matching module is absent

  - [x] 6.3 Write property test for dashboard tag filter AND semantics (Property 8)
    - **Property 8: Dashboard Tag Filter — AND Semantics**
    - **Validates: Requirements 5.3**
    - Generate arbitrary modules, associations, and non-empty selectedTagIds; assert every returned module has all selected tags and every module missing any selected tag is absent

  - [x] 6.4 Write property test for combined name + tag filter (Property 9)
    - **Property 9: Combined Name + Tag Filter**
    - **Validates: Requirements 5.4**
    - Generate modules, associations, query, and selectedTagIds; assert every returned module satisfies both conditions simultaneously

  - [x] 6.5 Implement `SearchBar` component in `src/components/SearchBar.tsx`
    - Controlled input with `value`, `onChange`, `placeholder`, and `maxLength` (200) props
    - Debounce `onChange` calls by 100ms using a `useEffect`/`setTimeout` pattern
    - _Requirements: 5.1, 5.6_

  - [x] 6.6 Implement `TagFilter` component in `src/components/TagFilter.tsx`
    - Render a list of `TagFilterChip` sub-components for each available tag (label max 50 chars, truncated with ellipsis)
    - Visually distinguish selected vs. unselected chips (distinct background/border)
    - Emit `onToggle(tagId)` on chip click
    - _Requirements: 5.2, 5.3, 5.8_

  - [x] 6.7 Wire `SearchBar`, `TagFilter`, and `filterModules` into `DashboardPage`
    - Manage `query` (string) and `selectedTagIds` (number[]) with `useReducer` inside `DashboardPage`
    - Pass filtered modules array to `ModuleGrid`
    - Show no-results message (with current filter state preserved) when filtered array is empty but modules array is not
    - Clear filters → show all modules
    - _Requirements: 5.4, 5.5, 5.7_

- [x] 7. Build the Module Detail page — header, section management, CollapseAll/ExpandAll
  - [x] 7.1 Implement `ModuleHeader` in `src/components/ModuleHeader.tsx`
    - Display module name with an edit-in-place control (click to show input, blur/Enter to save)
    - Include `TagInput` wired to the module's current tags and tag service functions from task 5
    - Show a Back button that calls `navigate(-1)`
    - _Requirements: 6.1_

  - [x] 7.2 Implement `SectionAccordion` in `src/components/SectionAccordion.tsx`
    - Accept `section`, `isExpanded`, `onToggle`, and `children` props
    - Render `SectionHeader` (section name + toggle chevron) and collapsible body
    - Display `SectionProgress` subtitle: "X / Y completed"
    - _Requirements: 6.2, 6.4, 9.4_

  - [x] 7.3 Implement `AddSectionForm` in `src/components/AddSectionForm.tsx`
    - Text input (maxLength=100) + submit button; validate 1–100 non-whitespace chars
    - On valid submit: `db.sections.add(...)` inside a transaction; clear input on success
    - Show inline validation error for empty/whitespace/too-long names; show toast error if DB write fails and preserve input value
    - _Requirements: 6.5, 6.6, 6.7_

  - [x] 7.4 Implement `ModuleDetailPage` in `src/pages/ModuleDetailPage.tsx`
    - Use `useLiveQuery` to load module, its sections, and all related data
    - Manage `expandedSectionIds` set with `useReducer`
    - Render `ModuleHeader`, `CollapseAllButton` / `ExpandAllButton`, list of `SectionAccordion` components, and `AddSectionForm`
    - "Collapse All" sets expandedSectionIds to empty; "Expand All" sets it to all section IDs
    - _Requirements: 6.1, 6.2, 6.3_

  - [x] 7.5 Write property test for section expand/collapse toggle idempotence (Property 10)
    - **Property 10: Section Expand/Collapse Toggle Is Idempotent Over Two Clicks**
    - **Validates: Requirements 6.4**
    - Use `fc.boolean()` for initial state; apply toggle twice; assert result equals initial state

- [x] 8. Implement ContentItem management — add form, ContentItemRow, delete modal
  - [x] 8.1 Implement `AddContentItemForm` in `src/components/AddContentItemForm.tsx`
    - Two fields: Title (maxLength=200, required) and URL (maxLength=2048, must start with `http://` or `https://`)
    - Field-level validation errors displayed below each failing field
    - On valid submit: `db.contentItems.add({ ..., order: maxExistingOrder + 1 })` inside a transaction that also inserts a default `ProgressStateRecord` with state `'Not Started'`
    - _Requirements: 7.1, 7.2, 9.6_

  - [x] 8.2 Write property test for ContentItem append-only order invariant (Property 11)
    - **Property 11: ContentItem Append-Only Order Invariant**
    - **Validates: Requirements 7.3**
    - Use `fc.array(fc.string(), { minLength: 1, maxLength: 20 })` as titles; add items sequentially; assert each new item's `order` is strictly greater than all prior items' `order`

  - [x] 8.3 Implement `QualityBadge` component in `src/components/QualityBadge.tsx`
    - Render badges for `viewCount`, `rating`, `channelName`/`domainName` only when those fields are present in `QualityMetadata`
    - Render nothing at all when `metadata` is `undefined` or has no present fields
    - _Requirements: 11.3, 11.4_

  - [x] 8.4 Implement `ContentItemRow` in `src/components/ContentItemRow.tsx`
    - Display title, URL as an `<a target="_blank" rel="noopener noreferrer">` link, `ProgressControl` (stub for now), `QualityBadge`, `MarkdownEditor` (stub for now), and a delete icon button
    - Delete icon opens a confirmation modal naming the ContentItem title; confirm → `db.transaction('rw', [db.contentItems, db.notes, db.progressStates], ...)` deletes item + associated note + progressState; cancel → modal dismissed, item unchanged
    - _Requirements: 7.4, 7.5, 7.6, 7.7_

  - [x] 8.5 Wire `AddContentItemForm` and `ContentItemRow` into `SectionAccordion`
    - Use `useLiveQuery` inside `SectionAccordion` (or lift to `ModuleDetailPage`) to subscribe to `contentItems.where('sectionId').equals(section.id).sortBy('order')`
    - Render one `ContentItemRow` per item; render `AddContentItemForm` at the bottom of the expanded body
    - _Requirements: 7.1, 7.3_

- [x] 9. Implement ProgressState — ProgressControl, optimistic updates, per-section summary
  - [x] 9.1 Implement `ProgressControl` component in `src/components/ProgressControl.tsx`
    - Render three selectable state buttons/tabs: "Not Started", "In Progress", "Completed"
    - Optimistic update: apply `onChange(newState)` immediately in local state
    - On DB failure: revert to previous state and display a toast error
    - _Requirements: 9.1, 9.2, 9.7_

  - [x] 9.2 Implement progress persistence in `src/db/progressService.ts`
    - `setProgressState(contentItemId: number, state: ProgressState): Promise<void>` — updates `progressStates` table
    - `getProgressState(contentItemId: number): Promise<ProgressStateRecord | undefined>`
    - _Requirements: 9.2_

  - [x] 9.3 Write property test for ProgressState persistence and default (Property 13)
    - **Property 13: ProgressState Persistence and Default**
    - **Validates: Requirements 9.2, 9.6**
    - Assert that every newly created ContentItem has state `'Not Started'` immediately after creation; assert that setting any valid `ProgressState` and re-reading it returns the same value

  - [x] 9.4 Wire `ProgressControl` into `ContentItemRow` and wire `progressService` to handle DB writes with optimistic revert
    - Connect `ProgressControl.onChange` to `setProgressState`; on rejection revert the local state in `ContentItemRow`
    - _Requirements: 9.2, 9.7_

  - [x] 9.5 Compute and display per-section progress summary and module completion badge
    - In `SectionAccordion` header, display `X / Y completed` where X = count of items with state `'Completed'` and Y = total items
    - In `ModuleCard`, derive completedCount / totalCount across all sections; show visual completion badge when completedCount === totalCount > 0
    - _Requirements: 9.3, 9.4, 9.5_

- [x] 10. Implement Notes and Markdown editor
  - [x] 10.1 Implement `MarkdownPreview` component in `src/components/MarkdownPreview.tsx`
    - Render `content` prop using `<ReactMarkdown remarkPlugins={[remarkGfm]}>` with support for H1–H3, bold, italic, inline code, code blocks, unordered lists
    - _Requirements: 8.5_

  - [x] 10.2 Implement `MarkdownEditor` component in `src/components/MarkdownEditor.tsx`
    - Accept `value`, `onChange`, `onBlur`, `maxLength` (10000), `mode` (`'edit' | 'preview'`), and `onToggleMode` props matching the design's `MarkdownEditorProps` interface
    - In edit mode: render a `<textarea>` with visible character counter "X / 10000 remaining"; reject input if `value.length > maxLength` and display error message
    - In preview mode: render `MarkdownPreview`
    - A toggle button switches between modes
    - `onBlur` is called when focus leaves the textarea; triggers DB persist upstream
    - _Requirements: 8.1, 8.3, 8.6, 8.7_

  - [x] 10.3 Implement note persistence in `src/db/noteService.ts`
    - `saveNote(contentItemId: number, content: string): Promise<void>` — upserts the Note record (inserts if absent, updates if present) inside a transaction; rejects if `content.length > 10000`
    - `getNote(contentItemId: number): Promise<Note | undefined>`
    - _Requirements: 8.2, 8.3_

  - [x] 10.4 Write property test for note persistence round-trip (Property 12)
    - **Property 12: Note Persistence Round-Trip**
    - **Validates: Requirements 8.2, 8.3**
    - Use `fc.string({ minLength: 1, maxLength: 10000 })` → assert saved content equals retrieved content; use `fc.string({ minLength: 10001 })` → assert `saveNote` rejects and DB retains previous content

  - [x] 10.5 Wire `MarkdownEditor` into `ContentItemRow`
    - Manage `mode` and `value` in `ContentItemRow` local state; call `saveNote` on `onBlur`; display toast error and retain unsaved content if `saveNote` rejects
    - _Requirements: 8.1, 8.2, 8.4_

- [x] 11. Checkpoint — Ensure all tests pass
  - Ensure all tests pass, ask the user if questions arise.

- [x] 12. Implement Web Search integration — SearchAdapter, YouTubeSearchAdapter, composite sort, WebSearchPanel
  - [x] 12.1 Define `SearchAdapter` interface and implement `compositeScore` sort in `src/services/searchService.ts`
    - Export `SearchResult` and `SearchAdapter` interfaces exactly as in the design
    - Implement `compositeScore(r: SearchResult): number` using weights `rating * 1000 + views * 0.001 + domain * 10`
    - Implement `SearchService` class that accepts a `SearchAdapter`, calls `adapter.search(query)`, sorts by composite score, and returns the sorted array
    - _Requirements: 11.1, 11.2_

  - [x] 12.2 Implement `YouTubeSearchAdapter` in `src/services/YouTubeSearchAdapter.ts`
    - Call YouTube Data API v3 `/search` and `/videos` endpoints with the provided API key
    - Map response to `SearchResult[]` including `qualityMetadata` (viewCount, rating from statistics, channelName)
    - _Requirements: 11.1, 11.3_

  - [x] 12.3 Write property test for quality score sort order (Property 14)
    - **Property 14: Quality Score Sort Order**
    - **Validates: Requirements 11.2**
    - Generate arbitrary `SearchResult[]` with arbitrary present/absent `QualityMetadata`; run through `SearchService`; assert every adjacent pair satisfies `compositeScore(a) >= compositeScore(b)`

  - [x] 12.4 Implement `WebSearchPanel` component in `src/components/WebSearchPanel.tsx`
    - `SearchQueryInput` (1–200 chars) with a submit button; call `SearchService.search(query)` on submit
    - Display loading spinner while awaiting results
    - Display error banner if search throws (network/API error); display empty-state message if results array is empty
    - Display one `SearchResultCard` per result: title, URL, `QualityBadge`, and an "Add to Section" button
    - "Add to Section" calls `db.contentItems.add(...)` + default `progressStates` insert inside a transaction; follows append-only order from task 8.1
    - _Requirements: 11.1, 11.3, 11.4, 11.5, 11.6, 11.7_

  - [x] 12.5 Mount `WebSearchPanel` inside `ModuleDetailPage`
    - Pass the current `moduleId` and active `sectionId` (from a section selector or the expanded section); render below `DriveControls` placeholder
    - _Requirements: 11.1_

- [x] 13. Implement Export/Import service
  - [x] 13.1 Implement `ExportImportService` in `src/services/exportImportService.ts`
    - `validate(raw: unknown): raw is ExportFile` — check `formatVersion === 1`, presence of all 7 required array keys, and that each item in each array has the required fields; return false on any violation
    - `export(): Promise<ExportFile>` — read all 7 tables from Dexie in a single `db.transaction('r', [...], ...)` and assemble the `ExportFile` object with `formatVersion: 1` and `exportedAt: new Date().toISOString()`
    - `downloadAsFile(data: ExportFile): void` — create a `Blob`, create an object URL, programmatically click a hidden `<a>` element with `download="backup-YYYY-MM-DD.json"` (date in local timezone)
    - `import(file: ExportFile): Promise<void>` — inside a single `db.transaction('rw', [...allTables], ...)`: clear all 7 tables, then `bulkAdd` each array from the file; let any error propagate to trigger automatic Dexie rollback
    - _Requirements: 12.1, 12.2, 12.5, 12.6_

  - [x] 13.2 Write property test for export–import round-trip fidelity (Property 15)
    - **Property 15: Export–Import Round-Trip Fidelity**
    - **Validates: Requirements 12.6**
    - Generate arbitrary `AppState` with `fast-check`; seed the database; export; clear; import; export again; assert second export deep-equals the first

  - [x] 13.3 Write property test for invalid ExportFile rejected without state mutation (Property 16)
    - **Property 16: Invalid ExportFile Is Rejected Without State Mutation**
    - **Validates: Requirements 12.5**
    - Use `fc.anything()` filtered to objects that fail at least one schema rule; assert `validate(raw)` returns `false` and DB record count is unchanged

  - [x] 13.4 Implement `ExportImportToolbar` component in `src/components/ExportImportToolbar.tsx`
    - Export button: calls `exportImportService.export()` then `downloadAsFile()`; shows toast on error
    - Import button: opens a hidden `<input type="file" accept=".json">`; on file selected, parse JSON, run `validate`; if invalid show error listing the violated rule; if valid show confirmation modal warning about data replacement
    - Confirmation modal: Cancel → dismiss, no DB write; Confirm → call `exportImportService.import(...)`; show toast on success or error
    - Mount `ExportImportToolbar` in the top navigation bar rendered by `App.tsx`
    - _Requirements: 12.1, 12.2, 12.3, 12.4, 12.5_

- [x] 14. Implement Google Drive Sync — GIS PKCE auth, DriveAuthContext, GoogleDriveService, DriveControls
  - [x] 14.1 Create `DriveAuthContext` in `src/contexts/DriveAuthContext.tsx`
    - Context stores `accessToken: string | null` (from `sessionStorage`) and exposes `setToken`, `clearToken`, `isAuthenticated`
    - Token is written to and read from `sessionStorage` only (never `localStorage`)
    - _Requirements: 13.1, 13.6_

  - [x] 14.2 Implement `GoogleDriveService` in `src/services/googleDriveService.ts`
    - `authenticate(): Promise<void>` — initiates GIS PKCE popup using `google.accounts.oauth2.initTokenClient`; stores resulting access token via `DriveAuthContext`
    - `isAuthenticated(): boolean` — checks `sessionStorage` for a valid token
    - `syncToDrive(data: ExportFile, onProgress: (p: DriveProgress) => void): Promise<void>` — phase `authenticating` → look up "StudyOrganizer" folder by name; if not found create via `drive.files.create`; phase `uploading` → upload as `multipart/form-data` to `https://www.googleapis.com/upload/drive/v3/files?uploadType=multipart`; phase `done`
    - `restoreFromDrive(onProgress: (p: DriveProgress) => void): Promise<ExportFile>` — phase `authenticating` → look up folder → find "studydata-backup.json" → phase `downloading` → GET file content → parse and return as `ExportFile`; phase `done`
    - All API failures set phase to `error` and throw
    - _Requirements: 13.1, 13.2, 13.3, 13.5, 13.7_

  - [x] 14.3 Implement `DriveControls` component in `src/components/DriveControls.tsx`
    - When not authenticated: show "Connect Google Drive" button only; no Sync or Restore buttons
    - When authenticated: show "Sync to Drive" and "Restore from Drive" buttons
    - While a Drive operation is in progress (`phase` ≠ `done` and ≠ `error`): show progress indicator; disable both Sync and Restore buttons
    - On authentication failure: display error message with reason
    - On Drive API failure: display error message indicating upload vs. download; DB is not modified
    - _Requirements: 13.4, 13.5, 13.6, 13.8_

  - [x] 14.4 Mount `DriveControls` inside `ModuleDetailPage`
    - Wrap `App.tsx` with `<DriveAuthContext.Provider>` so the token is accessible everywhere
    - Render `DriveControls` in `ModuleDetailPage` below the section list
    - _Requirements: 13.1, 13.6_

- [x] 15. Configure PWA / Service Worker and PwaInstallBanner
  - [x] 15.1 Configure `vite-plugin-pwa` in `vite.config.ts`
    - Set `registerType: 'autoUpdate'` and `injectRegisterSW: true`
    - Precache the app shell: HTML, compiled JS/CSS chunks, fonts, and icons
    - Add `runtimeCaching` entries: stale-while-revalidate for GIS script (`accounts.google.com/gsi`); network-only for search API calls and all `googleapis.com` requests
    - Create `public/manifest.webmanifest` with `name`, `short_name`, `start_url`, `display: 'standalone'`, `background_color`, `theme_color`, and icon entries for 192×192 and 512×512
    - _Requirements: 1.2, 1.3_

  - [x] 15.2 Implement `PwaInstallBanner` component in `src/components/PwaInstallBanner.tsx`
    - Create `PwaInstallContext` that captures the `beforeinstallprompt` event and stores the deferred prompt
    - `PwaInstallBanner` shows a visible install button; on click calls `deferredPrompt.prompt()` and hides the banner
    - Render `PwaInstallBanner` as the first child of `App.tsx` (above the router)
    - _Requirements: 1.4_

- [x] 16. Implement remaining property-based tests
  - [x] 16.1 Write property test for section expand/collapse idempotence — if not already covered in task 7.5
    - **Property 10: Section Expand/Collapse Toggle Is Idempotent Over Two Clicks**
    - **Validates: Requirements 6.4**
    - (Skip if task 7.5 was already implemented)

- [x] 17. Checkpoint — Ensure all tests pass
  - Ensure all tests pass, ask the user if questions arise.

- [x] 18. Apply responsive layout polish — Tailwind breakpoints, touch targets, no horizontal overflow
  - [x] 18.1 Audit and enforce Tailwind breakpoint rules across all page components
    - `DashboardPage` Module grid: `grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4`
    - `ModuleDetailPage` SectionAccordion body: single-column below `sm`, two-column `sm`+
    - Verify content renders without horizontal scroll from 320px to 1920px viewport (add `max-w-full overflow-x-hidden` to root where needed)
    - _Requirements: 2.1, 2.2, 2.3, 2.4, 2.6_

  - [x] 18.2 Ensure all interactive elements meet 44×44 CSS pixel touch target minimum on mobile
    - Check `ProgressControl` state buttons, `TagFilterChip`, `TagBadge` remove button, section accordion toggle, delete icon button
    - Add `min-h-[44px] min-w-[44px]` (or equivalent padding) to any element that falls below this threshold below the `sm` breakpoint
    - _Requirements: 2.5_

- [x] 19. Final checkpoint — Ensure all tests pass
  - Ensure all tests pass, ask the user if questions arise.

- [x] 20. Search-driven module generation (Discover flow)
  - [x] 20.1 Implement `classifyContentType` util in `src/utils/classifyContentType.ts`
    - Pure function mapping a `SearchResult` to `'Video' | 'Article' | 'Course' | 'Book'`
    - Video when `source` is the YouTube adapter; Course/Book via exported host-pattern tables; Article as the catch-all; malformed URL -> Article
    - Export the `ContentType` type and the host-pattern constant tables for test reuse
    - _Requirements: 14.3_

  - [x] 20.2 Implement `GoogleCseSearchAdapter` in `src/services/GoogleCseSearchAdapter.ts`
    - `implements SearchAdapter`, `name = 'web'`, constructor `(apiKey: string, cx: string)`
    - Calls the Programmable Search JSON API via `fetch`; maps items to `SearchResult` with `source: 'web'` and `qualityMetadata.domainName`
    - Throws descriptive errors on missing credentials, HTTP, or JSON failure (mirrors `YouTubeSearchAdapter`)
    - _Requirements: 14.2, 14.11_

  - [x] 20.3 Implement module generation service in `src/services/moduleGenerationService.ts`
    - `generateModulePreview(query, results)` — pure: classify, group into fixed-order non-empty sections, sort each via `sortByCompositeScore`
    - `saveGeneratedModule(preview)` — one `db.transaction('rw', [modules, sections, contentItems, progressStates], ...)`: module -> sections -> append-order content items -> default 'Not Started' progress; returns new module id
    - _Requirements: 14.4, 14.5, 14.6_

  - [x] 20.4 Write tests for classification and generation
    - Unit tests for `classifyContentType` across video/course/book/article and malformed URLs
    - **Property: grouping is total and disjoint** — every input result appears in exactly one section; no result dropped or duplicated (numRuns: 200)
    - Save round-trip: `saveGeneratedModule` persists the expected module/sections/items counts (fake-indexeddb)
    - _Requirements: 14.3, 14.4, 14.6_

  - [x] 20.5 Implement `DiscoverSearchPanel` and `GeneratedModulePreview` components
    - `DiscoverSearchPanel`: Discover query input + credential inputs (YouTube key, CSE key, CSE cx) persisted to `localStorage`; runs available adapters; loading/error/empty states (Req 14.9/14.10); skips web source with a notice when CSE creds are absent
    - `GeneratedModulePreview`: editable module name (defaults to query, 1-100 chars), grouped sections with results (reuse `QualityBadge`), 'Save module' and 'Discard'; nothing persisted until Save
    - _Requirements: 14.1, 14.2, 14.5, 14.8, 14.9, 14.10, 14.11_

  - [x] 20.6 Wire the Discover flow into `DashboardPage`
    - Mount `DiscoverSearchPanel` above the saved-modules area, visually distinct from the name/tag filter
    - On save, rely on the existing `useLiveQuery` subscription to show the new `ModuleCard` within 1s (no reload)
    - Keep manual create, manual sections, and per-section web search unchanged (Req 14.12)
    - _Requirements: 14.1, 14.7, 14.12_

  - [x] 20.7 Checkpoint — Ensure build and all tests pass
    - Run `npm run build` and `npm test`; ask the user if questions arise

- [x] 21. Settings page, selective backup, and minimalist dashboard
  - [x] 21.1 Add `SettingsPage` route and a credentials store
    - Add route `#/settings` -> `SettingsPage` in `App.tsx`; add a Settings gear control on the Dashboard that navigates to it; Back control returns to `/`
    - Create `src/services/credentialsStore.ts` wrapping localStorage for `youtube_api_key`, `google_cse_api_key`, `google_cse_cx` (get/set/clear + isSet)
    - _Requirements: 15.1, 15.3, 15.4_

  - [x] 21.2 Implement `ApiKeysSettings` section
    - Inputs for the three credentials, each showing Set/Not set status with update and clear actions; values persisted via the credentials store only
    - _Requirements: 15.3, 15.4, 15.5_

  - [x] 21.3 Implement `exportSelectedModules` and `ModuleSelectList`
    - `exportSelectedModules(moduleIds)` in `exportImportService.ts` builds a self-contained `ExportFile` from only the selected modules and their dependent rows (sections, contentItems, notes, progressStates, associations, referenced tags)
    - `ModuleSelectList` reusable checkbox list with select-all/none, defaults to all selected, disables confirm when none selected
    - _Requirements: 16.1, 16.2, 16.9_

  - [x] 21.4 Implement merge import with ID remapping (`importModulesMerge`)
    - Single `db.transaction('rw', [...all tables])`, never clears existing data; assigns fresh ids and remaps module->section->contentItem->note/progress and module->tag relationships; reuses existing tags by name
    - _Requirements: 16.3, 16.4, 16.5, 16.6_

  - [x] 21.5 Write tests for selective export and merge import
    - **Property: selective round-trip** — export selected modules then import into an EMPTY db reproduces those modules/sections/items/notes/progress/tags identically with ordering preserved (Req 16.10)
    - **Property: merge preserves existing** — importing into a NON-empty db leaves all pre-existing modules unchanged, adds the imported trees intact, and creates no duplicate tag names (Req 16.4-16.6)
    - _Requirements: 16.4, 16.5, 16.6, 16.10_

  - [x] 21.6 Implement `ExportImportSettings` and move Drive into Settings
    - `ExportImportSettings`: export uses `ModuleSelectList`; import parses+validates, then shows the file's modules in a `ModuleSelectList`, then calls `importModulesMerge`
    - Add `DriveSettings` using the existing `DriveControls`; remove `ExportImportToolbar` and `DriveControls` from `ModuleDetailPage` (no duplication)
    - _Requirements: 15.2, 15.6, 16.1, 16.3, 16.7, 16.8_

  - [x] 21.7 Rework `DashboardPage` into the minimalist layout
    - Slim top bar (title + Settings gear); centered Discover search hero with generous whitespace; secondary Create-module-manually action; saved-module grid below with the name/tag filter kept compact/secondary
    - Keep all existing manual flows working; verify no horizontal overflow and 44px touch targets
    - _Requirements: 14.1, 15.1, 2.1, 2.5, 2.6_

  - [x] 21.8 Checkpoint — Ensure build and all tests pass
    - Run `npm run build` and `npm test`; ask the user if questions arise

- [x] 22. Editing, deletion, and selective Discover save
  - [x] 22.1 Implement cascade-delete service in `src/db/deleteService.ts`
    - `deleteContentItem(id)` — removes the content item + its note + its progress state in one `db.transaction('rw', ...)`
    - `deleteSection(id)` — removes the section + all its content items + their notes/progress, atomically
    - `deleteModule(id)` — removes the module + all sections + content items + notes/progress + moduleTagAssociations (tags preserved), atomically
    - _Requirements: 18.4, 18.5, 18.6, 18.7_

  - [x] 22.2 Write tests for cascade deletes
    - **Property: no orphans** — after each delete, no descendant rows remain pointing at a removed parent; unrelated modules/sections/items are untouched; tag entities survive a module delete (numRuns ~30, fake-indexeddb)
    - _Requirements: 18.4, 18.5, 18.6_

  - [x] 22.3 Implement reusable `ConfirmDialog` component in `src/components/ConfirmDialog.tsx`
    - Props: title, message, confirmLabel, onConfirm, onCancel; role=dialog, aria-modal, focus trap, Escape + backdrop cancel; 44px targets
    - _Requirements: 18.2, 18.3_

  - [x] 22.4 Add ContentItem edit + delete to `ContentItemRow`
    - Edit toggle swaps the row for an inline editor (title + url, optional quality rating) reusing the add-form validation; Save calls `db.contentItems.update`; Cancel restores saved values
    - Delete (trash) button opens `ConfirmDialog` → `deleteContentItem`
    - _Requirements: 17.1, 17.2, 17.3, 17.4, 17.6, 18.1, 18.4_

  - [x] 22.5 Add Section rename + delete to `SectionAccordion`
    - Inline rename affordance in the header (validate 1-100 non-whitespace); Save calls `db.sections.update`
    - Delete button opens `ConfirmDialog` (names the section, warns it removes N items) → `deleteSection`
    - _Requirements: 17.5, 17.6, 18.1, 18.5_

  - [x] 22.6 Add Module delete to `ModuleHeader` / `ModuleDetailPage`
    - Delete button opens `ConfirmDialog` (warns it removes the whole module) → `deleteModule` → `navigate('/')`
    - _Requirements: 18.1, 18.6_

  - [x] 22.7 Add selective save to `GeneratedModulePreview`
    - Per-result checkbox (default all selected) + per-section select/clear-all toggle; derive a filtered preview on Save that keeps only selected results and drops empty sections; disable Save when none selected with an inline hint; preserve composite-score ordering
    - _Requirements: 19.1, 19.2, 19.3, 19.4, 19.5_

  - [x] 22.8 Checkpoint — Ensure build and all tests pass
    - Run `npm run build` and `npm test`; ask the user if questions arise

- [x] 23. Content item thumbnails
  - [x] 23.1 Implement `getThumbnail` util in `src/utils/thumbnail.ts`
    - Pure function: parse URL (try/catch -> { kind: 'none' }); YouTube (watch?v=, youtu.be, /shorts/, /embed/) -> `https://img.youtube.com/vi/<id>/mqdefault.jpg` (kind 'youtube'); otherwise favicon `https://www.google.com/s2/favicons?domain=<host>&sz=64` (kind 'favicon'). Export `ThumbnailKind`/`ThumbnailInfo`.
    - _Requirements: 20.2, 20.3, 20.5_

  - [x] 23.2 Write unit tests for `getThumbnail`
    - youtube watch/youtu.be/shorts/embed all yield the correct mqdefault URL; a non-youtube URL yields the favicon URL for its host; a malformed URL yields kind 'none'
    - _Requirements: 20.2, 20.3, 20.4_

  - [x] 23.3 Implement `ContentThumbnail` component in `src/components/ContentThumbnail.tsx`
    - Props `{ url, contentType?, className? }`; ~44px rounded square; `<img alt="">` with onError swapping to a type-based inline-SVG fallback icon (video/course/book/document); wrapper `aria-hidden`; kind 'none' shows the fallback immediately; no horizontal overflow
    - _Requirements: 20.1, 20.4, 20.6, 20.7_

  - [x] 23.4 Mount `ContentThumbnail` in `ContentItemRow` and `GeneratedModulePreview`
    - ContentItemRow display mode: thumbnail before the title, title/metadata in a `flex-1 min-w-0` column (titles wrap, no overflow); edit mode unchanged
    - GeneratedModulePreview result rows: thumbnail at the left, passing the section's content type for the fallback
    - _Requirements: 20.1, 20.6, 20.7_

  - [x] 23.5 Checkpoint — Ensure build and all tests pass
    - Run `npm run build` and `npm test`; ask the user if questions arise

- [x] 24. Collapsible tag filter cloud
  - [x] 24.1 Refactor `TagFilter` into a collapsed-by-default tag cloud in `src/components/TagFilter.tsx`
    - Add internal `expanded` state (default collapsed); keep `TagFilterProps` unchanged and return `null` when `availableTags` is empty
    - Header row: a "Filter by tags" trigger button with `aria-expanded` + `aria-controls`, an active-selection count (e.g. "Filter by tags (3)"), and a chevron that rotates when expanded; render the selected tags as removable `TagFilterChip`s inline next to the trigger (visible in both states)
    - When expanded, render the full chip cloud (`flex flex-wrap gap-2`, id matched by `aria-controls`) in-flow below the header so only the open state pushes the grid down; all chips use the existing `onToggle`
    - No change to chip styling, the 50-char label cap, AND-filter semantics, the store, or `filterModules`
    - _Requirements: 5.9, 5.10, 5.11, 5.12, 5.13, 5.14_

  - [x] 24.2 Checkpoint — Ensure build and all tests pass
    - Run `npm run build` and `npm test`; ask the user if questions arise

- [x] 25. Global navigation header on every page
  - [x] 25.1 Extract the nav bar into `src/components/AppHeader.tsx` and render it around all routes via a layout in `src/App.tsx`
    - Move the consolidated bar (wordmark/Home, Home icon, module-search input, Create, Settings gear) out of `DashboardPage` into `AppHeader`; use `useNavigate`/`useLocation`/`useSearchParams` (rendered inside `HashRouter`)
    - Add an `AppLayout` under `<HashRouter>` that renders `<AppHeader />` above the routed content so the header shows on Dashboard, Module detail, and Settings
    - Search input writes the term to the `q` URL query param and navigates to `/` on change (<=100ms debounce) and submit; value derives from current `q`; Create opens the modal on the Dashboard or navigates to `/` with `new=1` from other pages; Home/Settings navigate (no-op safe); responsive (labels collapse to icons, >=44px targets, no horizontal overflow)
    - _Requirements: 21.1, 21.2, 21.3, 21.4, 21.6, 21.8_

  - [x] 25.2 Wire `DashboardPage` to the URL params and drop its local top bar
    - Remove the in-body bar/`SearchBar`; initialize and sync the name-filter from the `q` param via `useSearchParams`, preserving Req 5 name semantics and the unchanged `filterModules`/tag-filter combine
    - If `new=1` is present, open `CreateModuleModal` on mount then clear that param; keep the collapsible tag cloud in the Dashboard body (not the header)
    - Remove any local top-bar markup from `ModuleDetailPage` and `SettingsPage`
    - _Requirements: 21.5, 21.6, 21.7, 21.9_

  - [x] 25.3 Checkpoint — Ensure build and all tests pass
    - Run `npm run build` and `npm test`; ask the user if questions arise

- [x] 26. Multi-source search with trust-aware ranking
  - [x] 26.1 Extend quality signals, composite score, and classification
    - Add optional `communityScore`, `authorReputation`, `sourceAuthority`, and `signals: string[]` to `QualityMetadata` in `src/db/types.ts` (all optional; backward compatible with existing exports)
    - In `src/services/searchService.ts`, extend `compositeScore` with an authority term (combine `sourceAuthority` with log-normalized `authorReputation`) and fold a log-normalized `communityScore` into the rating term via max(rating/5, normalizedCommunity); keep weights as inspectable constants (rating 1.0, authority 0.7, view 0.5, domain 0.25); keep it pure and `sortByCompositeScore` unchanged
    - In `src/utils/classifyContentType.ts`, classify results whose `source` is the Open Library id as Book before the URL heuristics; keep YouTube->Video and existing host tables; grouping stays total and disjoint
    - Update/extend unit tests for `compositeScore` (new terms, normalization bounds, no-metadata still 0) and `classifyContentType` (Open Library -> Book)
    - _Requirements: 22.4, 22.5, 22.6, 22.7, 22.8_

  - [x] 26.2 Create the source registry in `src/services/searchSources.ts`
    - Export `SourceDescriptor` ({ id, label, authority, requiresCredentials, build }) and `SEARCH_SOURCES` with baseline authority weights (indicative: Wikipedia 0.9, Stack Exchange 0.8, Open Library 0.75, YouTube 0.6, Dev.to 0.55, Google CSE 0.4)
    - Export `buildActiveAdapters()` returning `{ adapters, skipped }`: credential-free sources always build; YouTube/Google CSE build only when their stored credentials exist (read via credentialsStore); skipped descriptors drive the non-blocking notice
    - Add a shared `runMultiSourceSearch(query)` helper (build -> Promise.allSettled -> flatten successful -> sortByCompositeScore) so every surface shares identical behavior
    - Unit-test `buildActiveAdapters` (with/without credentials) and `runMultiSourceSearch` fan-out (one source failing still returns others)
    - _Requirements: 22.1, 22.2, 22.3, 22.10, 22.11, 22.12, 22.13_

  - [x] 26.3 Implement `WikipediaSearchAdapter` in `src/services/WikipediaSearchAdapter.ts`
    - MediaWiki search (`origin=*`, CORS); map hits to article-page results; map size/pageviews into the popularity (view) signal; set `sourceAuthority` and `signals` (e.g. "reference"); tag `source`
    - Unit tests with mocked fetch (MSW): mapping, signals populated only from real fields, failure rejects
    - _Requirements: 22.1, 22.4, 22.5, 22.8_

  - [x] 26.4 Implement `StackExchangeSearchAdapter` in `src/services/StackExchangeSearchAdapter.ts`
    - Stack Exchange 2.3 `/search/advanced` (site=stackoverflow, CORS, no key); carry score (votes) as `communityScore` and asker/answerer reputation as `authorReputation`; build `signals` (e.g. "1.2k votes"); tag `source`
    - Unit tests with mocked fetch: votes/reputation mapped, signals real-only, failure rejects
    - _Requirements: 22.1, 22.5, 22.6, 22.8_

  - [x] 26.5 Implement `OpenLibrarySearchAdapter` in `src/services/OpenLibrarySearchAdapter.ts`
    - Open Library `/search.json` (CORS, no key); URL -> work page; edition/readinglog counts into popularity; set `sourceAuthority`/`signals`; tag `source` so classification yields Book
    - Unit tests with mocked fetch: mapping + Book classification end-to-end
    - _Requirements: 22.1, 22.4, 22.5, 22.8_

  - [x] 26.6 Implement `DevToSearchAdapter` in `src/services/DevToSearchAdapter.ts`
    - dev.to `/api/articles?q=`; positive reactions + comments into `communityScore`; set `sourceAuthority`/`signals`; tag `source`; best-effort (CORS/HTTP failure rejects and is skipped by the fan-out)
    - Unit tests with mocked fetch: mapping, best-effort failure handling
    - _Requirements: 22.1, 22.5, 22.8, 22.10_

  - [x] 26.7 Add `SourceBadge` and wire both search surfaces to multi-source
    - Create `src/components/SourceBadge.tsx` rendering the source label + real `qualityMetadata.signals` (never placeholders); mount it beside `QualityBadge` in `GeneratedModulePreview` and `WebSearchPanel` result rows
    - Refactor `DiscoverSearchPanel` to use `runMultiSourceSearch`/`buildActiveAdapters` and show a generalized skipped-source notice
    - Refactor `WebSearchPanel` to use the same fan-out so per-section search returns all media types; update its heading/help text; keep the add-to-section behavior unchanged
    - _Requirements: 22.3, 22.8, 22.9, 22.11_

  - [x] 26.8 Checkpoint — Ensure build and all tests pass
    - Run `npm run build` and `npm test`; ask the user if questions arise

- [ ] 27. Three-column dashboard with pinning and reordering
  - [x] 27.1 Add the `dashboardPrefs` store (Dexie v2) and its type
    - Add `DashboardColumn` (`'all' | 'started'`) and `DashboardPref` ({ id?, moduleId, column, pinned, order }) to `src/db/types.ts`
    - In `src/db/db.ts` add `dashboardPrefs!: Table<DashboardPref, number>` and `this.version(2).stores({ ...all v1 stores..., dashboardPrefs: '++id, moduleId, column, [moduleId+column]' })` (additive migration, no data transform)
    - _Requirements: 23.9_

  - [x] 27.2 Implement `dashboardPrefsService` and wire delete cleanup
    - Create `src/db/dashboardPrefsService.ts` with `getPrefs(column)`, `setPinned(moduleId, column, pinned)`, `reorder(column, orderedModuleIds)` (dense 0..n-1), and `removeModulePrefs(moduleId)`; all multi-row writes in one `db.transaction('rw', db.dashboardPrefs, ...)`
    - Extend `deleteModule` in `src/db/deleteService.ts` to call `removeModulePrefs(moduleId)` within its existing cascade transaction (add dashboardPrefs to the transaction tables) so deleting a module clears its pins/order in both columns
    - Service tests (fake-indexeddb): setPinned upsert, reorder writes dense order, removeModulePrefs clears both columns, and a cascade-delete test asserting no orphan dashboardPrefs rows remain
    - _Requirements: 23.9, 23.10_

  - [x] 27.3 Implement pure started-detection and column-ordering utils with tests
    - `src/utils/isModuleStarted.ts`: `isModuleStarted(moduleId, sections, contentItems, progressStates)` — true when any ContentItem under the module has ProgressState In Progress or Completed
    - `src/utils/orderModulesForColumn.ts`: pure `orderModulesForColumn(modules, prefsForColumn)` — pinned first then unpinned, each group by `order` asc, deterministic fallback (updatedAt desc, then id) for modules with no pref row; total permutation (no drops/dupes)
    - Unit/property tests: pinned-before-unpinned, stable deterministic fallback, output is a permutation of input; started predicate for In Progress/Completed vs Not Started/empty
    - _Requirements: 23.2, 23.5, 23.8_

  - [x] 27.4 Add the drag-and-drop dependency
    - Install `@dnd-kit/core`, `@dnd-kit/sortable`, and `@dnd-kit/utilities` (pinned versions) as runtime dependencies for accessible pointer+keyboard reordering
    - Verify the app still type-checks and builds after install
    - _Requirements: 23.6, 23.13_

  - [x] 27.5 Build `ModuleColumn` and add pin/drag affordances to `ModuleCard`
    - Create `src/components/ModuleColumn.tsx`: a titled, independently scrollable column (`overflow-y-auto`, bounded height via flex `min-h-0`) wrapping a dnd-kit `SortableContext` (vertical list) of module cards; per-column empty state; raises `onReorder(orderedModuleIds)` and `onTogglePin(moduleId)`; keyboard sensor enabled; drags clamped so pinned items stay in the pinned group
    - Extend `src/components/ModuleCard.tsx` with optional `pinned`, `onTogglePin`, and drag-handle props (a pin button with `aria-pressed`, a drag handle); existing card content unchanged and the affordances render only when used inside a column (Discover preview reuse unaffected)
    - _Requirements: 23.4, 23.5, 23.6, 23.13_

  - [ ] 27.6 Refactor `DashboardPage` into the responsive three-column layout
    - Replace the centered single column with a responsive grid: left `ModuleColumn` (all modules), center `DiscoverSearchPanel`, right `ModuleColumn` (started modules); wide = 3 columns with independent side scroll, narrow = stacked Discover -> all -> started
    - Extend the live query to also read `dashboardPrefs`; left list = `orderModulesForColumn(filterModules(all, q, tags), allPrefs)` (keep the TagFilter + Req 5 filtering on the left), right list = `orderModulesForColumn(startedModules, startedPrefs)` using `isModuleStarted`; wire `onReorder`/`onTogglePin` per column to `dashboardPrefsService`; keep the global AppHeader and `new=1`/`q` handling intact
    - _Requirements: 23.1, 23.2, 23.3, 23.4, 23.7, 23.11, 23.12_

  - [ ] 27.7 Checkpoint — Ensure build and all tests pass
    - Run `npm run build` and `npm test`; ask the user if questions arise

- [ ] 28. Dark mode
  - [ ] 28.1 Theme infrastructure: Tailwind class strategy, theme module, provider, anti-flash
    - Set `darkMode: 'class'` in `tailwind.config.js`
    - Create `src/theme/theme.ts`: `type Theme = 'light'|'dark'`, `getInitialTheme()` (stored localStorage value > prefers-color-scheme > 'light'), `applyTheme(theme)` (toggle `dark` class on `document.documentElement` + persist to localStorage)
    - Add a `ThemeProvider` + `useTheme()` hook (`src/theme/ThemeContext.tsx`) holding theme state from `getInitialTheme()`, applying it in an effect, exposing `toggleTheme()`; wrap the app in `App.tsx` (alongside DriveAuthProvider)
    - Add a tiny inline script in `index.html` that sets the `dark` class on `<html>` from localStorage/prefers-color-scheme before first paint (Req 24.9)
    - Unit tests for `getInitialTheme` precedence and `applyTheme` (root class + localStorage) with mocked matchMedia/localStorage
    - _Requirements: 24.4, 24.5, 24.8, 24.9_

  - [ ] 28.2 ThemeToggle component and header placement
    - Create `src/components/ThemeToggle.tsx`: button with `aria-pressed`, dynamic `aria-label`, sun/moon icon by theme, >=44px target, focus ring; uses `useTheme()`
    - Mount it in `AppHeader` immediately to the right of the Settings button (Req 24.1)
    - Light render test: aria-pressed reflects theme; clicking toggles
    - _Requirements: 24.1, 24.2, 24.3, 24.6_

  - [ ] 28.3 Dark-style the core surfaces
    - Add a dark base on the app shell/root (`dark:bg-gray-900 dark:text-gray-100`) so unstyled surfaces stay legible (Req 24.10)
    - Add `dark:` variants to `AppHeader` (bg/border/icons/search input), `DashboardPage` background + column headings, `ModuleColumn` (headings/empty text), and `ModuleCard` (card bg/border/name/meta/tags) meeting AA contrast (Req 24.7)
    - _Requirements: 24.3, 24.7, 24.10_

  - [ ] 28.4 Checkpoint — Ensure build and all tests pass
    - Run `npm run build` and `npm test`; ask the user if questions arise

## Notes

- Tasks marked with `*` are optional and can be skipped for faster MVP delivery
- Each task references specific requirements for traceability
- Checkpoints at tasks 11, 17, and 19 ensure incremental validation
- Property-based tests use `fast-check` with `numRuns: 100` for async properties and `numRuns: 200` for pure function properties
- All Dexie multi-table writes use `db.transaction('rw', [...tables], ...)` for atomic rollback
- The `filterModules` pure function is shared between the UI and property tests — no mocking required
- Google Drive token is stored in `sessionStorage` only (never `localStorage`) per security requirements
- The `compositeScore` sort is applied in `SearchService` after the adapter returns results, never inside the adapter

## Task Dependency Graph

```json
{
  "waves": [
    { "id": 0, "tasks": ["1.1", "1.2"] },
    { "id": 1, "tasks": ["2.1", "3.1"] },
    { "id": 2, "tasks": ["2.2"] },
    { "id": 3, "tasks": ["2.3", "2.4", "4.1", "4.2"] },
    { "id": 4, "tasks": ["4.3", "5.1", "5.2", "5.3"] },
    { "id": 5, "tasks": ["5.4", "5.5", "5.6", "5.7", "5.8", "6.1"] },
    { "id": 6, "tasks": ["6.2", "6.3", "6.4", "6.5", "6.6"] },
    { "id": 7, "tasks": ["6.7", "7.1", "7.2", "7.3"] },
    { "id": 8, "tasks": ["7.4", "7.5", "8.1", "8.3"] },
    { "id": 9, "tasks": ["8.2", "8.4", "9.1", "9.2"] },
    { "id": 10, "tasks": ["8.5", "9.3", "9.4", "10.1", "10.3"] },
    { "id": 11, "tasks": ["9.5", "10.2", "10.4"] },
    { "id": 12, "tasks": ["10.5", "12.1"] },
    { "id": 13, "tasks": ["12.2", "12.3", "13.1"] },
    { "id": 14, "tasks": ["12.4", "13.2", "13.3", "14.1"] },
    { "id": 15, "tasks": ["12.5", "13.4", "14.2"] },
    { "id": 16, "tasks": ["14.3"] },
    { "id": 17, "tasks": ["14.4", "15.1"] },
    { "id": 18, "tasks": ["15.2", "16.1"] },
    { "id": 19, "tasks": ["18.1", "18.2"] },
    { "id": 20, "tasks": ["20.1", "20.2"] },
    { "id": 21, "tasks": ["20.3"] },
    { "id": 22, "tasks": ["20.4", "20.5"] },
    { "id": 23, "tasks": ["20.6"] },
    { "id": 24, "tasks": ["20.7"] },
    { "id": 25, "tasks": ["21.1", "21.3"] },
    { "id": 26, "tasks": ["21.2", "21.4"] },
    { "id": 27, "tasks": ["21.5", "21.6"] },
    { "id": 28, "tasks": ["21.7"] },
    { "id": 29, "tasks": ["21.8"] },
    { "id": 30, "tasks": ["22.1", "22.3"] },
    { "id": 31, "tasks": ["22.2", "22.7"] },
    { "id": 32, "tasks": ["22.4", "22.5"] },
    { "id": 33, "tasks": ["22.6"] },
    { "id": 34, "tasks": ["22.8"] },
    { "id": 35, "tasks": ["23.1", "23.3"] },
    { "id": 36, "tasks": ["23.2", "23.4"] },
    { "id": 37, "tasks": ["23.5"] },
    { "id": 38, "tasks": ["24.1"] },
    { "id": 39, "tasks": ["24.2"] },
    { "id": 40, "tasks": ["25.1"] },
    { "id": 41, "tasks": ["25.2"] },
    { "id": 42, "tasks": ["25.3"] },
    { "id": 43, "tasks": ["26.1", "26.2"] },
    { "id": 44, "tasks": ["26.3", "26.4"] },
    { "id": 45, "tasks": ["26.5", "26.6"] },
    { "id": 46, "tasks": ["26.7"] },
    { "id": 47, "tasks": ["26.8"] },
    { "id": 48, "tasks": ["27.1", "27.4"] },
    { "id": 49, "tasks": ["27.2", "27.3"] },
    { "id": 50, "tasks": ["27.5"] },
    { "id": 51, "tasks": ["27.6"] },
    { "id": 52, "tasks": ["27.7"] },
    { "id": 53, "tasks": ["28.1"] },
    { "id": 54, "tasks": ["28.2"] },
    { "id": 55, "tasks": ["28.3"] },
    { "id": 56, "tasks": ["28.4"] }
  ]
}
```
