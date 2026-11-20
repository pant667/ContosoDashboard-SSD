# Implementation Plan: Document Upload and Management

**Branch**: `001-document-management` | **Date**: 2026-11-20 | **Spec**: [spec.md](./spec.md)

**Input**: Feature specification from `/specs/001-document-management/spec.md`

## Summary

Add an offline-capable document-management feature to the existing ASP.NET Core 8/Blazor Server application. The implementation will add EF Core entities and indexes, a document service and replaceable file-storage/scanning abstractions, Blazor pages for upload/browse/share/manage/task integration, and dashboard/report extensions. Authorization will remain at service boundaries and combine ownership, project membership, team scope, role, task access, and active explicit shares. Documents will be stored outside browser-accessible storage, with local disk storage and SQL Server LocalDB as the initial training implementations.

## Technical Context

**Language/Version**: C# / ASP.NET Core 8, .NET 8

**Primary Dependencies**: Entity Framework Core 8, SQL Server LocalDB, Blazor Server, Bootstrap 5.3; existing mock claims/cookie authentication

**Storage**: SQL Server LocalDB for metadata/activity; private local filesystem for document bytes (replaceable through an interface)

**Testing**: Add a focused .NET test project; xUnit preferred if compatible with the local SDK. EF SQL/LocalDB integration tests plus temporary-directory storage tests and scanner validation tests are required.

**Target Platform**: Local Windows training environment with SQL Server LocalDB and Blazor Server

**Project Type**: Single ASP.NET Core 8 Blazor Server web application

**Performance Goals**: 500-document list/search pages <=2 seconds; previews <=3 seconds; uploads up to 25 MB <=30 seconds under typical network conditions

**Constraints**: Offline/local core workflows; no cloud credentials; private file storage; 25 MB per-file maximum; integer keys preserved; category remains text; explicit shares expire 30 calendar days after creation or renewal

**Scale/Scope**: Training-scale collection up to 500 documents, existing mock users/projects/tasks/teams

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Principle / constraint | Plan compliance |
|---|---|
| I. Training-First, Honest Scope | Mock authentication remains explicitly training-only; local security scanning and storage limitations remain documented. |
| II. Secure Authorization and Data Isolation | Document service and content endpoints enforce owner/project/team/role/share/task scope; UI checks complement, not replace, service checks. Add permitted and denied tests. |
| III. Offline and Replaceable Infrastructure | LocalDB plus private local disk are initial implementations; file-storage and scanner interfaces enable future replacements. No cloud dependency. |
| IV. Testable Changes | Plan includes service, authorization, upload rollback, share expiry, and UI/validation test coverage. |
| V. Simple, Maintainable Layering | Reuse Models/Data/Services/Pages; avoid speculative abstractions and keep orchestration in testable document services. |

**Gate result:** PASS. No unresolved constitution exception is identified.

## Project Structure

### Documentation (this feature)

```text
specs/001-document-management/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   └── document-service.md
└── tasks.md
```

`tasks.md` is created only by `/speckit.tasks`, not `/speckit.plan`.

### Source Code (repository root)

```text
ContosoDashboard/
├── Models/
│   ├── Document.cs
│   ├── DocumentShare.cs
│   ├── DocumentActivity.cs
│   └── TaskDocument.cs
├── Services/
│   ├── DocumentService.cs
│   ├── IFileStorage.cs
│   ├── LocalFileStorage.cs
│   ├── IFileSecurityScanner.cs
│   └── LocalFileSecurityScanner.cs
├── Pages/
│   ├── Documents.razor
│   ├── DocumentDetails.razor
│   ├── SharedDocuments.razor
│   └── DocumentUpload.razor
├── Data/
│   └── ApplicationDbContext.cs
└── Program.cs

tests/
└── # Test project location and framework finalized during implementation tasks
```

**Structure Decision**: Keep the feature in the existing single-project Blazor Server structure. Add document entities under `ContosoDashboard/Models`, document orchestration under `ContosoDashboard/Services`, and feature UI under `ContosoDashboard/Pages`. Private storage remains outside `wwwroot`; browser access is only through authorized document operations.

## Complexity Tracking

No violations requiring constitution exception approval.
