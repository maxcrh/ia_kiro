# Requirements Document

## Introduction

The Study Materials Organizer is a 100% client-side (serverless) Progressive Web Application (PWA) built with React, TypeScript, Tailwind CSS, and Dexie.js. It allows users to organize, store, and track study materials and web links entirely in the browser without any backend server or external database. The application persists all data locally via IndexedDB (through Dexie.js) and provides optional backup and synchronization features. The UI must be fully responsive for both desktop and mobile screens.

## Glossary

- **Application**: The Study Materials Organizer SPA/PWA.
- **Dashboard**: The main/home page displaying all modules as cards.
- **Module**: A top-level study unit grouping related content (e.g., "React Fundamentals", "UX Design").
- **ModuleCard**: A UI card component representing a single Module on the Dashboard.
- **Tag**: A user-defined label (e.g., "React", "UX", "Beginner", "Priority") that can be assigned to a Module for classification.
- **Section**: A named, collapsible group of content items within a Module's detail page.
- **ContentItem**: A single study resource (URL + title) stored inside a Section.
- **Note**: An editable text field (supporting plain text or Markdown) attached to a ContentItem for key takeaways.
- **ProgressState**: An enumerated status ("Not Started", "In Progress", "Completed") assigned to a ContentItem.
- **Database**: The Dexie.js IndexedDB database instance used as the primary persistence layer.
- **SearchQuery**: A text string entered by the user to find Modules or online resources.
- **WebSearch**: The client-side integration that queries external public APIs to find online study resources ranked by quality and engagement metrics.
- **QualityMetadata**: Engagement and relevance data attached to a web search result (e.g., view count, rating, channel/domain relevance score).
- **ExportFile**: A JSON file representing the full application state exported by the user.
- **DriveSync**: The Google Drive API client-side integration for backing up and restoring the ExportFile.
- **PWA**: Progressive Web App — a web application installable on the user's device and capable of offline operation.

---

## Requirements

### Requirement 1: Client-Side Only Architecture

**User Story:** As a user, I want the application to run entirely in my browser without any backend server, so that I can use it offline and without creating an account.

#### Acceptance Criteria

1. THE Application SHALL execute all logic, state management, and data persistence exclusively in the browser, making no outbound network requests to any server-side endpoint at runtime.
2. WHEN the Application is loaded for the first time with a network connection, THE Application SHALL cache all static assets (HTML, CSS, JavaScript, fonts, and icons) in the browser's service worker cache so that subsequent loads do not require a network connection.
3. IF the device has no network connection after the initial page load, THEN THE Application SHALL serve all static assets from the browser cache and render the full UI without displaying a network error.
4. WHERE the PWA install prompt is supported by the browser, THE Application SHALL display a visible install button or banner in the UI that, when activated, triggers the browser's native add-to-home-screen prompt.
5. THE Application SHALL load and render the Dashboard, defined as all primary navigation elements and module list visible in the viewport, within 3 seconds when measured using the Lighthouse mobile throttling profile (simulated 4G, mid-tier CPU) with a pre-populated service worker cache.

---

### Requirement 2: Responsive Layout

**User Story:** As a user, I want the application to look and work well on both my phone and my desktop, so that I can study from any device.

#### Acceptance Criteria

1. THE Application SHALL adapt its layout and typography to the viewport width, applying distinct layout rules at the 640px, 768px, and 1024px breakpoint thresholds.
2. WHILE the viewport width is below 640px, THE Application SHALL render content in a single-column layout.
3. WHILE the viewport width is 640px or above, THE Application SHALL render the Module grid in at least a two-column layout.
4. WHILE the viewport width is 1024px or above, THE Application SHALL render the Module grid in at least a three-column layout.
5. WHILE the viewport width is below 640px, THE Application SHALL ensure all interactive touch targets measure at least 44×44 CSS pixels.
6. THE Application SHALL render all page content without horizontal scrolling at any viewport width from 320px to 1920px.

---

### Requirement 3: Module Dashboard

**User Story:** As a user, I want a home page that shows all my study modules as cards in a grid, so that I can quickly see and navigate to any module.

#### Acceptance Criteria

1. THE Dashboard SHALL display all Modules as ModuleCards arranged in a grid layout with a minimum of 1 column and a maximum of 4 columns, adjusting column count based on available viewport width.
2. WHEN no Modules exist, THE Dashboard SHALL display an empty-state message and a prominent "Create Module" call-to-action button.
3. THE ModuleCard SHALL display the Module name truncated to a maximum of 60 characters with an ellipsis, the count of ContentItems, the count of completed ContentItems, and up to 5 Tags assigned to the Module; if more than 5 Tags are assigned, the ModuleCard SHALL display the first 5 Tags and a numeric indicator showing the remaining count.
4. WHEN the user clicks a ModuleCard, THE Application SHALL navigate to that Module's detail page and update the page content without navigating away from the single-page application.
5. WHEN the user clicks the "Create Module" button, THE Dashboard SHALL display a creation form with a Module name input field and an optional Tags input field.
6. WHEN the user submits the creation form with a Module name between 1 and 100 non-whitespace characters, THE Database SHALL persist the new Module.
7. WHEN the Database persists the new Module, THE Dashboard SHALL display the new ModuleCard within 1 second without requiring a page reload.
8. IF the user submits the creation form with an empty or whitespace-only Module name, THEN THE Application SHALL display a validation error message and SHALL NOT create the Module.
9. WHEN the user dismisses the creation form without submitting, THE Dashboard SHALL close the form and preserve the Dashboard state without creating a Module.

