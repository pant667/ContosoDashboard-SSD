---
description: "Dependency-ordered implementation tasks for document upload and management"
---

# Tasks: Document Upload and Management

**Input**: Design documents in `specs/001-document-management/` (`spec.md`, `plan.md`, `data-model.md`, `contracts/document-service.md`, `research.md`, and `quickstart.md`)

**Tests**: Included because the project constitution and implementation plan require automated behavior, authorization, failure, storage, scanner, and LocalDB integration coverage.

**Organization**: Tasks are grouped by the five prioritized user stories. Complete shared infrastructure first; each story has an independent verification criterion.

## Path Conventions

- Application: `ContosoDashboard/`
- Tests: `tests/ContosoDashboard.Tests/`
- Feature documentation: `specs/001-document-management/`

## Phase 1: Setup

**Purpose**: Establish test infrastructure and private local storage configuration for the existing .NET 8 Blazor Server application.

- [ ] T001 Create `tests/ContosoDashboard.Tests/ContosoDashboard.Tests.csproj` targeting .NET 8 with xUnit, the test SDK, EF Core SQL Server test dependencies, and a project reference to `ContosoDashboard/ContosoDashboard.csproj`.
- [ ] T002 Add a configurable private application-data storage root in `ContosoDashboard/appsettings.json`; default it outside `wwwroot` and document the local training default without adding cloud credentials.

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Add shared document persistence, storage/scanning boundaries, and centralized authorization needed before any user story.

- [ ] T003 [P] Create `ContosoDashboard/Models/Document.cs` with integer `DocumentId`; `Title` string 255 and required; optional `Description` string 2000?; required `Category` string 100 stored as text and validated against an allowlist; optional `ProjectId`; required immutable `UploaderId`; optional tags; UTC `UploadTime`; `FileSizeBytes` long constrained to 1..25 MiB; validated MIME `FileType` string 255; display-only `OriginalFileName` string 255; unique GUID-based `StorageIdentity` string 255; and `IsAvailable` false until scan/storage succeeds.
- [ ] T004 [P] Create `ContosoDashboard/Models/DocumentShare.cs` with integer key, required document and creator references, UTC creation/expiry and optional renewal/revocation times; require exactly one of `SharedWithUserId` or `SharedWithTeamKey` (team key string 100?), and treat a share as active only when it has no revocation and `ExpiresAt > now`.
- [ ] T005 [P] Create `ContosoDashboard/Models/DocumentActivity.cs` as an immutable audit record with integer key, required document and acting-user references, UTC action time, optional non-sensitive `Details` string 1000?, and text actions limited to Upload, Download, Preview, Delete, Share, ShareRenewal, ShareRevoke, Replace, and Edit.
- [ ] T006 [P] Create `ContosoDashboard/Models/TaskDocument.cs` with integer key, required task/document/attaching-user references, attached time, and optional project ID that must equal the associated task's project.
- [ ] T007 Configure `ContosoDashboard/Data/ApplicationDbContext.cs` with document/share/activity/task-link DbSets, relationships, required/length/uniqueness constraints, recipient and task-link invariants, delete behavior that preserves audit history, and indexes for uploader, project, upload time, category, share expiry, document/action time, and acting-user/action time.
- [ ] T008 [P] Define replaceable `IFileStorage` save/open/delete operations in `ContosoDashboard/Services/IFileStorage.cs` and `IFileSecurityScanner.ScanAsync(Stream, originalName, declaredMime)` in `ContosoDashboard/Services/IFileSecurityScanner.cs`.
- [ ] T009 Implement GUID-identity private local-disk save/open/delete operations in `ContosoDashboard/Services/LocalFileStorage.cs`; reject paths outside the configured application-data root and never expose storage identities through browser-facing URLs.
- [ ] T010 Implement the fail-closed local training scanner in `ContosoDashboard/Services/LocalFileSecurityScanner.cs`, checking extension/MIME compatibility and blocked executable/archive signatures, and document that it is not production malware protection.
- [ ] T011 Centralize owner, administrator, project-member/manager, department/team, task, and active-share access decisions in `ContosoDashboard/Services/DocumentAccessPolicy.cs`; use current membership and role data rather than UI visibility as authorization.
- [ ] T012 Register document services, local storage, and the scanner in `ContosoDashboard/Program.cs`, bind the private storage setting from `ContosoDashboard/appsettings.json`, and ensure file bytes are never served through static-file middleware.
- [ ] T013 Add EF Core migration files under `ContosoDashboard/Data/Migrations/` for document schema changes and update `ContosoDashboard/Program.cs` database initialization so existing LocalDB users receive the schema without losing seeded dashboard data.

