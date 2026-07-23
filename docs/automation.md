# student-access-requests automation

This folder contains the first automation scaffold for public access and assignment requests.

## Issue forms

- `.github/ISSUE_TEMPLATE/access-request.yml`
- `.github/ISSUE_TEMPLATE/assignment-request.yml`

- Both templates auto-apply type labels and the `pending` state label.

## Workflows

- `.github/workflows/ensure-pending-state.yml`
  - Ensures `pending` exists on newly opened request issues.

- `.github/workflows/validate-access-request.yml`
  - Trigger: access request issue opened, reopened, or edited.
  - Loads access codes from the private `student-registry` repo.
  - Redacts the submitted access code immediately.
  - If invalid, comments, closes the issue, and marks it `failed`.
  - If valid, marks it `validated` and notifies the reviewer handle.

- `.github/workflows/validate-assignment-request.yml`
  - Trigger: assignment request issue opened or reopened.
  - Validates required fields (`Organization`, `Repository name`).
  - If invalid, marks `failed`, comments guidance, and closes the issue.
  - If valid, marks `validated` and keeps the request ready for `approve`.

- `.github/workflows/process-request-label-actions.yml`
  - Single label-action workflow for `approve` and `reinvite`.
  - Access approve: requires `validated`, invites requester to `student-intake`, and comments next steps.
  - Access reinvite: re-sends invitation and comments instructions.
  - Assignment approve: requires `assignment-request` + `validated`, then first check is issue-author identity in `student-registry/data/students.json`.
  - If author is not registered, processing stops immediately.
  - For assignment requests, template repository is derived automatically from requested repository name by appending `startercode`.
  - This consolidation prevents multi-workflow label fan-out and keeps processing to one workflow run per label action.

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
- The label-action workflow uses `STUDENT_INTAKE_INVITE_TOKEN` for access invite/reinvite routes.
