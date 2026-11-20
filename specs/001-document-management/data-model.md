# Data Model

## Document

| Field | Type | Rules |
|---|---|---|
| DocumentId | int PK | Existing integer-key convention |
| Title | string 255 | Required |
| Description | string 2000? | Optional |
| Category | string 100 | Required; text allowlist |
| ProjectId | int? FK | Optional project association |
| UploaderId | int FK | Required; immutable |
| Tags | text or separate join entity | Optional custom tags |
| UploadTime | DateTime | UTC |
| FileSizeBytes | long | 1..25 MiB |
| FileType | string 255 | Validated MIME |
| OriginalFileName | string 255 | Display only |
| StorageIdentity | string 255 unique | GUID-based private identity |
| IsAvailable | bool | False until scan/storage succeeds |

Relationships: uploader, optional project, many shares/activities/task links.

Indexes: uploader, project, upload time, category, and accessible-query composites as warranted.

## DocumentShare

| Field | Type | Rules |
|---|---|---|
| DocumentShareId | int PK | Integer key |
| DocumentId | int FK | Required |
| SharedWithUserId | int? FK | One recipient form |
| SharedWithTeamKey | string 100? | Department/team key |
| CreatedAt | DateTime | UTC |
| ExpiresAt | DateTime | Created/renewed +30 days |
| RenewedAt | DateTime? | Set on renewal |
| RevokedAt | DateTime? | Set on revoke |
| CreatedByUserId | int FK | Acting user |

Exactly one of user/team recipient is set. Active requires no revoke and `ExpiresAt > now`.

## DocumentActivity

Immutable audit record.

| Field | Type | Rules |
|---|---|---|
| DocumentActivityId | int PK | Integer key |
| DocumentId | int FK | Required |
| ActingUserId | int FK | Required |
| Action | text | Upload, Download, Preview, Delete, Share, ShareRenewal, ShareRevoke, Replace, Edit |
| ActionTime | DateTime | UTC |
| Details | string 1000? | Optional non-sensitive summary |

Indexes: document/action time and acting user/action time.

## TaskDocument

Join between existing TaskItem and Document.

| Field | Type | Rules |
|---|---|---|
| TaskDocumentId | int PK | Integer key |
| TaskId | int FK | Required |
| DocumentId | int FK | Required |
| ProjectId | int? | Must equal task project |
| AttachedByUserId | int FK | Required |
| AttachedAt | DateTime | UTC |

## Existing Entity Integration

- **User:** uploader, owner, recipient, and role/team scope.
- **Project:** project document scope and project-manager scope.
- **ProjectMember:** project/task access.
- **TaskItem:** task association and inherited project.
- **Notification:** share/project-document notifications.

## Lifecycle Rules

1. A document starts unavailable until scan/storage succeeds.
2. Metadata commits only after storage and scan success.
3. Replace uses a staged identity and guards concurrent replace/delete.
4. Delete removes file, metadata, shares, and task links but retains audit activity.
5. Share is active only before expiry and without revoke.
6. Renewal resets expiry to 30 UTC calendar days from renewal.
7. Access recalculates project membership and share state.