---

### Requirement 4: Module Tagging and Classification

**User Story:** As a user, I want to assign custom tags to my study modules, so that I can classify and identify them at a glance.

#### Acceptance Criteria

1. THE Application SHALL allow the user to assign between 1 and 10 Tags to a Module during Module creation and from the Module detail page.
2. THE Application SHALL display a Tag input that suggests up to 20 previously used Tags matching the current input string, starting from the first character typed, enabling reuse of existing Tags.
3. THE Application SHALL allow the user to create a new Tag by entering a Tag name of 1 to 30 characters that is not present in the existing Tag list, and THE Application SHALL reject Tag names that exceed 30 characters with an error message indicating the character limit.
4. WHEN a Tag is assigned to a Module, THE Database SHALL persist the Tag association and THE ModuleCard SHALL reflect the updated Tags within 2 seconds.
5. WHEN the user removes a Tag from a Module, THE Database SHALL update the Tag association and THE ModuleCard SHALL reflect the removal within 2 seconds.
6. IF a Tag is removed from all Modules, THEN THE Application SHALL retain that Tag in the previously used Tags list for future suggestions.

---

### Requirement 5: Dashboard Filtering and Search

**User Story:** As a user, I want to filter modules by tags and search by name on the dashboard, so that I can quickly locate the specific module I need.

#### Acceptance Criteria

1. THE Dashboard SHALL provide a text search bar that filters the visible ModuleCards to those whose Module name contains the SearchQuery string (case-insensitive), accepting a SearchQuery of up to 200 characters.
2. THE Dashboard SHALL provide a multi-select Tag filter control that lists all Tags in use across all Modules, with each Tag label limited to 50 characters.
3. WHEN the user selects one or more Tags in the multi-select filter, THE Dashboard SHALL display only ModuleCards that have ALL selected Tags assigned (AND logic).
4. WHEN the search bar contains a SearchQuery and one or more Tags are selected, THE Dashboard SHALL apply both filters simultaneously, displaying only ModuleCards that match both the name search and the Tag filter.
5. WHEN the search bar is cleared and no Tags are selected, THE Dashboard SHALL display all ModuleCards.
6. THE Dashboard SHALL update the filtered ModuleCard list in real time as the user types in the search bar, with no more than 100ms of debounce delay.
7. IF the applied filters produce no matching ModuleCards, THEN THE Dashboard SHALL display a no-results message indicating no modules match the current filter criteria, and SHALL preserve the current filter state.
8. WHEN the user selects a Tag in the multi-select filter, THE Application SHALL visually distinguish selected Tags from unselected Tags in the filter control.
9. THE Tag filter control SHALL be collapsed by default, presenting a single compact trigger (labeled for tag filtering) rather than rendering the full list of Tag chips inline, so that the control occupies a bounded vertical area regardless of the number of available Tags.
10. WHILE the Tag filter control is collapsed, THE control SHALL display the currently selected Tag chips inline alongside the trigger, each remaining individually removable, so that active filters stay visible without expanding the control.
11. WHEN the trigger reports active filters, THE trigger SHALL display the count of currently selected Tags.
12. WHEN the user activates the trigger, THE control SHALL toggle between collapsed and expanded; WHILE expanded, THE control SHALL render the full set of available Tag chips (the Tag cloud) in-flow below the trigger, pushing subsequent content (the module grid) down only while expanded.
13. THE expand/collapse trigger SHALL be keyboard operable and SHALL expose its expanded or collapsed state to assistive technology (e.g. via aria-expanded).
14. WHEN no Tags are in use across all Modules, THE Tag filter control (trigger included) SHALL NOT be rendered.

---

### Requirement 6: Module Detail Page and Section Layout

**User Story:** As a user, I want each module to have a detail page with collapsible sections, so that I can organize study materials by topic without being overwhelmed.

#### Acceptance Criteria

1. THE Application SHALL render a dedicated detail page for each Module, accessible by navigating from the corresponding ModuleCard.
2. THE Module detail page SHALL display all Sections belonging to the Module as an accordion layout where each Section can be individually expanded or collapsed.
3. THE Module detail page SHALL provide a "Collapse All" button that collapses all Sections simultaneously and an "Expand All" button that expands all Sections simultaneously.
4. WHEN the user clicks a Section header, THE Application SHALL toggle that Section's expanded or collapsed state without affecting other Sections.
5. WHEN the user submits a Section name between 1 and 100 non-whitespace characters, THE Database SHALL persist the Section and THE Module detail page SHALL render the new Section without requiring a page reload.
6. IF the user submits a new Section with an empty, whitespace-only, or name exceeding 100 characters, THEN THE Application SHALL display a validation error and SHALL NOT create the Section.
7. IF the Database fails to persist the new Section, THEN THE Application SHALL display an error notification and preserve the entered Section name in the input field.

