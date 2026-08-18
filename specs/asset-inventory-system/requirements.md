# Requirements Document

## Introduction

This document defines formal requirements for the Asset Inventory System — a self-contained, client-side HTML/JavaScript web application providing CMDB-style asset inventory management. The system targets infrastructure/systems teams managing Windows/VMware environments, enabling full CRUD operations, CSV bulk import/export, multi-column filtering, and browser localStorage persistence without any backend server dependency.

## Glossary

- **System**: The Asset Inventory System web application running in the browser
- **Asset**: A single inventory record representing a server, network device, storage, application, or other infrastructure component
- **DataStore**: The localStorage-backed persistence layer managing asset records
- **CSV_Parser**: The component responsible for parsing and generating RFC 4180 compliant CSV text
- **Column_Mapper**: The component that matches CSV column headers to internal CMDB field names using alias tables
- **Field_Validator**: The component that validates asset field values against format rules and allowed values
- **Filter_Manager**: The component managing multi-column filtering, full-text search, and sort state
- **Table_Manager**: The component rendering paginated asset data into the HTML table
- **CRUD_Controller**: The component orchestrating create, read, update, and delete operations
- **Upsert**: An operation that inserts a new record or updates an existing record if a matching hostname already exists

## Requirements

### Requirement 1: Asset Data Model

**User Story:** As an infrastructure team member, I want a well-defined asset data model with CMDB-standard fields, so that I can consistently track infrastructure components across my environment.

#### Acceptance Criteria

1. THE System SHALL define assets with the following required fields: hostname, environment, status, and assetType
2. THE System SHALL define assets with the following optional fields: ipAddress, os, location, owner, description, cluster, datacenter, cpuCount, memoryGB, storageGB, application, businessUnit, url link, and customFields
3. WHEN a new asset is created, THE System SHALL auto-generate a UUID v4 identifier and assign it as the asset's unique id
4. WHEN a new asset is created, THE System SHALL auto-generate createdAt and updatedAt fields as ISO 8601 UTC timestamps with millisecond precision
5. WHEN an asset field value is changed via CRUD operation or CSV import, THE System SHALL update the updatedAt timestamp to the current time while preserving the original createdAt timestamp
6. THE System SHALL enforce a maximum length of 255 characters for free-text string fields (os, location, owner, description, cluster, datacenter, application, businessUnit)
7. THE System SHALL define customFields as a string-keyed object where each key is 1-64 characters, each value is a string of 1-255 characters, and the maximum number of keys per asset is 20

### Requirement 2: Field Validation

**User Story:** As an infrastructure team member, I want field-level validation enforcing correct formats, so that data quality is maintained across the inventory.

#### Acceptance Criteria

1. WHEN a hostname is provided, THE Field_Validator SHALL accept only strings of 1-63 characters containing alphanumeric characters and hyphens with no leading or trailing hyphen
2. WHEN an ipAddress is provided, THE Field_Validator SHALL accept only valid IPv4 addresses with four octets each in range 0-255 and no leading zeros
3. WHEN an environment value is provided, THE Field_Validator SHALL accept only one of (case-insensitive comparison): Production, Staging, Development, Test, DR, Lab, Decommissioned
4. WHEN a status value is provided, THE Field_Validator SHALL accept only one of (case-insensitive comparison): Active, Inactive, Provisioning, Maintenance, Decommissioning, Retired
5. WHEN an assetType value is provided, THE Field_Validator SHALL accept only one of (case-insensitive comparison): Server, Virtual Machine, Network Device, Storage, Application, Database, Other
6. WHEN cpuCount is provided, THE Field_Validator SHALL accept only positive integers in the range 1 to 9999
7. WHEN memoryGB or storageGB is provided, THE Field_Validator SHALL accept only positive numbers in the range 0.01 to 999999.99 with at most two decimal places
8. IF any required field is empty or whitespace-only, THEN THE Field_Validator SHALL reject the asset and return an error message identifying each failing field name and indicating it is required
9. IF an optional field fails its format validation, THEN THE Field_Validator SHALL reject the asset and return an error message identifying the field name and the validation rule that was violated
10. IF multiple fields fail validation simultaneously, THEN THE Field_Validator SHALL return all validation errors in a single response rather than stopping at the first failure
11. WHEN a free-text optional field (os, location, owner, description, cluster, datacenter, application, or businessUnit) is provided, THE Field_Validator SHALL accept only strings of 1 to 255 characters

### Requirement 3: CSV Parsing

**User Story:** As an infrastructure team member, I want to import CSV files from various tools (RVTools, AD exports, vCenter scripts), so that I can bulk-populate my inventory without manual data entry.

