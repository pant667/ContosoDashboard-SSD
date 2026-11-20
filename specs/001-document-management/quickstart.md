# Quickstart Validation

## Prerequisites

- .NET 8 SDK
- SQL Server LocalDB
- Run the application from `ContosoDashboard`
- Seeded mock users from `README.md`

## Run

```powershell
cd ContosoDashboard
dotnet run
```

Open the local URL displayed, then login using a seeded mock user.

## Validation scenarios

### Upload

1. Login as `ni.kang@contoso.com`.
2. Submit a PDF under 25 MB with title/category.
3. Confirm success and find it in My Documents.
4. Submit unsupported content and an over-25-MB file; confirm specific rejection and no orphan.

### Search and preview

1. Filter category/project/date and sort title/upload date/category/file size.
2. Search title, description, tag, uploader, or project.
3. Preview PDF/image and download.
4. Confirm unauthorized documents and direct requests are denied.

### Share and expiry

1. Share a document as owner with a second user/team.
2. Confirm recipient notification and Shared with Me visibility.
3. Renew and confirm 30-day UTC expiry.
4. Revoke or evaluate expiry and confirm access is immediately removed.

### Task/project integration

1. Login as project member or manager.
2. Upload/attach from task detail.
3. Confirm task/project association.
4. Confirm authorized task/project visibility and actions.

### Dashboard and reports

1. Login with at least five accessible documents.
2. Confirm exactly five latest accessible documents and unique accessible count.
3. Login as Administrator and open reports.
4. Confirm document-type, uploader, and access-pattern summaries.

## Failure checks

- Deny storage during upload and confirm no metadata.
- Force database failure after staging and confirm staged file removal.
- Repeat replace/delete operations and confirm no partial changes.
- Attempt denied owner/project/team/share/admin operations and confirm rejection.

## Expected evidence

- Automated permitted/denied authorization tests.
- Automated upload rollback tests.
- 30-day share creation/renewal/expiry tests.
- List/search timing with 500 accessible documents.