---

### Requirement 7: Content Item Management

**User Story:** As a user, I want to add, view, and delete individual study resources within a section, so that I can maintain a curated list of materials.

#### Acceptance Criteria

1. THE Application SHALL provide a manual input form within each Section that accepts a Page Title/Identifier (1–200 characters) and a URL (maximum 2048 characters, must begin with `http://` or `https://`), and WHEN the user submits the form with both fields populated, THE Database SHALL append the new ContentItem to that Section.
2. IF the user submits the manual input form with an empty Title, a Title exceeding 200 characters, an empty URL, or a URL that does not begin with `http://` or `https://`, THEN THE Application SHALL display a field-level validation error message indicating which field failed and why, and SHALL NOT create the ContentItem.
3. THE Application SHALL always append new ContentItems to the end of the Section's existing item list and SHALL NEVER automatically overwrite or delete existing ContentItems.
4. THE Application SHALL display each ContentItem with its Title, URL as a clickable external link that opens in a new browser tab, ProgressState, QualityMetadata (if available), and a Notes field of up to 1000 characters.
5. THE Application SHALL provide a delete control on each ContentItem that, WHEN clicked, opens a confirmation modal identifying the ContentItem by its Title and asking the user to confirm deletion.
6. WHEN the user confirms deletion in the confirmation modal, THE Database SHALL remove the ContentItem within 3 seconds and THE Section SHALL no longer display it.
7. WHEN the user cancels the confirmation modal, THE ContentItem SHALL remain in the Section unchanged and the modal SHALL be dismissed.

---

### Requirement 8: Notes and Highlights

**User Story:** As a user, I want each study resource to have an editable notes field that supports Markdown, so that I can record key takeaways alongside the resource.

#### Acceptance Criteria

1. THE Application SHALL render an editable Notes field on each ContentItem that accepts plain text or Markdown-formatted text up to 10,000 characters.
2. WHEN the user edits the Notes field and the focus leaves the field (blur event), THE Database SHALL persist the updated note content for that ContentItem within 2 seconds.
3. IF the note content exceeds 10,000 characters, THEN THE Application SHALL reject the input and display an error message indicating the character limit has been exceeded, preserving the last valid saved content.
4. IF the Database fails to persist the updated note content, THEN THE Application SHALL display an error message indicating the save failed and retain the unsaved content in the Notes field.
5. THE Application SHALL render a Markdown preview of the Notes field content when the ContentItem is in view mode, supporting at minimum: headings (H1–H3), bold, italic, inline code, code blocks, and unordered lists.
6. THE Application SHALL provide a toggle control on each ContentItem to switch between edit mode (raw text input) and view mode (rendered Markdown preview).
7. WHILE the Notes field is in edit mode, THE Application SHALL display the raw Markdown text in a textarea control with a visible character count showing the number of characters remaining out of 10,000.

---

### Requirement 9: Progress Tracking

**User Story:** As a user, I want to mark each study resource with a progress status, so that I can track what I have completed, what is in progress, and what I have not started.

#### Acceptance Criteria

1. THE Application SHALL display a ProgressState control on each ContentItem with three selectable states: "Not Started", "In Progress", and "Completed".
2. WHEN the user changes the ProgressState of a ContentItem, THE Database SHALL persist the new ProgressState within 1 second.
3. THE ModuleCard SHALL display the count of ContentItems with ProgressState "Completed" out of the total ContentItem count for that Module.
4. THE Module detail page SHALL display a per-Section progress summary showing the count of "Completed" ContentItems out of the total ContentItems in that Section.
5. WHEN all ContentItems in a Module have ProgressState "Completed", THE ModuleCard SHALL display a visual completion badge.
6. WHEN a new ContentItem is created, THE Application SHALL assign it a default ProgressState of "Not Started".
7. IF the Database fails to persist the ProgressState change, THEN THE Application SHALL display an error message and revert the ProgressState control to its previous state.

---

### Requirement 10: Offline Persistence with Dexie.js

**User Story:** As a user, I want all my data to be saved automatically in my browser so that my study materials are available every time I open the app, even offline.

#### Acceptance Criteria