#### Acceptance Criteria

1. WHEN a CSV file is provided, THE CSV_Parser SHALL parse it according to RFC 4180 rules handling quoted fields, embedded commas, and embedded newlines
2. WHEN a CSV file is provided, THE CSV_Parser SHALL auto-detect the delimiter by counting occurrences of comma, semicolon, and tab in the first non-empty row and selecting the character with the highest occurrence count
3. WHEN a CSV file contains CRLF or LF line endings, THE CSV_Parser SHALL treat both as valid record terminators and produce identical parsed output regardless of line ending style
4. THE CSV_Parser SHALL treat the first row as column headers and produce structured output containing a headers array and a rows array of string arrays where each row array length equals the headers array length
5. WHEN generating CSV output for export, THE CSV_Parser SHALL produce valid RFC 4180 compliant text from an array of assets using CRLF line endings and quoting any field that contains a comma, double-quote, or newline
6. IF a CSV file contains malformed data including unclosed quoted fields or rows with fewer columns than the header row, THEN THE CSV_Parser SHALL return an error indicating the line number and nature of the malformation rather than producing partial output
7. IF a CSV file is empty or contains only a header row with no data rows, THEN THE CSV_Parser SHALL return a valid result with the headers array populated (or empty for a zero-byte file) and an empty rows array

### Requirement 4: Column Mapping

**User Story:** As an infrastructure team member, I want CSV columns automatically mapped to CMDB fields using fuzzy matching and alias tables, so that imports from different tools work without manual column assignment.

#### Acceptance Criteria

1. WHEN CSV headers are provided, THE Column_Mapper SHALL match them to CMDB field names by first attempting exact case-insensitive comparison against a predefined alias table, and then applying normalized fuzzy matching (ignoring whitespace, underscores, and hyphens) to unmatched headers
2. THE Column_Mapper SHALL include at minimum the following aliases in its predefined alias table: for hostname — "Name", "VM Name", "VM", "Computer", "ComputerName", "CN", "Device Name", "DNS Name"; for ipAddress — "IP", "IP Address", "IPAddress", "Primary IP", "IP Addresses"; for os — "OS", "Operating System", "OS according to the configuration file", "Guest OS"; for cpuCount — "CPUs", "Num CPUs", "CPU", "NumberOfCPUs"; for memoryGB — "Memory", "Memory MB", "RAM", "Size MB"; for cluster — "Cluster", "VMCluster"; for datacenter — "Datacenter", "DC"
3. WHEN multiple CSV headers could map to the same CMDB field, THE Column_Mapper SHALL use the first match by left-to-right column position in the CSV and leave subsequent duplicates unmapped
4. WHEN a CSV header does not match any known alias, THE Column_Mapper SHALL include it in an unmapped headers list within the mapping result for user review
5. WHEN no CSV headers can be mapped to known fields, THE Column_Mapper SHALL report an error indicating no mappable columns were found and SHALL NOT proceed with import
6. THE Column_Mapper SHALL return a mapping result object containing: an array of mapped pairs (CSV header to CMDB field name), an array of unmapped CSV header names, and the total count of successfully mapped columns

### Requirement 5: CSV Import with Upsert

**User Story:** As an infrastructure team member, I want bulk CSV import with upsert behavior keyed on hostname, so that re-importing updated exports merges changes without creating duplicates.

#### Acceptance Criteria

1. WHEN a valid CSV is imported, THE System SHALL validate each row and persist only rows that pass validation
2. WHEN a CSV row fails validation, THE System SHALL skip that row and record the row number and specific field errors
3. WHEN an imported asset has a hostname matching an existing asset (case-insensitive), THE DataStore SHALL update the existing record by overwriting all mapped fields with the CSV row values (including empty values) while preserving its original id and createdAt timestamp
4. WHEN an imported asset has a hostname not matching any existing asset, THE DataStore SHALL insert it as a new record
5. WHEN import completes, THE System SHALL display a summary showing counts of created records, updated records, and skipped records with their error details
6. WHEN the same CSV is imported twice, THE DataStore SHALL contain the same number of assets as after the first import
7. WHEN a CSV file contains multiple rows with the same hostname (case-insensitive), THE System SHALL process them in file order so that the last occurrence determines the final stored values

### Requirement 6: CRUD Operations

**User Story:** As an infrastructure team member, I want to create, read, update, and delete assets through the UI, so that I can manage individual records as my environment changes.

#### Acceptance Criteria

