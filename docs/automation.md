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
  - Trigger: `approve` label on issues labeled `access-request` after validation.
  - Requires the `validated` label before continuing.
  - Current mode: approval acknowledgement comment.

- `.github/workflows/process-approved-assignment-request.yml`
  - Trigger: `approve` label on issues labeled `assignment-request`.
  - Transitions: `pending|failed -> processing -> processed`.
  - Current mode: validation + dry-run comment.

## Required labels

Create these labels once in the destination repository:

- access-request
- pending
- validated
- failed
- approve
- assignment-request
- processing
- processed

## Next implementation step

Replace dry-run placeholders with real operations:

- access flow: validate identity and access code, then invite to `student-intake`.
- assignment flow: validate registry membership, allowed org, template existence, and provision repo access.

## Access code source

- The access code is stored in the private `student-registry` repo at `config/access-codes.json`.
- The access-request validation workflow reads it with the `STUDENT_REGISTRY_READ_TOKEN` secret.
- The reviewer notification uses the `ACCESS_REQUEST_REVIEWER_HANDLE` secret.