1. THE Database SHALL use Dexie.js as the IndexedDB wrapper and SHALL define versioned schemas for the following stores: `modules`, `tags`, `moduleTagAssociations`, `sections`, `contentItems`, `notes`, and `progressStates`, where each store defines at minimum a primary key and all fields required to reconstruct the entity's full state.
2. WHEN the user creates, updates, or deletes any entity (Module, Tag, Section, ContentItem, Note, ProgressState), THE Database SHALL persist the change before the Application provides a success response to the user interaction, and SHALL treat the operation as successful only after the Dexie.js write call resolves without error.
3. WHEN the Application loads, THE Application SHALL read all data from THE Database and restore the full UI state, including all Modules, Sections, ContentItems, Notes, and ProgressStates, completing the restore within 3 seconds for a dataset of up to 10,000 total records.
4. IF a Dexie.js operation fails, THEN THE Application SHALL display an error notification to the user describing the failure, SHALL preserve the pre-operation state of the affected entity without partial writes, and SHALL NOT silently discard the change.
5. IF the Application load reads no existing data from THE Database, THEN THE Application SHALL render an empty initial state with no modules, sections, content items, notes, or progress states, without displaying an error notification.

---

### Requirement 11: Web Search Integration with Quality Filtering

**User Story:** As a user, I want to search for online study resources and have the results ranked by quality and engagement, so that I can quickly identify the best materials to add to my modules.

#### Acceptance Criteria

1. THE Application SHALL provide a web search interface within the Module detail page that accepts a SearchQuery of 1 to 200 characters and retrieves online study resources from a public search API.
2. WHEN search results are retrieved, THE Application SHALL sort the results in descending order by a composite quality score calculated as: rating (weighted highest), then view count (weighted second), then channel or domain relevance score (weighted lowest), with ties resolved by preserving the original retrieval order, so that higher-scoring results appear first.
3. THE Application SHALL display QualityMetadata badges alongside each search result, showing available metrics such as view count, rating, and channel or domain name.
4. IF no QualityMetadata is available for a search result, THEN THE Application SHALL display the result without badges and SHALL NOT display placeholder or fabricated metric values.
5. WHEN the user selects a search result to add, THE Application SHALL append a new ContentItem to the target Section with the result's title, URL, and any available QualityMetadata, following the append-only principle defined in Requirement 7.
6. IF the web search API call fails, THEN THE Application SHALL display an error message indicating the search could not be completed and SHALL NOT append any ContentItem automatically.
7. IF the web search API call succeeds but returns no results, THEN THE Application SHALL display an empty-state message indicating no results were found for the given SearchQuery and SHALL NOT append any ContentItem automatically.

---

### Requirement 12: Export and Import (JSON Backup)

**User Story:** As a user, I want to export all my data to a JSON file and import it back, so that I can manually back up and restore my study materials.

#### Acceptance Criteria

1. WHEN the user triggers the Export action, THE Application SHALL serialize the full application state — including all Modules, Tags, Sections, ContentItems, Notes, and ProgressStates — into a single JSON ExportFile containing a format version field, and SHALL trigger a browser file download with a filename in the format `backup-YYYY-MM-DD.json`.
2. WHEN the user triggers the Import action, THE Application SHALL accept a JSON ExportFile, validate that it conforms to the expected schema including the presence of a valid format version field, and restore the full application state from it.
3. WHEN the user initiates an Import, THE Application SHALL display a confirmation modal warning that the import will permanently replace the current data, and SHALL proceed with the restore only after the user explicitly confirms.
4. IF the user cancels the import confirmation modal, THEN THE Application SHALL dismiss the modal and SHALL NOT modify the existing application state.
5. IF the provided ExportFile fails schema validation, THEN THE Application SHALL display an error message indicating which validation rule was violated and SHALL NOT modify the existing application state.
6. THE Application SHALL guarantee that exporting the application state and then importing the resulting ExportFile produces an application state where all entities — including their fields, identifiers, relationships, and ordering — are identical to the pre-export state.

---

### Requirement 13: Google Drive Sync

**User Story:** As a user, I want to back up and restore my study data to my personal Google Drive, so that I can access it across devices without a backend server.

#### Acceptance Criteria

1. THE Application SHALL integrate the Google Drive API using the Google OAuth 2.0 client-side (implicit or PKCE) flow to authenticate the user without a backend server.
2. WHEN the user initiates a "Sync to Drive" action and is authenticated, THE Application SHALL serialize the full application state into an ExportFile and upload or overwrite it as a single file named "studydata-backup.json" in a folder named "StudyOrganizer" in the user's Google Drive, completing the upload within 30 seconds.
3. WHEN the user initiates a "Restore from Drive" action and is authenticated, THE Application SHALL download the most recent ExportFile from the "StudyOrganizer" folder in Google Drive and follow the same validation and import process defined in Requirement 12.
4. IF the Google OAuth authentication fails or the user denies permission, THEN THE Application SHALL display an error message indicating the reason (authentication failure or permission denied) and SHALL NOT attempt any Drive read or write operation.
5. IF the Google Drive API call fails during upload or download, THEN THE Application SHALL display an error message indicating whether the failure occurred during upload or download, and SHALL NOT modify the existing Database.
6. WHILE the user is not authenticated with Google, THE Application SHALL display a "Connect Google Drive" button and SHALL NOT expose Sync or Restore controls.
7. IF the "StudyOrganizer" folder does not exist in the user's Google Drive when a "Sync to Drive" action is initiated, THEN THE Application SHALL create the folder before uploading the ExportFile.
8. WHILE a Drive upload or download operation is in progress, THE Application SHALL display a visible progress indicator and SHALL disable the Sync and Restore controls until the operation completes or fails.

