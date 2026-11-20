# Phase 0 Research

## Security scanning

- **Decision:** Introduce `IFileSecurityScanner` and implement a local training scanner that validates declared/detected extension-MIME compatibility and rejects blocked executable/archive signatures. It will fail closed on read or scan errors and will be documented as not production malware protection.
- **Rationale:** Core workflows must operate locally and the constitution forbids implying production safeguards.
- **Alternatives considered:** External cloud antivirus (rejected: network/credentials); no scanning (rejected: conflicts with FR-007); a Windows-only engine (rejected: poor reproducibility).

## File storage

- **Decision:** Store bytes beneath a private application-data directory, use GUID storage identities, and expose bytes only through authorized download/preview operations. Define `IFileStorage` for save/open/delete operations.
- **Rationale:** Files remain outside `wwwroot`, original names cannot collide, and failed operations can be compensated.
- **Alternatives considered:** Files under `wwwroot` (rejected: browser bypass); database blobs (rejected for replaceable file workflow); Azure Blob (deferred).

## Upload consistency

- **Decision:** Scan and stage first, store under a GUID, then commit metadata and task links in one database transaction. On database failure or cancellation, delete the staged file and compensate metadata. Guard repeat operations.
- **Rationale:** Prevents orphaned metadata and files.
- **Alternatives considered:** Metadata-first (rejected: orphan risk); file-first without compensation (rejected).

## Authorization scope

- **Decision:** Centralize checks in `DocumentService`. Access comes from ownership, administrator role, project manager/member scope, existing department/team scope, task access, or an active user/team share. Delete additionally requires owner or project-manager scope. Share renewal/revoke requires existing owner/authorized-manager scope.
- **Rationale:** Satisfies FR-015/034 and prevents UI-only authorization.
- **Alternatives considered:** Razor-page-only authorization (rejected); database row-level security (not present).

## Team scope

- **Decision:** Derive initial team scope from existing `Department`, `UserRole`, and project memberships rather than adding a separate team entity. A Team Lead may manage team-member documents in the same department only within otherwise authorized project/document scope.
- **Rationale:** The starter model has no team entity.
- **Alternatives considered:** New team entity (rejected: speculative); global team-lead scope (rejected: overly broad).

## Share lifecycle

- **Decision:** Store `ExpiresAt` and `RenewedAt`. Creation sets expiry to 30 calendar days from creation in UTC. Renewal sets renewal time and expiry to 30 days from renewal. Access requires expiry after now and no revoke.
- **Rationale:** UTC makes lifecycle rules deterministic.
- **Alternatives considered:** Read-time-only expiry (insufficient lifecycle visibility); fixed end-of-day expiry (not as direct).

## Categories and supported types

- **Decision:** Category remains text and must use a configurable predefined allowlist. Upload requires title and category. Allow PDF, DOC/DOCX, XLS/XLSX, PPT/PPTX, TXT, JPEG, and PNG using consistent extension/MIME validation.
- **Rationale:** Matches FR-038/002 and supports SC-008.
- **Alternatives considered:** Category enum (conflicts with text storage); unrestricted text (weaker validation).

## Testing approach

- **Decision:** Add implementation tests for validation, owner/project/team/share/administrator authorization, share expiry, upload rollback, and storage/scanner failures. Use temporary directories and deterministic streams.
- **Rationale:** The starter has no test project, while the constitution requires automated tests and failure coverage.
- **Alternatives considered:** EF in-memory only (rejected: incomplete); manual-only validation (rejected).
