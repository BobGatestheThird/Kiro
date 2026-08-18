# Implementation Plan: Asset Inventory System

## Overview

Build a self-contained HTML/JavaScript web application providing CMDB-style asset inventory management. The system runs entirely client-side as a single HTML file with embedded JS/CSS, uses browser localStorage for persistence, and supports full CRUD operations, CSV bulk import/export with column mapping, multi-column filtering, and table/card rendering with pagination.

## Tasks

- [ ] 1. Set up project structure and core data layer
  - [ ] 1.1 Create the single HTML file with base structure, embedded CSS reset, and module scaffolding
    - Create `asset-inventory.html` with HTML5 doctype, viewport meta, embedded `<style>` block with CSS variables for theming, and empty `<script>` block
    - Define the application shell layout: header, toolbar area, main content area, and modal container
    - _Requirements: 13.1, 13.2, 13.3, 13.4_

  - [ ] 1.2 Implement data model constants and utility functions
    - Define `ENVIRONMENTS`, `STATUSES`, `ASSET_TYPES` enum arrays
    - Define `COLUMN_ALIASES` mapping object with all known aliases per field
    - Implement `generateId()` UUID v4 generator
    - Implement `isValidIPv4(ip)` validation function
    - _Requirements: 1.1, 1.2, 2.1, 2.2, 2.3, 2.4, 2.5_

  - [ ] 1.3 Implement the DataStore module
    - Implement `getAll()`, `getById(id)`, `save(asset)`, `delete(id)`, `bulkImport(assets)`, `exportAll()`, `clear()`, `getStats()`
    - Serialize/deserialize assets to/from localStorage under a single key `inventory_assets`
    - Handle localStorage quota exceeded errors with user-facing message
    - Handle corrupted JSON gracefully (treat as empty, warn user)
    - Enforce unique IDs and unique hostnames (case-insensitive) as invariants
    - Auto-generate `id`, `createdAt`, `updatedAt` on new asset creation
    - On update, refresh `updatedAt` while preserving `createdAt`
    - _Requirements: 9.1, 9.2, 9.3, 9.4, 9.5, 9.6, 1.3, 1.4, 1.5_

  - [ ] 1.4 Implement the FieldValidator module
    - Implement `validate(asset)` returning `{ valid, errors[] }` with all errors collected (not stopping at first)
    - Validate required fields: hostname, environment, status, assetType
    - Validate hostname format: 1-63 chars, alphanumeric + hyphens, no leading/trailing hyphen
    - Validate IPv4 format: four octets 0-255, no leading zeros
    - Validate enum fields against allowed values (case-insensitive comparison)
    - Validate numeric fields: cpuCount (positive integer 1-9999), memoryGB/storageGB (positive 0.01-999999.99, max 2 decimal places)
    - Validate free-text field max length (255 chars)
    - Implement HTML tag stripping (`<[^>]*>` and remaining `<`, `>`) for input sanitization
    - _Requirements: 2.1, 2.2, 2.3, 2.4, 2.5, 2.6, 2.7, 2.8, 2.9, 2.10, 2.11, 12.2, 12.3_