---

### Requirement 14: Search-Driven Module Generation

**User Story:** As a user, I want to search the internet from the Dashboard and have the Application generate a Module whose Sections are organized by content type, so that I can build a study module from search results instead of only finding items one at a time.

#### Acceptance Criteria

1. THE Dashboard SHALL provide a "Discover" search interface, visually distinct from the name/tag filter defined in Requirement 5, that accepts a SearchQuery of 1 to 200 characters.
2. WHEN the user submits a Discover SearchQuery, THE Application SHALL query one or more configured search sources (at minimum: a video source and a web source) and retrieve online study resources.
3. THE Application SHALL classify each retrieved result into exactly one of four content types — Video, Article, Course, or Book — using deterministic, inspectable rules based on the result's source and URL/domain, defaulting unmatched web results to Article.
4. WHEN results are classified, THE Application SHALL present a preview of a proposed Module containing one Section per content type that has at least one result, with each Section's results ordered by the composite quality score defined in Requirement 11.2, and SHALL NOT persist any entity to the Database at this stage.
5. THE preview SHALL display an editable proposed Module name that defaults to the submitted SearchQuery, constrained to 1 to 100 non-whitespace characters.
6. WHEN the user confirms the preview via a "Save module" action, THE Database SHALL persist, in a single atomic transaction, the new Module, one Section per non-empty content type, and all previewed results as ContentItems (including any available QualityMetadata), following the append-only principle of Requirement 7.
7. WHEN the Database persists the generated Module, THE Dashboard SHALL display the new ModuleCard within 1 second without requiring a page reload.
8. WHEN the user dismisses or cancels the preview without confirming, THE Application SHALL discard the proposed Module and SHALL NOT modify the Database.
9. IF every configured search source fails, THEN THE Application SHALL display an error message indicating the search could not be completed and SHALL NOT present a preview.
10. IF the search sources succeed but return no results, THEN THE Application SHALL display an empty-state message indicating no results were found and SHALL NOT present a preview.
11. WHERE a search source requires credentials (e.g., an API key or search-engine identifier), THE Application SHALL accept those credentials via the UI, persist them only in the browser's local storage, and SHALL NOT hardcode or transmit them to any endpoint other than the corresponding search provider.
12. THE Application SHALL retain the existing manual Module creation (Requirement 3), manual Section creation (Requirement 6), and per-Section web search (Requirement 11) flows unchanged and available alongside the Discover generation flow.

---

### Requirement 15: Dedicated Settings Page

**User Story:** As a user, I want a separate Settings page that groups API keys, backup/restore, and Google Drive sync, so that these advanced controls are out of the way of my day-to-day study flow.

#### Acceptance Criteria

1. THE Application SHALL provide a dedicated Settings page at its own route, reachable from a visible control on the Dashboard, and SHALL return the user to the Dashboard via a back control.
2. THE Settings page SHALL present three distinct sections: API Keys, Export/Import, and Google Drive.
3. THE API Keys section SHALL provide inputs for the YouTube Data API key, the Google Custom Search API key, and the Google Custom Search engine identifier (cx), persisting each only in the browser's local storage and never transmitting them to any endpoint other than the corresponding Google API.
4. THE API Keys section SHALL indicate, for each credential, whether it is currently set, and SHALL allow the user to update or clear each credential.
5. WHEN a Discover search (Requirement 14) runs, THE Application SHALL read the credentials from the values persisted by the Settings page.
6. THE Application SHALL move the Export/Import and Google Drive controls out of the Module detail page and into the Settings page, and SHALL NOT duplicate those controls elsewhere.

---

### Requirement 16: Selective Export, Import, and Drive Backup

**User Story:** As a user, I want to choose which modules are included when I export, import, or sync, so that I can back up or move only the modules I care about without affecting the rest.

#### Acceptance Criteria

1. WHEN the user initiates an Export, THE Application SHALL present a selectable list of all existing Modules and SHALL include in the ExportFile only the Modules the user selects, together with their associated Sections, ContentItems, Notes, ProgressStates, and the Tags referenced by those Modules.
2. THE Application SHALL provide select-all and select-none controls for the Module selection list, and SHALL default to all Modules selected.
3. WHEN the user initiates an Import, THE Application SHALL parse and validate the ExportFile per Requirement 12.2, SHALL present the Modules contained in the file as a selectable list, and SHALL import only the Modules the user selects.
4. THE Application SHALL perform every Import in merge mode: imported Modules SHALL be added to the existing data without deleting or overwriting any existing Module, Section, ContentItem, Note, ProgressState, or Tag.
5. WHEN importing in merge mode, THE Application SHALL assign new identifiers to all imported entities and SHALL remap every parent-child relationship (Module to Section, Section to ContentItem, ContentItem to Note and ProgressState, Module to Tag) so that no imported identifier collides with an existing identifier and no relationship is broken.
6. WHEN importing a Tag whose name already exists in the current data, THE Application SHALL reuse the existing Tag rather than creating a duplicate, and SHALL associate the imported Modules with that existing Tag.
7. WHEN the user initiates a Google Drive backup, THE Application SHALL apply the same per-Module selection defined for Export, uploading only the selected Modules and their associated data.
8. WHEN the user initiates a Google Drive restore, THE Application SHALL download the backup, present its Modules as a selectable list, and import the selected Modules in merge mode per criteria 4 through 6.
9. IF the user selects no Modules for an Export or Drive backup, THEN THE Application SHALL disable the confirm action and SHALL display a message indicating at least one Module must be selected.
10. THE Application SHALL guarantee that exporting a selected set of Modules and then importing that ExportFile into an empty application reproduces those Modules, Sections, ContentItems, Notes, ProgressStates, and Tag associations identically, preserving ordering.