**Checkpoint**: Document entities, schema, replaceable private storage, scanner, and service-boundary authorization are ready for story work.

## Phase 3: User Story 1 - Upload and Categorize a Document (Priority: P1) 🎯 MVP

**Goal**: Authenticated employees can upload one or more valid files with required metadata and receive per-file progress and outcome feedback; successful uploads are discoverable in My Documents.

**Independent Test**: Upload an allowed file with valid metadata and find it in My Documents; verify missing metadata, unsupported/oversized/unsafe files, and storage/database failures return actionable errors and leave no orphaned file or metadata.

### Tests for User Story 1

- [ ] T014 [US1] Add upload service tests in `tests/ContosoDashboard.Tests/Services/DocumentServiceUploadTests.cs` covering required title/category, configured category allowlist, supported extension/MIME pairs, 1..25 MiB size limits, multi-file outcomes, scanner rejection, storage failure, and database-failure compensation using temporary storage and deterministic streams.

### Implementation for User Story 1

- [ ] T015 [US1] Implement `UploadAsync` and its request validation in `ContosoDashboard/Services/DocumentService.cs`: scan and stage before metadata commit, authorize personal/project upload context, commit document/task metadata atomically, set availability only after success, and compensate staged bytes/metadata on failure or cancellation.
- [ ] T016 [US1] Create `ContosoDashboard/Pages/DocumentUpload.razor` with authenticated single/multiple file selection, required title/category, optional description/project/tags, upload progress, and a specific success or failure result for every file.
- [ ] T017 [US1] Create the initial My Documents view in `ContosoDashboard/Pages/Documents.razor`, listing only the current user's accessible documents and displaying title, category, upload date, size, and project so successful uploads can be independently verified.

**Checkpoint**: Upload, validation, scanning, rollback, and basic My Documents discovery work without later story features.

## Phase 4: User Story 2 - Browse, Search, and Preview Documents (Priority: P1)

**Goal**: Users can safely find, sort, filter, preview, and download documents within their current permission scope.

**Independent Test**: Seed accessible and inaccessible documents; verify list/search/filter/sort results exclude inaccessible items, and authorized PDF/image preview and download work while direct unauthorized requests disclose no content or storage identity.

### Tests for User Story 2

- [ ] T018 [US2] Add query and content-operation tests in `tests/ContosoDashboard.Tests/Services/DocumentServiceQueryTests.cs` for permission-scoped search, all supported sort/filter fields, shared visibility, unauthorized direct content requests, and PDF/JPEG/PNG preview allowlisting.

### Implementation for User Story 2

- [ ] T019 [US2] Implement `GetAccessibleDocumentsAsync`, `GetDocumentAsync`, and permission-scoped query composition in `ContosoDashboard/Services/DocumentService.cs`, applying access scope before title/description/tag/uploader/project search, category/project/date filters, and title/upload-date/category/file-size sorting.
- [ ] T020 [US2] Extend `ContosoDashboard/Pages/Documents.razor` with category, project, and date-range filters; title, description, tag, uploader, and project search; and title, upload-date, category, and file-size sorting.
- [ ] T021 [US2] Create `ContosoDashboard/Pages/SharedDocuments.razor` to show only documents with an active explicit share to the current user or team, using the same metadata and permission-safe list actions.
- [ ] T022 [US2] Add permission-rechecking download and PDF/JPEG/PNG preview operations to `ContosoDashboard/Services/DocumentService.cs`, expose them through authenticated document-ID routes in `ContosoDashboard/Program.cs` without storage identities, and create `ContosoDashboard/Pages/DocumentDetails.razor` for authorized preview/download.

**Checkpoint**: Search, filtering, sorting, shared-document discovery, and authorized content delivery are independently verifiable.

## Phase 5: User Story 3 - Manage and Share Documents (Priority: P2)

**Goal**: Owners and authorized project managers can manage document metadata/files and explicitly share, renew, or revoke access with notifications and 30-day expiry.

**Independent Test**: An owner updates metadata, replaces a file, shares with a user/team, renews and revokes a share, and confirms recipient notification/access changes; unauthorized edits/deletions cause no data change.

### Tests for User Story 3

- [ ] T023 [US3] Add management and share lifecycle tests in `tests/ContosoDashboard.Tests/Services/DocumentServiceManagementTests.cs` for owner/project-manager permissions, denied mutations, replacement rollback, confirmed deletion, exactly-one-recipient validation, share notification, 30-calendar-day UTC expiry/renewal, and immediate revocation/expiry access loss.

### Implementation for User Story 3