1. WHEN a user submits a new asset with valid data and a hostname not already present in the DataStore (case-insensitive), THE CRUD_Controller SHALL persist it to the DataStore with auto-generated id and timestamps, and refresh the table display to include the new asset
2. WHEN a user submits a new asset with invalid data, THE CRUD_Controller SHALL display one or more validation error messages identifying each failing field and its specific violation, without persisting any data to the DataStore
3. WHEN a user updates an existing asset with valid data, THE CRUD_Controller SHALL persist the changes, update the updatedAt timestamp to the current time, and preserve the original id and createdAt timestamp
4. WHEN a user initiates deletion of one or more selected assets, THE CRUD_Controller SHALL prompt the user for confirmation before removing them from the DataStore, and after confirmed deletion the assets SHALL no longer be retrievable by id
5. WHEN a user duplicates an existing asset, THE CRUD_Controller SHALL open an edit form pre-filled with the source asset's field values, a new unique id, and new createdAt and updatedAt timestamps, without persisting the duplicate until the user submits it
6. IF a user submits a new asset or updates an existing asset with a hostname that matches another asset already in the DataStore (case-insensitive), THEN THE CRUD_Controller SHALL reject the operation and display an error message indicating the hostname is already in use

### Requirement 7: Filtering and Search

**User Story:** As an infrastructure team member, I want multi-column filtering and full-text search, so that I can quickly locate specific assets in a large inventory.

#### Acceptance Criteria

1. WHEN column filters are applied, THE Filter_Manager SHALL return only assets where each filtered field's value contains the corresponding filter string as a substring (AND logic across all active filters)
2. WHEN a full-text search query of 1 or more non-whitespace characters is entered, THE Filter_Manager SHALL return only assets containing the query string as a substring in at least one field value
3. WHEN both column filters and a search query are active, THE Filter_Manager SHALL apply column filters first, then search within that filtered subset
4. WHEN no filters or search query are active, THE Filter_Manager SHALL return all assets
5. THE Filter_Manager SHALL perform case-insensitive matching for both column filters and full-text search
6. WHEN a sort is applied to a field containing null or empty values, THE Filter_Manager SHALL order results by the specified field in ascending or descending direction, placing assets with null or empty sort-field values after all non-empty values regardless of sort direction
7. WHEN a user clears all column filters and the search query, THE Filter_Manager SHALL return the full unfiltered asset set within 100 milliseconds for datasets of up to 10,000 assets

### Requirement 8: Table Rendering and Pagination

**User Story:** As an infrastructure team member, I want a paginated table with row selection, so that I can efficiently browse and act on large inventories without performance degradation.

#### Acceptance Criteria

1. THE Table_Manager SHALL support two display modes: Table View (default) and Card View, toggled via a UI control
2. WHEN in Table View, THE Table_Manager SHALL render assets in pages with a configurable page size selectable from the values 15, 25, 50, or 100 rows per page, defaulting to 15 (optimized for 24-inch or larger monitors at 1920x1080 resolution)
3. WHEN in Card View, THE Table_Manager SHALL render assets as title cards in a responsive grid layout displaying 8-12 cards per page (optimized for 24-inch or larger monitors), with each card showing hostname, status, environment, assetType, and IP address
4. WHEN pagination is applied, THE Table_Manager SHALL ensure the union of all pages equals the complete filtered dataset with no duplicates or omissions
5. THE Table_Manager SHALL display the current page number, total page count, and total asset count so the user can identify their position within the dataset
6. THE Table_Manager SHALL support row selection via a checkbox per row (Table View) or a selectable highlight per card (Card View) for individual selection, and a header control to select or deselect all items on the current page
7. WHEN a user navigates to a different page, THE Table_Manager SHALL preserve selections made on previously visited pages
8. WHEN in Table View and a column header is clicked, THE Table_Manager SHALL sort by that column in ascending order on first click and toggle between ascending and descending order on subsequent clicks of the same column
9. WHEN a column header is clicked, THE Table_Manager SHALL display a visual indicator on the sorted column showing the current sort direction
10. WHEN the user switches between Table View and Card View, THE Table_Manager SHALL preserve the current filter state, sort order, and page position

### Requirement 9: Data Persistence

**User Story:** As an infrastructure team member, I want all data persisted in browser localStorage, so that my inventory survives browser tab closes and page refreshes without any server dependency.

#### Acceptance Criteria