---

### Requirement 17: Editing Existing Content

**User Story:** As a user, I want to edit a content item or rename a section after creating it, so that I can fix mistakes and keep my materials accurate.

#### Acceptance Criteria

1. THE Application SHALL provide an edit control on each ContentItem that reveals an inline editor pre-filled with the item's current Title and URL (and quality rating when present).
2. WHEN the user saves an edited ContentItem with a Title of 1-200 characters and a URL that begins with http:// or https:// and is at most 2048 characters, THE Database SHALL persist the updated fields and THE Section SHALL display the updated values without a page reload.
3. IF the user saves an edited ContentItem with an empty Title, a Title over 200 characters, an empty URL, or a URL not beginning with http:// or https://, THEN THE Application SHALL display a field-level validation error and SHALL NOT persist the change.
4. WHEN the user cancels an in-progress ContentItem edit, THE Application SHALL discard the edits and restore the previously saved values.
5. THE Application SHALL provide an edit control on each Section header that lets the user rename the Section inline; WHEN the user saves a Section name of 1-100 non-whitespace characters, THE Database SHALL persist the new name and the UI SHALL reflect it without a page reload; an empty, whitespace-only, or over-100-character name SHALL be rejected with a validation error.
6. IF the Database fails to persist an edit, THEN THE Application SHALL display an error notification and SHALL retain the user's entered values in the editor.

---

### Requirement 18: Deleting Content, Sections, and Modules

**User Story:** As a user, I want to delete a content item, a section, or an entire module, so that I can remove materials I no longer need.

#### Acceptance Criteria

1. THE Application SHALL provide a delete control on each ContentItem, each Section, and each Module.
2. WHEN the user activates any delete control, THE Application SHALL display a confirmation dialog that names the target entity and states what will be removed, and SHALL perform the deletion only after the user explicitly confirms.
3. WHEN the user cancels the confirmation dialog, THE Application SHALL dismiss the dialog and SHALL NOT modify the Database.
4. WHEN the user confirms deletion of a ContentItem, THE Database SHALL remove that ContentItem together with its Note and ProgressState in a single atomic transaction, and the Section SHALL no longer display it.
5. WHEN the user confirms deletion of a Section, THE Database SHALL remove that Section and all of its ContentItems together with their Notes and ProgressStates in a single atomic transaction, and the Module detail page SHALL no longer display the Section.
6. WHEN the user confirms deletion of a Module, THE Database SHALL remove that Module and all of its Sections, ContentItems, Notes, ProgressStates, and its ModuleTagAssociations in a single atomic transaction, SHALL preserve the Tag entities themselves, and THE Application SHALL navigate back to the Dashboard where the Module no longer appears.
7. IF a delete transaction fails, THEN THE Application SHALL display an error notification and SHALL leave the targeted entity and all of its descendants unchanged (no partial deletion).

---

### Requirement 19: Selective Save of Discover Results

**User Story:** As a user, I want to choose which search results are included before saving a generated module, so that I keep only the results I actually want.

#### Acceptance Criteria

1. THE generated module preview (Requirement 14) SHALL render a selection control on every result, with all results selected by default.
2. THE preview SHALL provide a per-Section selection control that selects or deselects all results within that content-type Section at once.
3. WHEN the user confirms Save, THE Application SHALL persist only the selected results, SHALL create a Section only for a content type that has at least one selected result, and SHALL skip entirely any content type with no selected results.
4. WHILE no results are selected across the entire preview, THE Application SHALL disable the Save action and SHALL display a message indicating at least one result must be selected.
5. THE Application SHALL preserve, within each saved Section, the composite-score ordering (Requirement 11.2) of the selected results.

---

### Requirement 20: Content Item Thumbnails

**User Story:** As a user, I want each content item to show a small thumbnail, so that I can scan and recognize my materials visually.

#### Acceptance Criteria