- [ ] T024 [US3] Implement owner-only metadata edits and staged file replacement plus owner/project-manager permanent deletion in `ContosoDashboard/Services/DocumentService.cs`; compensate failed replacements, remove stored bytes and task/share links on deletion, and retain audit activity.
- [ ] T025 [US3] Implement `ShareAsync`, `RenewShareAsync`, and `RevokeShareAsync` in `ContosoDashboard/Services/DocumentService.cs`, enforcing exactly one user/team recipient, authorized sharing scope, UTC expiry 30 days from creation or renewal, and immediate access removal on revoke/expiry.
- [ ] T026 [US3] Extend `ContosoDashboard/Pages/DocumentDetails.razor` with owner/project-manager metadata edit, replacement, delete confirmation, and user/team share-renew-revoke flows; update `ContosoDashboard/Models/Notification.cs` and `ContosoDashboard/Services/NotificationService.cs` so successful explicit shares create in-app recipient notifications.

**Checkpoint**: Management and share lifecycle behavior passes permitted and denied access tests independently.

## Phase 6: User Story 4 - Connect Documents to Tasks and the Dashboard (Priority: P2)

**Goal**: Authorized task users can attach or upload task documents, and the dashboard displays five latest accessible documents plus the unique accessible count.

**Independent Test**: Attach/upload against an accessible task and verify task/project associations and task visibility; verify dashboard recency and count include all accessible owned, project, team, and explicitly shared documents.

### Tests for User Story 4

- [ ] T027 [US4] Add task and dashboard integration tests in `tests/ContosoDashboard.Tests/Services/DocumentTaskDashboardTests.cs` covering task permission denial, inherited task project, task document listing, exactly five latest accessible documents, and unique accessible-document counts across all access scopes.

### Implementation for User Story 4

- [ ] T028 [US4] Implement `AttachToTaskAsync` and `GetTaskDocumentsAsync` in `ContosoDashboard/Services/DocumentService.cs`, verify task/document access, persist `TaskDocument`, and set the link project ID from the task's project.
- [ ] T029 [US4] Create `ContosoDashboard/Pages/TaskDetails.razor` with authorized attachment/upload and document preview/download actions; update `ContosoDashboard/Pages/Tasks.razor` to navigate to task details and show attached documents there.
- [ ] T030 [US4] Implement `GetDashboardDocumentsAsync` in `ContosoDashboard/Services/DocumentService.cs` and extend `ContosoDashboard/Pages/Index.razor` with the five latest unique accessible documents and the total unique accessible-document count, including shared documents.

**Checkpoint**: Task-linked documents and access-aware dashboard summary can be validated independently.

## Phase 7: User Story 5 - Notify Users and Track Document Activity (Priority: P3)

**Goal**: Project members receive new-project-document notifications, document actions are auditable, and administrators can review activity and summary reports.

**Independent Test**: Project uploads notify intended members; upload/download/preview/delete/share/renewal/revoke actions produce actor/document/action/time audit records; administrators can view type, uploader, and access-pattern summaries while non-administrators are denied.

### Tests for User Story 5

- [ ] T031 [US5] Add activity, project-notification, and report authorization tests in `tests/ContosoDashboard.Tests/Services/DocumentActivityReportTests.cs` covering required audit dimensions/actions, project-member recipients, report aggregates, and administrator-only access.

### Implementation for User Story 5

- [ ] T032 [US5] Ensure `ContosoDashboard/Services/DocumentService.cs` records upload, download, preview, deletion, share, renewal, revocation, replacement, and edit activities with acting user, document, action, and UTC timestamp; create project-upload notifications through `ContosoDashboard/Services/NotificationService.cs` for intended project members.
- [ ] T033 [US5] Implement administrator-only recent activity and document-type, active-uploader, and access-pattern report queries in `ContosoDashboard/Services/DocumentService.cs`, enforcing administrator scope at the service boundary.
- [ ] T034 [US5] Create authenticated administrator-only `ContosoDashboard/Pages/DocumentReports.razor` with document type, active uploader, and access-pattern summaries backed by the report service operations.

**Checkpoint**: Notifications, audit records, and administrator reports meet their independent acceptance criteria.

## Phase 8: Polish & Cross-Cutting Concerns

**Purpose**: Validate the complete workflow, performance targets, documentation, and training-only limitations.

