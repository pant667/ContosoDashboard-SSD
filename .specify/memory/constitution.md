<!--
Sync Impact Report
Version change: unversioned template → 1.0.0 (initial constitution)
Modified principles:
- Unspecified principle slot → I. Training-First, Honest Scope
- Unspecified principle slot → II. Secure Authorization and Data Isolation
- Unspecified principle slot → III. Offline and Replaceable Infrastructure
- Unspecified principle slot → IV. Testable Changes
- Unspecified principle slot → V. Simple, Maintainable Layering
Added sections: Additional Constraints; Development Workflow and Quality Gates
Removed sections: None
Follow-up TODOs: Confirm the original ratification date; it is not established by
the repository history or project documentation.
-->

# ContosoDashboard Constitution

## Core Principles

### I. Training-First, Honest Scope
All application behavior and documentation MUST represent ContosoDashboard as a
training project, not a production-ready product. Mock authentication,
deliberately simplified behavior, and known limitations MUST be identified
accurately. Any example that depends on production-grade identity or operations
MUST be labeled as guidance rather than as an implemented capability. This
prevents learners from mistaking training shortcuts for production safeguards.

### II. Secure Authorization and Data Isolation
Every protected operation MUST enforce authentication and authorization at the
appropriate application and service boundaries. Data access MUST verify the
current user's permitted scope, including project or task membership where
applicable; hiding a UI element alone is not authorization. Changes to access
control MUST include tests for allowed and denied access. Because authentication
is mocked for training, documentation MUST NOT imply that it secures real
accounts.

### III. Offline and Replaceable Infrastructure
The application MUST remain usable for its documented local training workflow
without requiring cloud accounts or external services. Infrastructure access
MUST be kept behind suitable interfaces and dependency injection where this
supports independent implementations; business rules MUST NOT depend directly
on a cloud provider. A proposed cloud migration is not an implemented
capability until its implementation and tests exist.

### IV. Testable Changes
Each change that alters observable behavior MUST add or update automated tests
for its expected behavior and important failure or authorization cases. Changes
MUST pass the relevant available tests before merge; any test that cannot be
run MUST be identified with the reason and the remaining risk. Tests and
documentation are part of the change, not optional follow-up work.

### V. Simple, Maintainable Layering
Code MUST preserve clear responsibilities among the UI, services, data access,
and models. New abstractions MUST address a concrete testing, change-isolation,
or infrastructure boundary; speculative frameworks and duplicate layers MUST
not be introduced without a documented need. Prefer straightforward,
asynchronous data access and keep business rules in testable services.

## Additional Constraints

- The documented application stack is ASP.NET Core 8, Blazor Server, Entity
  Framework Core, SQL Server LocalDB, and Bootstrap. Stack changes MUST be
  documented and verified against the local training setup.
- Runtime workflows MUST NOT require network access or external service
  credentials unless the change explicitly revises the offline-training
  constraint and includes a local alternative.
- Training-only limitations MUST remain visible in documentation. The project
  MUST NOT be represented as suitable for production deployment.

## Development Workflow and Quality Gates

- Feature work MUST have clear requirements and acceptance criteria before
  implementation; plans and task lists MUST remain consistent with those
  requirements when the Spec Kit workflow is used.
- Before a change is accepted, reviewers MUST check its tests, authorization
  boundaries, documentation, and consistency with these principles.
- Changes to security-sensitive behavior MUST include tests covering both
  permitted and forbidden cases and documentation of any training-only
  simplification.
- Any exception to a MUST in this constitution MUST be stated in the change
  proposal, justified, and approved as part of the amendment or review.

## Governance

This constitution governs project decisions and supersedes conflicting local
development guidance. Amendments MUST be proposed with their rationale and a
Sync Impact Report, reviewed and approved by the project maintainers, and
updated in this file. Related project guidance MUST be synchronized when an
amendment changes its requirements.

Constitution versions use semantic versioning. A MAJOR increment denotes a
backward-incompatible removal or redefinition of a principle or governance
rule; a MINOR increment denotes a new principle or section, or materially
expanded requirements; a PATCH increment denotes clarifications and
non-semantic wording changes. The version and amendment date MUST be updated
with every approved change.

Every feature review MUST evaluate compliance with this constitution.
Reviewers MUST require tests and documentation specified by the applicable
principles, and MUST record and approve any exception before merge. Compliance
does not make the training application production-ready.

**Version**: 1.0.0 | **Ratified**: TODO(RATIFICATION_DATE): confirm original adoption date | **Last Amended**: 2026-11-20