- [ ] 2. Checkpoint - Ensure data layer is solid
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 3. Implement CSV parsing and column mapping
  - [ ] 3.1 Implement the CsvParser module
    - Implement `parse(csvText)` handling RFC 4180: quoted fields, embedded commas, embedded newlines
    - Implement `detectDelimiter(csvText)` auto-detecting comma, semicolon, or tab by occurrence count in first row
    - Implement `stringify(assets, fields)` generating RFC 4180 compliant CSV with CRLF endings
    - Handle both CRLF and LF line endings producing identical output
    - Return error with line number for malformed CSV (unclosed quotes, short rows)
    - Return valid result with empty rows array for header-only or empty files
    - Implement formula injection protection: prefix values starting with `=`, `+`, `-`, `@` with single quote on export
    - _Requirements: 3.1, 3.2, 3.3, 3.4, 3.5, 3.6, 3.7, 10.1, 10.2_

  - [ ] 3.2 Implement the ColumnMapper module
    - Implement `mapHeaders(csvHeaders)` with case-insensitive exact match against alias table, then normalized fuzzy match (ignoring whitespace, underscores, hyphens)
    - Include all aliases from Requirements 4.2 (RVTools, AD export, vCenter conventions)
    - First-match priority: when multiple headers map to same field, use leftmost column
    - Report unmapped headers for user review
    - Error when zero columns are mappable
    - Return structured result: mapped pairs, unmapped headers, count of mapped columns
    - _Requirements: 4.1, 4.2, 4.3, 4.4, 4.5, 4.6_

  - [ ] 3.3 Implement the CSV import pipeline (importCsv function)
    - Orchestrate: parse → map → validate → sanitize → bulk persist (upsert by hostname)
    - Process rows in file order; last occurrence wins for duplicate hostnames within same file
    - Strip HTML tags from all field values before storage
    - Strip formula injection prefix (leading single quote) on re-import for round-trip integrity
    - Return summary: created count, updated count, skipped count, error details per row
    - _Requirements: 5.1, 5.2, 5.3, 5.4, 5.5, 5.6, 5.7, 11.3, 11.4, 12.2_

  - [ ]* 3.4 Write property tests for CSV round-trip integrity
    - **Property 1: CSV Round-Trip Integrity** — For any valid asset set, export then import produces equivalent field values
    - **Validates: Requirements 3.1, 3.5, 11.1, 11.2**

  - [ ]* 3.5 Write property tests for import idempotency and upsert
    - **Property 2: Import Idempotency** — Importing same CSV twice yields same asset count
    - **Property 3: Upsert Identity Preservation** — Upsert preserves original id and createdAt
    - **Validates: Requirements 5.6, 5.3, 1.5**

  - [ ]* 3.6 Write property tests for column mapping
    - **Property 20: Column Mapping Case Insensitivity** — Any known alias in any case maps correctly
    - **Property 21: Column Mapping First-Match Priority** — Duplicate aliases map only first occurrence
    - **Validates: Requirements 4.1, 4.3**

- [ ] 4. Checkpoint - Ensure CSV pipeline works end-to-end
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 5. Implement validation property tests
  - [ ]* 5.1 Write property tests for hostname validation
    - **Property 5: Hostname Validation** — Accepts iff 1-63 chars, alphanumeric + hyphens, no leading/trailing hyphen
    - **Validates: Requirement 2.1**

  - [ ]* 5.2 Write property tests for IPv4 validation
    - **Property 6: IPv4 Validation** — Accepts iff four dot-separated octets 0-255, no leading zeros
    - **Validates: Requirement 2.2**

  - [ ]* 5.3 Write property tests for enum validation
    - **Property 7: Enum Validation** — Accepts iff value exactly matches predefined allowed values
    - **Validates: Requirements 2.3, 2.4, 2.5**

  - [ ]* 5.4 Write property tests for validation completeness
    - **Property 4: Validation Completeness** — No invalid asset can enter the DataStore through any code path
    - **Validates: Requirements 2.8, 5.1, 6.2**

  - [ ]* 5.5 Write property tests for security properties
    - **Property 18: Formula Injection Protection** — Export prefixes dangerous characters with single quote
    - **Property 19: HTML Tag Stripping on Import** — No HTML tags remain after import
    - **Validates: Requirements 10.2, 12.2**

- [ ] 6. Implement filtering, search, and sort
  - [ ] 6.1 Implement the FilterManager module
    - Implement `apply(assets, filters)` with AND logic across all active column filters (case-insensitive substring)
    - Implement `search(assets, query)` for full-text search across all field values (case-insensitive)
    - Implement `sort(assets, field, dir)` with null/empty values sorted last regardless of direction
    - Implement `getActiveFilters()` to maintain filter state
    - Apply filters first, then search within filtered subset
    - Return all assets when no filters/search active
    - _Requirements: 7.1, 7.2, 7.3, 7.4, 7.5, 7.6, 7.7_

  - [ ]* 6.2 Write property tests for filter and search
    - **Property 10: Filter Subset and AND Logic** — Filtered result is subset where all filters match
    - **Property 11: Search Containment** — Every search result contains query in at least one field
    - **Property 12: Filter-Then-Search Composition** — Filter+search result is subset of filter-only result
    - **Property 13: Sort Ordering** — Consecutive pairs satisfy comparison for specified direction
    - **Validates: Requirements 7.1, 7.2, 7.3, 7.5, 7.6**