1. THE Application SHALL display a thumbnail to the left of the title for every ContentItem, both in a Section's item list and in the Discover generated-module preview.
2. WHERE a ContentItem URL is a YouTube video, THE Application SHALL derive the video thumbnail image URL from the video id and display it, without requiring any API key.
3. WHERE a ContentItem URL is not a YouTube video, THE Application SHALL display the favicon of the URL's domain as the thumbnail.
4. IF the thumbnail image fails to load, or the URL cannot be parsed, THEN THE Application SHALL display a generic fallback icon chosen by the item's content type (video, article, course, or book) and SHALL NOT show a broken image.
5. THE Application SHALL derive every thumbnail purely from the ContentItem URL at render time and SHALL NOT persist any additional thumbnail data in the Database.
6. THE thumbnail SHALL be decorative: it SHALL carry empty alternative text (or be hidden from assistive technology) so it does not duplicate the title for screen-reader users, and the title SHALL remain the primary accessible label and link.
7. THE thumbnail SHALL render at a fixed small size (approximately 40 to 48 CSS pixels square) with rounded corners and SHALL NOT cause horizontal overflow at any supported viewport width.

---

### Requirement 21: Global Navigation Header

**User Story:** As a user, I want a persistent navigation header on every page, so that I can jump home, create a module, open settings, or search my modules from anywhere without first returning to the Dashboard.

#### Acceptance Criteria

1. THE Application SHALL display a single, consistent navigation header at the top of every page (Dashboard, Module detail, and Settings).
2. THE navigation header SHALL provide a Home control that navigates to the Dashboard, a Create-module control, a Settings control that navigates to the Settings page, and a module-search input.
3. WHEN the user is already on the target page of a navigation control, THE control SHALL remain functional and SHALL NOT produce an error (e.g. activating Home while on the Dashboard is a no-op navigation).
4. WHEN the user submits or changes the module-search input from any page, THE Application SHALL navigate to the Dashboard and apply the entered text as the module name filter so the matching ModuleCards are shown.
5. THE Application SHALL carry the module-search term in the URL as a query parameter so that the filtered Dashboard view survives navigation and can be bookmarked or shared; WHEN the parameter is present on load, THE Dashboard SHALL initialize its name filter from it.
6. WHEN the user activates the Create-module control from a page other than the Dashboard, THE Application SHALL navigate to the Dashboard and open the create-module interface.
7. THE module-name search in the header SHALL preserve the existing name-filter semantics defined in Requirement 5 (case-insensitive containment, up to 200 characters) and SHALL combine with the Dashboard tag filter unchanged.
8. THE navigation header SHALL be responsive and SHALL NOT cause horizontal overflow at any supported viewport width; secondary labels MAY collapse to icons on narrow viewports while every control remains reachable and keyboard operable.
9. THE collapsible tag filter (Requirement 5.9–5.14) SHALL remain in the Dashboard body and SHALL NOT be moved into the global navigation header.

---

### Requirement 22: Multi-Source Search with Trust-Aware Ranking

**User Story:** As a user, I want every search to pull study material of all kinds — videos, articles, blogs, courses, and books — from several trustworthy sources at once, ranked by community rating and source/author authority, so that I can rely on the top results regardless of media type.

#### Acceptance Criteria

1. THE Application SHALL query, for a single SearchQuery, multiple content sources spanning at least: videos (YouTube), community Q&A/articles (Stack Exchange), articles/blogs (Dev.to), reference articles (Wikipedia), and books (Open Library).
2. THE Application SHALL run every configured source that requires no user credentials by default, without the user having to configure anything, so that a search returns a mix of media types out of the box.
3. WHERE a source requires credentials that are not configured (e.g. YouTube, or the optional Google Custom Search), THE Application SHALL skip only that source, SHALL continue with the remaining sources, and SHALL surface a non-blocking notice naming the skipped source.
4. THE Application SHALL normalize every result, regardless of source, into the common result shape and SHALL classify it into exactly one content type (Video, Article, Course, or Book) using the deterministic rules of Requirement 14.3, extended so that book sources classify as Book and course platforms classify as Course.
5. THE composite quality score SHALL incorporate, when a source provides them: a community rating/vote signal (e.g. Stack Exchange score, Dev.to reactions), a popularity signal (e.g. YouTube view count, Wikipedia pageviews, Open Library edition/readership counts), and a source/author authority signal.
6. THE Application SHALL assign each source a baseline authority weight (an inspectable, source-level trust value) and SHALL incorporate an author-reputation signal where the source exposes one (e.g. Stack Exchange answerer reputation), combining these into the authority component of the composite score.
7. THE Application SHALL rank the combined results across all sources by the composite score in descending order, with deterministic tie-breaking, so that higher community-rated and higher-authority items appear first regardless of which source produced them.
8. THE Application SHALL display, for each result, a transparency indicator naming the source and the concrete signals behind its ranking (e.g. "Stack Overflow · 1.2k votes", "Wikipedia · reference", "YouTube · 340K views"), and SHALL NOT fabricate or show placeholder values for signals a result does not have.
9. IF every queried source fails, THEN THE Application SHALL display an error indicating the search could not be completed and SHALL NOT present results; IF sources succeed but collectively return no results, THEN THE Application SHALL display an empty-state message.
10. IF an individual source fails or times out while at least one other source succeeds, THEN THE Application SHALL present the successful results and SHALL NOT fail the whole search.
11. THE multi-source search SHALL be available from every search surface in the Application — both the Dashboard Discover flow (Requirement 14) and the per-Section search (Requirement 11) — so that any search can return all media types.
12. THE Application SHALL keep the optional Google Custom Search source available and configurable via Settings (Requirement 15), so that the user may enable broad web results in addition to the built-in sources.
13. THE authority weights and scoring signal mapping SHALL be defined in inspectable application code (not opaque or hidden), so that the ranking rationale can be reviewed and adjusted.

