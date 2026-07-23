# student-access-requests automation

This folder contains the first automation scaffold for public access and assignment requests.

## Issue forms

- `.github/ISSUE_TEMPLATE/access-request.yml`
- `.github/ISSUE_TEMPLATE/assignment-request.yml`

Both templates auto-apply type labels and the `pending` state label.

## Workflows

- `.github/workflows/ensure-pending-state.yml`
  - Ensures `pending` exists on newly opened request issues.

- `.github/workflows/validate-access-request.yml`
  - Trigger: access request issue opened, reopened, or edited.
  - Loads access codes from the private `student-registry` repo.
  - Redacts the submitted access code immediately.
  - If invalid, comments, closes the issue, and marks it `failed`.
  - If valid, marks it `validated` and notifies the reviewer handle.

- `.github/workflows/process-approved-access-request.yml`
  - Trigger: `approve` label added to an `access-request` issue.
  - Requires the request to already have the `validated` label.
  - Invites the requester to `student-intake` and posts a direct intake form link.

- `.github/workflows/reinvite-student-intake.yml`
  - Trigger: `reinvite` label added to a processed `access-request` issue.
  - Re-sends invitation to `student-intake` and posts intake instructions again.

- `.github/workflows/process-approved-assignment-request.yml`
  - Trigger: `approve` label added to an `assignment-request` issue.
  - First check: issue author must exist in `student-registry/data/students.json`.
  - If author is not registered, processing stops immediately and marks the issue `failed`.
  - Only registered students proceed to organization/template/repository checks.

## Required labels

Create these labels once in the destination repository:

- access-request
- pending
- validated
- failed
- approve
- reinvite
- reinvited
- assignment-request
- processing
- processed

## Next implementation step

Replace dry-run placeholders with real operations:

- access flow: validate identity and access code, then invite to `student-intake`.
- assignment flow: validate registry membership, allowed org, template existence, and provision repo access.
- assignment flow is now identity-first; provisioning hook is still the next step.

## Access code source

- The access code is stored in the private `student-registry` repo at `config/access-codes.json`.
- The access-request validation workflow reads it with the `STUDENT_REGISTRY_READ_TOKEN` secret.
- The reviewer notification uses the `ACCESS_REQUEST_REVIEWER_HANDLE` secret.
- The approval workflow uses `STUDENT_INTAKE_INVITE_TOKEN` to invite approved users into `student-intake`.
