# Requirements Document

## Introduction

The application already supports a user-selectable dark/light theme on its core surfaces (global header, dashboard, module columns, module cards, and the discover/search panel). The theme is driven by an existing, reusable infrastructure: a theme module (`getInitialTheme`/`applyTheme`), a `ThemeProvider`/`useTheme` context, a `ThemeToggle` button in the global header, and Tailwind's `class` dark-mode strategy that toggles a `dark` class on the document root. An inline anti-flash script sets the correct class before React mounts.

However, several pages and their components were built without `dark:` styling variants. When a user who has selected the dark theme opens the Module Detail page or the Settings page, those surfaces and the components they render remain in light styling, producing an inconsistent and, in places, low-contrast experience.

This feature closes that gap. It is an incremental styling effort that reuses the existing theme infrastructure without introducing any new theme mechanism, toggle, or persistence path. The goal is that every page and every visible or interactive component in the application honors the currently active theme, with readable contrast and visible focus states in both themes. The scope is the Module Detail page, the Settings page, and all of the components they render, plus remaining shared components (dialogs, modals, badges, form controls, the PWA install banner) that are not yet theme-aware, so that the whole application is consistent.

## Glossary

- **Application**: The study-materials-organizer React single-page application.
- **Theme**: The active color scheme, one of `light` or `dark`, resolved and persisted by the existing theme infrastructure.
- **Active_Theme**: The theme currently applied to the Application, reflected by the presence or absence of the `dark` class on the document root element.
- **Theme_Infrastructure**: The existing, pre-built set of theme utilities that resolve, apply, persist, and expose the Active_Theme — specifically the theme module (`getInitialTheme`, `applyTheme`), the `ThemeProvider`/`useTheme` context, the `ThemeToggle` control, the Tailwind `class` dark-mode configuration, and the inline anti-flash script in `index.html`.
- **Theme_Toggle**: The existing single control in the global header that switches the Active_Theme between `light` and `dark`.
- **Dark_Variant**: A Tailwind utility class prefixed with `dark:` that applies only while the `dark` class is present on the document root.
- **Covered_Surface**: A page or component that renders correct styling for both the `light` and `dark` values of the Active_Theme.
- **Uncovered_Surface**: A page or component that, at the start of this feature, lacks Dark_Variant styling and therefore renders light styling regardless of the Active_Theme. The set of Uncovered_Surfaces is enumerated in Requirement 2.
- **Module_Detail_Page**: The page at `src/pages/ModuleDetailPage.tsx`, including its loading skeleton, not-found state, inline error alerts, collapse/expand controls, and empty-state text.
- **Settings_Page**: The page at `src/pages/SettingsPage.tsx`, including its outer container, page heading, and the three section cards.
- **Detail_Component**: Any component rendered by the Module_Detail_Page, including `ModuleHeader`, `SectionAccordion`, `AddSectionForm`, `SectionContent`, and the components those render (such as content item rows, note editors, and tag editors).
- **Settings_Component**: Any component rendered by the Settings_Page, specifically `ApiKeysSettings`, `ExportImportSettings`, and `DriveSettings`, and the components those render.
- **Shared_Component**: A theme-agnostic reusable component rendered across multiple pages that is not yet theme-aware, specifically `ConfirmDialog`, `CreateModuleModal`, `GeneratedModulePreview`, and `PwaInstallBanner`.
- **UI_Element_Category**: A recurring visual element type that must receive consistent dark styling — specifically text, surfaces/cards, borders, form inputs, buttons, badges/pills, loading skeletons, error/alert boxes, and empty-state messages.
- **Contrast_Standard**: The WCAG 2.1 Level AA minimum contrast ratio — 4.5:1 for normal-size text and 3:1 for large text (at least 18pt, or 14pt bold) and for meaningful user-interface component boundaries and states.
- **Focus_Indicator**: The visible focus styling (such as a focus ring) shown on an interactive element when it receives keyboard focus.

## Requirements

### Requirement 1: Reuse of the existing theme infrastructure