---

### Requirement 23: Three-Column Dashboard with Pinning and Reordering

**User Story:** As a user, I want my home page organized into three columns — all my modules on the left, Discover in the middle, and the modules I have started on the right — each list scrollable with the ability to pin and reorder, so that I can keep the modules I care about within reach while discovering new material.

#### Acceptance Criteria

1. THE Dashboard SHALL present three regions on wide viewports: a left column listing all Modules, a center column containing the Discover interface (Requirement 14), and a right column listing only Modules the user has started.
2. A Module SHALL be considered started WHEN at least one of its ContentItems has a ProgressState of In Progress or Completed; THE right column SHALL list exactly the started Modules and SHALL update within 1 second of a progress change without a page reload.
3. A started Module SHALL also remain listed in the left column (all Modules); the same Module MAY appear in both the left and right columns simultaneously.
4. THE left and right columns SHALL each be independently vertically scrollable when their content exceeds the available height, without causing the page or the center column to scroll horizontally.
5. THE user SHALL be able to pin and unpin a Module within a column; WHILE a Module is pinned in a column, THE column SHALL order it ahead of all unpinned Modules in that column.
6. THE user SHALL be able to reorder Modules within a column by dragging a Module card to a new position; THE resulting order SHALL be persisted and SHALL survive a page reload.
7. THE pin state and the manual order SHALL be tracked independently per column, so that pinning or reordering a Module in one column SHALL NOT change its pin state or order in the other column.
8. WITHIN a column, THE display order SHALL be: pinned Modules first (in their manual order), then unpinned Modules (in their manual order); a Module with no manual order yet SHALL fall back to a deterministic default (e.g. most-recently-updated or creation order).
9. THE Application SHALL persist pin state and per-column order in the local Database so the arrangement is restored on next load, and SHALL NOT require any backend or network for this persistence.
10. WHEN a Module is deleted (Requirement 18), THE Application SHALL remove its dashboard pin/order entries so no orphaned arrangement data remains.
11. ON narrow viewports THE three columns SHALL stack vertically in the order Discover, then all Modules, then started Modules, and every column SHALL remain reachable and usable (pin, reorder, and scroll) without horizontal overflow.
12. THE existing name/tag filtering (Requirement 5) SHALL continue to apply to the left column's list of all Modules; the global navigation header (Requirement 21) SHALL remain present above the three columns.
13. THE drag-and-drop reordering SHALL be operable by keyboard as well as pointer, or SHALL provide an equivalent keyboard-accessible means of moving a Module within its column, so the feature meets accessibility expectations.

---

### Requirement 24: Dark Mode

**User Story:** As a user, I want to switch the application between a light and a dark theme from a button in the global header, so that I can study comfortably in low-light conditions.

#### Acceptance Criteria

1. THE global navigation header (Requirement 21) SHALL provide a theme toggle control positioned immediately to the right of the Settings control.
2. THE theme toggle SHALL switch the Application between exactly two themes, light and dark, as a simple on/off control (no third option).
3. WHEN the user activates the theme toggle, THE Application SHALL apply the newly selected theme immediately across the currently visible UI without requiring a page reload.
4. THE Application SHALL persist the user's selected theme in the browser's local storage and SHALL restore it on subsequent loads so the choice survives reloads and new sessions.
5. WHERE the user has not yet made a manual choice, THE Application SHALL initialize the theme from the operating system / browser dark-mode preference (prefers-color-scheme), defaulting to light when no preference is detectable.
6. THE theme toggle control SHALL expose its current state to assistive technology (e.g. an accessible label and/or aria-pressed) and SHALL be keyboard operable with a visible focus indicator and a minimum 44x44px target.
7. WHILE the dark theme is active, THE core surfaces — the global header, the Dashboard background and its three columns, and the ModuleCards — SHALL render with dark-appropriate background, text, and border colors that meet WCAG AA contrast for normal text.
8. THE theme SHALL be applied via a single global mechanism (a theme class/attribute on the document root) so that themed components derive their colors from it rather than each toggling independently.
9. THE Application SHALL avoid a flash of the wrong theme on initial load by applying the stored/preferred theme before first paint where feasible.
10. Surfaces not yet restyled for dark mode SHALL remain fully functional and legible (no invisible text) while the dark theme is active, even if their visual polish is completed incrementally.