- [ ] 7. Implement CRUD controller
  - [ ] 7.1 Implement the CrudController module
    - Implement `create(formData)` — validate, sanitize, check hostname uniqueness, persist, return result
    - Implement `update(id, formData)` — validate, sanitize, check hostname uniqueness (excluding self), persist with updated timestamp
    - Implement `delete(ids)` — prompt confirmation, remove from DataStore, return deleted count
    - Implement `duplicate(id)` — clone asset with new id and timestamps, open in edit form without persisting
    - Reject create/update when hostname conflicts with existing asset (case-insensitive)
    - _Requirements: 6.1, 6.2, 6.3, 6.4, 6.5, 6.6_

  - [ ]* 7.2 Write property tests for CRUD operations
    - **Property 8: ID Uniqueness Invariant** — All asset IDs are distinct after any operation
    - **Property 9: Hostname Uniqueness Invariant** — All hostnames (case-insensitive) are distinct
    - **Property 15: Delete Consistency** — Deleted asset not retrievable, others unaffected
    - **Property 16: Timestamp Monotonicity** — updatedAt >= createdAt always holds
    - **Validates: Requirements 9.3, 9.4, 6.4, 1.5**

- [ ] 8. Checkpoint - Ensure core logic modules are complete
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 9. Implement UI: Table and Card rendering with pagination
  - [ ] 9.1 Implement the TableManager module — Table View
    - Render paginated table with configurable page size (15, 25, 50, 100 — default 15)
    - Implement row selection via checkboxes (per-row + select-all on current page)
    - Preserve selections across page navigation
    - Implement column header click-to-sort with visual direction indicator
    - Display current page number, total pages, and total asset count
    - _Requirements: 8.1, 8.2, 8.4, 8.5, 8.6, 8.7, 8.8, 8.9_

  - [ ] 9.2 Implement the TableManager module — Card View
    - Render assets as title cards in responsive grid layout (8-12 cards per page)
    - Each card shows: hostname, status, environment, assetType, IP address
    - Implement selectable highlight per card for selection
    - Support select/deselect-all control for current page
    - _Requirements: 8.1, 8.3, 8.6_

  - [ ] 9.3 Implement view toggle and state preservation
    - Add UI control to toggle between Table View and Card View
    - Preserve filter state, sort order, and page position when switching views
    - _Requirements: 8.10_

  - [ ]* 9.4 Write property test for pagination completeness
    - **Property 14: Pagination Completeness** — Union of all pages equals complete dataset, no duplicates or omissions
    - **Validates: Requirement 8.2**

  - [ ]* 9.5 Write property test for localStorage serialization
    - **Property 17: localStorage Serialization Round-Trip** — Serialize then deserialize produces equivalent assets
    - **Validates: Requirements 9.1, 9.2**

- [ ] 10. Implement UI: Forms, modals, and CSV upload workflow
  - [ ] 10.1 Implement asset create/edit form modal
    - Build form with all asset fields (required fields marked, enum fields as dropdowns)
    - Wire form submission to CrudController.create/update
    - Display inline validation errors per field
    - Render all user-supplied values via textContent (not innerHTML)
    - _Requirements: 6.1, 6.2, 6.3, 2.8, 2.9, 2.10, 12.1, 12.5_

  - [ ] 10.2 Implement CSV upload UI with preview and column mapping display
    - File input accepting .csv files
    - Show parsed preview with mapped columns highlighted
    - Display validation errors per row (red rows with error tooltips)
    - Show unmapped columns for user awareness
    - Confirm button to proceed with import; display summary on completion
    - _Requirements: 5.1, 5.2, 5.5, 4.4, 4.6_

  - [ ] 10.3 Implement CSV export and download
    - Export current filtered view to CSV via Blob API
    - Filename format: `inventory_export_YYYY-MM-DD.csv`
    - Handle zero-asset export (header-only CSV)
    - Export all defined asset fields as columns
    - _Requirements: 10.1, 10.2, 10.3, 10.4, 10.5_

  - [ ] 10.4 Implement filter/search UI controls
    - Add per-column filter inputs (dropdowns for enum fields, text for others)
    - Add global search text input with 250ms debounce
    - Add clear-all-filters button
    - Wire controls to FilterManager and refresh table on change
    - _Requirements: 7.1, 7.2, 7.3, 7.4, 7.7_

  - [ ] 10.5 Implement delete confirmation dialog and bulk actions
    - Confirmation modal before delete operations
    - Support bulk delete of selected assets
    - Wire to CrudController.delete
    - _Requirements: 6.4_