**User Story:** As a developer maintaining the Application, I want the full-coverage dark mode to reuse the existing theme infrastructure, so that there is a single source of truth for the Active_Theme and no duplicate or conflicting theme logic is introduced.

#### Acceptance Criteria

1. THE Application SHALL derive the Active_Theme for all Covered_Surfaces solely from the presence or absence of the `dark` class on the document root element managed by the Theme_Infrastructure, and SHALL NOT read theme state from any other source.
2. THE Application SHALL resolve all theme-specific styling on Covered_Surfaces through Dark_Variant utility classes that respond to the `dark` class, and SHALL NOT determine theme-specific styling by reading the Active_Theme in JavaScript to conditionally emit class names or inline styles.
3. THE Application SHALL retain exactly one Theme_Toggle, located in the global header, and SHALL NOT present any additional theme-switching control on any Covered_Surface.
4. WHERE a Covered_Surface requires theme-aware styling, THE Application SHALL style that surface using only the Theme_Infrastructure, and SHALL NOT introduce any additional theme state, theme persistence key, or theme application mechanism beyond the Theme_Infrastructure.

### Requirement 2: Enumerated coverage of all previously uncovered surfaces

**User Story:** As a user who prefers the dark theme, I want every page and component that was previously light-only to honor my selected theme, so that no part of the Application appears inconsistent.

#### Acceptance Criteria

1. WHILE the Active_Theme is `dark`, THE Module_Detail_Page SHALL apply Dark_Variant styling to each of its outer container, loading skeleton, not-found state, collapse/expand controls, empty-section message, and inline delete-error alert, such that every UI_Element_Category instance within them renders with the established dark palette at a contrast ratio meeting the Contrast_Standard.
2. WHILE the Active_Theme is `dark`, THE Settings_Page SHALL apply Dark_Variant styling to its outer container, its page heading, and each of its three section cards including each card heading, such that every UI_Element_Category instance within them renders with the established dark palette at a contrast ratio meeting the Contrast_Standard.
3. WHILE the Active_Theme is `dark`, THE Application SHALL apply Dark_Variant styling to every Detail_Component enumerated in the Glossary, such that every UI_Element_Category instance each Detail_Component renders displays the established dark palette at a contrast ratio meeting the Contrast_Standard.
4. WHILE the Active_Theme is `dark`, THE Application SHALL apply Dark_Variant styling to every Settings_Component enumerated in the Glossary, such that every UI_Element_Category instance each Settings_Component renders displays the established dark palette at a contrast ratio meeting the Contrast_Standard.
5. WHILE the Active_Theme is `dark`, THE Application SHALL apply Dark_Variant styling to every Shared_Component enumerated in the Glossary, such that every UI_Element_Category instance each Shared_Component renders displays the established dark palette at a contrast ratio meeting the Contrast_Standard.
6. WHILE the Active_Theme is `light`, THE Application SHALL render the surfaces listed in acceptance criteria 1 through 5 with styling visually identical to their light appearance prior to this feature, with no Dark_Variant styling applied.

### Requirement 3: Consistent styling across UI element categories

**User Story:** As a user, I want text, cards, inputs, buttons, badges, skeletons, alerts, and empty states to follow a consistent dark palette, so that the dark theme feels cohesive rather than piecemeal.

#### Acceptance Criteria

1. WHILE the Active_Theme is `dark`, THE Application SHALL apply the established dark palette to every UI_Element_Category instance on all Covered_Surfaces, such that no instance of any UI_Element_Category retains light-theme styling.
2. WHILE the Active_Theme is `dark`, THE Application SHALL render every instance of a given UI_Element_Category that shares the same semantic role across different Covered_Surfaces with identical background, text, and border treatments from the established dark palette.
3. WHILE the Active_Theme is `dark`, THE Application SHALL render form inputs on Covered_Surfaces with a dark input surface from the established dark palette, an input-text color meeting the Contrast_Standard against that surface, and a placeholder color meeting the Contrast_Standard against that surface.
4. WHILE the Active_Theme is `dark`, THE Application SHALL render error and alert boxes on Covered_Surfaces with background, border, and text colors from the established dark palette, each meeting the Contrast_Standard against its adjacent background, and with an accent color distinct from non-error surfaces on the same page.
5. WHILE the Active_Theme is `dark`, THE Application SHALL render loading skeletons on Covered_Surfaces with placeholder surfaces from the established dark palette that are visually distinguishable from their container background at a contrast ratio of at least 1.5:1.
6. WHILE the Active_Theme is `dark`, THE Application SHALL render buttons on Covered_Surfaces with background, text, and border treatments from the established dark palette, each meeting the Contrast_Standard against its adjacent background.
7. WHILE the Active_Theme is `dark`, THE Application SHALL render badges and pills on Covered_Surfaces with background and foreground colors from the established dark palette at a foreground-to-background contrast ratio meeting the Contrast_Standard.