1. THE DataStore SHALL serialize all assets as JSON and store them in browser localStorage under a single key
2. WHEN assets are retrieved, THE DataStore SHALL deserialize from localStorage and return Asset objects with all field values matching their original types (strings as strings, numbers as numbers, timestamps as ISO 8601 strings)
3. THE DataStore SHALL maintain unique IDs across all assets — no two assets share the same id
4. THE DataStore SHALL maintain unique hostnames (case-insensitive) across all assets — no two assets share the same hostname
5. IF localStorage quota is exceeded during a write operation, THEN THE System SHALL display an error message indicating storage limits have been reached and SHALL preserve the previously stored data without corruption
6. IF localStorage contains data that cannot be parsed as valid JSON, THEN THE DataStore SHALL treat the store as empty, return an empty asset array, and display a warning message indicating data corruption was detected

### Requirement 10: CSV Export

**User Story:** As an infrastructure team member, I want to export my inventory to CSV, so that I can share data with other teams or import into other tools.

#### Acceptance Criteria

1. WHEN export is triggered, THE CSV_Parser SHALL generate valid RFC 4180 CSV text from the currently displayed (filtered) asset dataset, including a header row as the first line containing field names
2. WHEN a field value begins with any of the characters =, +, -, or @, THE CSV_Parser SHALL prefix that value with a single quote to prevent formula injection in spreadsheet applications
3. WHEN the user initiates an export, THE System SHALL produce a downloadable CSV file via the Blob API with a filename containing the prefix "inventory_export" and a date stamp in ISO 8601 format
4. WHEN the asset dataset contains zero assets, THE System SHALL still generate a valid CSV file containing only the header row
5. THE CSV_Parser SHALL export all defined asset fields (hostname, environment, status, assetType, ipAddress, os, location, owner, description, cluster, datacenter, cpuCount, memoryGB, storageGB, application, businessUnit) as columns in the output

### Requirement 11: CSV Round-Trip Integrity

**User Story:** As an infrastructure team member, I want export-then-import to preserve my data, so that I can use CSV as a reliable backup and transfer format.

#### Acceptance Criteria

1. WHEN a valid asset dataset is exported to CSV and that CSV is subsequently imported, THE System SHALL produce an equivalent set of assets where each re-imported asset has matching values for all user-editable fields (hostname, ipAddress, os, environment, location, owner, status, assetType, description, cluster, datacenter, cpuCount, memoryGB, storageGB, application, businessUnit) while id, createdAt, and updatedAt are excluded from the equivalence check
2. THE CSV_Parser SHALL format field values such that re-parsing the CSV output produces string values equal to the original field values after trim-normalization
3. WHEN a CSV produced by export contains formula injection prefixes (leading single quote added per Requirement 10.2), THE CSV_Parser SHALL strip the leading single quote during re-import so that the original field value is restored
4. WHEN numeric fields (cpuCount, memoryGB, storageGB) are exported and re-imported, THE System SHALL preserve the numeric value such that the re-imported number equals the original value

### Requirement 12: Security and Input Safety

**User Story:** As an infrastructure team member, I want all user-supplied data rendered safely, so that malicious CSV content cannot execute scripts in the browser.

#### Acceptance Criteria

1. THE System SHALL render all user-supplied values using textContent (not innerHTML) at every DOM insertion point including table cells, form field pre-fills, modal dialogs, and notification messages
2. WHEN importing CSV data, THE System SHALL remove all substrings matching the pattern `<[^>]*>` and any remaining angle bracket characters (`<` and `>`) from field values before storage
3. WHEN a user creates or edits an asset through the UI, THE System SHALL remove all substrings matching the pattern `<[^>]*>` and any remaining angle bracket characters (`<` and `>`) from field values before storage
4. THE System SHALL operate entirely client-side with no network transmission of asset data
5. THE System SHALL not assign user-supplied values to element properties that execute code including innerHTML, outerHTML, document.write, eval, or javascript: protocol URIs

### Requirement 13: Deployment and Dependencies

**User Story:** As an infrastructure team member, I want the entire application delivered as a single HTML file with zero external dependencies, so that any team member can use it by simply opening a file in their browser.

#### Acceptance Criteria

1. THE System SHALL be implemented as a single HTML file with all CSS and JavaScript embedded inline, containing no external references such as `<link>` stylesheet tags, `<script src>` tags, `@import` directives, or remote `url()` references
2. THE System SHALL require no backend server, external CDN resources, or network connectivity to function, and SHALL NOT issue any network requests via fetch, XMLHttpRequest, or dynamic script loading
3. THE System SHALL function in modern browsers supporting ES6+ (Chrome 60+, Firefox 55+, Edge 79+)
4. THE System SHALL function when opened directly from the local filesystem via the file:// protocol without requiring a web server