- [ ] T035 [P] Add LocalDB integration and 500-document list/search performance coverage in `tests/ContosoDashboard.Tests/Integration/DocumentLocalDbTests.cs`, including project/task/share relationships and the 2-second query targets where the local test environment permits.
- [ ] T036 Update `specs/001-document-management/quickstart.md` and `README.md` with the upload, browse, share/expiry, task, dashboard, report, LocalDB, private-storage, scanner, mock-authentication, and training-only validation instructions and limitations.
- [ ] T037 Run `dotnet test tests/ContosoDashboard.Tests/ContosoDashboard.Tests.csproj` and `dotnet build ContosoDashboard/ContosoDashboard.csproj`; fix failures and record command outcomes or environment blockers in `specs/001-document-management/quickstart.md`.

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No prerequisites; creates the test project and private-storage configuration.
- **Foundational (Phase 2)**: Depends on Setup; all entities, database mappings, scanner/storage implementations, DI, migration, and authorization policy block story implementation.
- **User Stories (Phases 3–7)**: Depend on the completed foundation. Priority and implementation/file overlap recommend US1 → US2 → US3, then US4; US5 follows the action logging and notification work in US3/US4.
- **Polish (Phase 8)**: Depends on all selected stories.

### User Story Dependencies

- **US1 (P1)**: Starts after Phase 2; no story dependency. MVP.
- **US2 (P1)**: Starts after Phase 2; builds on the initial My Documents page and upload data from US1, so deliver after US1 for a clean independently testable increment.
- **US3 (P2)**: Starts after US2 content/list operations and US1 upload; share and management behavior can then be verified against existing documents.
- **US4 (P2)**: Starts after US1 upload and Phase 2; task/dashboard work is logically separable from US3, but shared `DocumentService.cs` integration should be coordinated if run concurrently.
- **US5 (P3)**: Starts after the shared activity model and can follow US3/US4 to cover all actions and project notifications in its reports.

### Within Each User Story

- Write and run the story's tests before its implementation tasks.
- Complete service authorization and data behavior before UI integration.
- Keep each UI action behind service-level authorization; UI visibility is not an access check.
- Preserve file/metadata rollback and audit behavior as applicable to the story.

### Parallel Opportunities

- T003–T006 are independent model files and can be created in parallel.
- T008 interface definitions can be prepared in parallel; T009 and T010 are separate adapter files once their interfaces and configuration are agreed.
- Within each story, test files and implementation files can be worked on in parallel only when tests use the agreed service contract and target distinct files; implementation should not be considered complete until tests pass.
- T035 can run independently from T036 once story interfaces stabilize.
- Cross-story parallelism is possible for US3 and US4 with separate ownership, but both touch `DocumentService.cs`; coordinate integration to avoid conflicting edits.

## Parallel Example: Foundational Models

```text
Task: T003 Create Document model in ContosoDashboard/Models/Document.cs
Task: T004 Create DocumentShare model in ContosoDashboard/Models/DocumentShare.cs
Task: T005 Create DocumentActivity model in ContosoDashboard/Models/DocumentActivity.cs
Task: T006 Create TaskDocument model in ContosoDashboard/Models/TaskDocument.cs
```

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1 Setup and Phase 2 Foundational prerequisites.
2. Complete Phase 3 US1 upload validation, storage/scanning, rollback, UI, and initial My Documents discovery.
3. Run US1 upload tests and verify valid upload plus invalid/unsafe/storage/database failure behavior independently.
4. Stop for MVP validation before adding search, sharing, task, or reporting scope.

### Incremental Delivery

1. Deliver US1 upload as the minimum useful release.
2. Add US2 browse/search/preview/download and verify permission filtering.
3. Add US3 management and expiring explicit shares.
4. Add US4 task and dashboard integration.
5. Add US5 project notifications, audit review, and administration reports.
6. Complete Phase 8 cross-cutting validation and training documentation.

## Independent Test Criteria by Story

- **US1**: Valid single/multiple uploads appear in My Documents; invalid, unsafe, oversized, storage-failed, and DB-failed uploads leave no orphaned records or bytes.
- **US2**: Lists/search/filters/sorts show only accessible records; permitted preview/download works and unauthorized direct access returns no content.
- **US3**: Authorized edits/replacements/deletions and user/team share lifecycle succeed; denied changes do not mutate data; expiry/revocation removes recipient access.
- **US4**: Authorized task attachments inherit the task project; dashboard shows five latest accessible documents and the exact unique accessible count.
- **US5**: Relevant actions record actor/document/action/time; project members receive intended notifications; only administrators can retrieve activity reports.

## Notes

- Every task uses the required checkbox, sequential ID, optional `[P]`, story label for story phases, and an explicit file path.
- `[P]` marks work in different files that has no incomplete-task dependency.
- The local scanner is a training safeguard, not production malware protection; mock authentication remains training-only.