### Requirement 4: Readable contrast in dark mode

**User Story:** As a user with limited vision, I want dark-mode surfaces to meet accessibility contrast guidelines, so that I can read text and distinguish controls comfortably.

#### Acceptance Criteria

1. WHILE the Active_Theme is `dark`, THE Application SHALL render normal-size text on Covered_Surfaces against its background at a contrast ratio of at least 4.5:1, and large text (at least 18pt, or 14pt bold) at a contrast ratio of at least 3:1, as defined by the Contrast_Standard.
2. WHILE the Active_Theme is `dark`, THE Application SHALL render the boundary and the default, hover, focus, and active state styling of each enabled interactive control on Covered_Surfaces at a contrast ratio of at least 3:1 against its adjacent background, as defined by the Contrast_Standard, excluding controls that are disabled or inactive.
3. WHILE the Active_Theme is `dark`, THE Application SHALL render badges, pills, and status indicators on Covered_Surfaces such that their normal-size foreground text meets a contrast ratio of at least 4.5:1, their large foreground text meets at least 3:1, and the indicator boundary that conveys state meets at least 3:1 against the adjacent background, as defined by the Contrast_Standard.

### Requirement 5: Visible focus states in both themes

**User Story:** As a keyboard user, I want focus indicators to stay visible after dark styling is applied, so that I can always tell which control is focused.

#### Acceptance Criteria

1. WHILE the Active_Theme is `dark`, WHEN an interactive control on a Covered_Surface receives keyboard focus, THE Application SHALL display a Focus_Indicator on that control.
2. WHILE the Active_Theme is `light`, WHEN an interactive control on a Covered_Surface receives keyboard focus, THE Application SHALL display the previously established Focus_Indicator on that control.
3. WHILE the Active_Theme is `dark`, THE Application SHALL render each Focus_Indicator on a Covered_Surface at a contrast ratio of at least 3:1 against the background directly adjacent to that Focus_Indicator, meeting the Contrast_Standard for user-interface components.
4. WHILE the Active_Theme is `dark`, THE Application SHALL preserve the Focus_Indicator on every interactive control on a Covered_Surface such that the applied dark styling does not remove, hide, or render the Focus_Indicator visually indistinguishable from its adjacent background.

### Requirement 6: No flash of the incorrect theme

**User Story:** As a user who selected the dark theme, I want pages to appear in the correct theme from first paint, so that I never see a flash of light styling when opening or reloading a page.

#### Acceptance Criteria

1. WHEN a Covered_Surface is first rendered after the Application loads or is reloaded, THE Application SHALL display that surface in the Active_Theme resolved by the Theme_Infrastructure in the first painted frame, without any preceding painted frame in the opposite theme.
2. WHEN the user navigates to the Module_Detail_Page or the Settings_Page, THE Application SHALL display the destination page in the Active_Theme in the first painted frame of that page, without any preceding painted frame in the opposite theme.
3. WHEN the user activates the Theme_Toggle, THE Application SHALL update every currently rendered Covered_Surface to the newly Active_Theme within the first painted frame after the toggle is activated.
4. WHILE the Theme_Toggle is being activated, THE Application SHALL NOT display any painted frame in which one Covered_Surface shows the newly Active_Theme while another currently rendered Covered_Surface shows the opposite theme.