- [ ] 11. Implement toolbar, notifications, and dashboard stats
  - [ ] 11.1 Implement toolbar with action buttons and stats display
    - Add New Asset, Import CSV, Export CSV buttons
    - Display DataStore stats: total count, counts by type/status/environment
    - Add duplicate asset action (accessible from row context or selection)
    - _Requirements: 6.5, 1.1, 1.2_

  - [ ] 11.2 Implement notification/toast system
    - Success notifications for CRUD operations and imports
    - Error banners for parse failures, quota exceeded, data corruption
    - Import summary notification with created/updated/skipped counts
    - _Requirements: 5.5, 9.5, 9.6_

- [ ] 12. Final integration and wiring
  - [ ] 12.1 Wire all modules together and verify end-to-end flows
    - Connect UI events to controllers and managers
    - Ensure table refreshes after all mutations (create, update, delete, import)
    - Verify no innerHTML usage anywhere — audit all DOM insertion points for textContent
    - Verify no external resource references (links, scripts, imports, fetch calls)
    - Verify file:// protocol compatibility
    - _Requirements: 12.1, 12.4, 12.5, 13.1, 13.2, 13.3, 13.4_

  - [ ] 12.2 Implement customFields support
    - Allow adding/editing/removing custom key-value pairs on assets
    - Enforce customFields constraints: key 1-64 chars, value 1-255 chars, max 20 keys
    - Include customFields in CSV export (as JSON or flattened columns)
    - _Requirements: 1.7_

  - [ ] 12.3 Implement asset maximum field length enforcement
    - Enforce 255-char max on all free-text fields in both UI and import paths
    - _Requirements: 1.6, 2.11_

- [ ] 13. Final checkpoint - Ensure all tests pass
  - Ensure all tests pass, ask the user if questions arise.

## Notes

- Tasks marked with `*` are optional and can be skipped for faster MVP
- Each task references specific requirements for traceability
- Checkpoints ensure incremental validation
- Property tests validate universal correctness properties from the design document using fast-check
- Unit tests validate specific examples and edge cases
- The entire application is a single HTML file — all modules are embedded JavaScript within `<script>` tags
- All DOM rendering uses textContent (never innerHTML) per security requirements
- Testing framework: property tests use fast-check loaded via a separate test harness file (Node.js with jsdom or browser-based test runner)

## Task Dependency Graph

```json
{
  "waves": [
    { "id": 0, "tasks": ["1.1"] },
    { "id": 1, "tasks": ["1.2"] },
    { "id": 2, "tasks": ["1.3", "1.4"] },
    { "id": 3, "tasks": ["3.1", "3.2"] },
    { "id": 4, "tasks": ["3.3", "6.1"] },
    { "id": 5, "tasks": ["3.4", "3.5", "3.6", "5.1", "5.2", "5.3", "5.4", "5.5"] },
    { "id": 6, "tasks": ["6.2", "7.1"] },
    { "id": 7, "tasks": ["7.2"] },
    { "id": 8, "tasks": ["9.1", "9.2"] },
    { "id": 9, "tasks": ["9.3", "9.4", "9.5"] },
    { "id": 10, "tasks": ["10.1", "10.2", "10.3", "10.4", "10.5"] },
    { "id": 11, "tasks": ["11.1", "11.2"] },
    { "id": 12, "tasks": ["12.1", "12.2", "12.3"] }
  ]
}
```
