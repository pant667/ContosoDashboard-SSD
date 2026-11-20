# Document Feature Contract

The existing application is Blazor Server, so this contract defines the service/UI boundary rather than an HTTP REST API.

## DocumentService operations

- `UploadAsync(uploaderId, UploadDocumentRequest, Stream)` validates title/category/type/size/security, verifies context authorization, stages storage, and returns per-file success/failure.
- `GetAccessibleDocumentsAsync(userId, DocumentQuery)` applies ownership/project/team/role/active-share scope before sorting/filtering/search.
- `GetDocumentAsync(documentId, requestingUserId)` returns details only after permission checks.
- `ReplaceFileAsync(documentId, requestingUserId, Stream)` requires owner scope and performs staged replacement.
- `UpdateMetadataAsync(documentId, requestingUserId, DocumentMetadataUpdate)` requires owner scope.
- `DeleteAsync(documentId, requestingUserId)` requires owner or authorized project-manager scope.
- `ShareAsync(documentId, requestingUserId, ShareRequest)` creates explicit user/team shares within existing document scope.
- `RenewShareAsync(documentShareId, requestingUserId)` requires owner/authorized manager and resets expiry to 30 UTC calendar days.
- `RevokeShareAsync(documentShareId, requestingUserId)` requires owner/authorized manager.
- `AttachToTaskAsync(taskId, documentId, requestingUserId)` requires task/document access and inherits the task project.
- `GetTaskDocumentsAsync(taskId, requestingUserId)` requires task access and filters document permission.
- `GetDashboardDocumentsAsync(userId)` returns five latest accessible documents and the unique accessible count.
- `GetRecentDocumentActivityAsync(filter, requestingAdministratorId)` is administrator-only.

## Content operations

- Download and preview operations re-check permissions, open only private storage streams, and record activity.
- Preview supports PDF/JPEG/PNG only.
- Routes and endpoints must never expose storage identity or allow unauthenticated access.

## File and scanner boundaries

- `IFileStorage.SaveAsync(Stream, intendedIdentity)`
- `IFileStorage.OpenAsync(storageIdentity)`
- `IFileStorage.DeleteAsync(storageIdentity)`
- `IFileSecurityScanner.ScanAsync(Stream, originalName, declaredMime)`

## Request validation

- Title required, max 255.
- Category required, from configured predefined values.
- Description max 2000.
- Original filename max 255.
- File size 1 byte through 25 MiB.
- MIME/extension combination is allowlisted for PDF, Word, Excel, PowerPoint, TXT, JPEG, and PNG.
- Project/task IDs must identify authorized entities.
- A share recipient must be exactly one user or team.

## Failure contract

- Validation or scan denial leaves no storage or metadata change.
- Storage failure never leaves a committed record.
- Database failure deletes staged storage.
- Denied access returns no detail or data change.

## Notifications

- Share success creates in-app notifications for each intended recipient.
- Project upload creates notifications for project members.
- Notification and document consistency must be handled by the same transaction/compensation design.
