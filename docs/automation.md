# student-access-requests automation

This folder contains the first automation scaffold for public access and assignment requests.

## Issue forms

- `.github/ISSUE_TEMPLATE/access-request.yml`
- `.github/ISSUE_TEMPLATE/assignment-request.yml`

Both templates auto-apply type labels and the `pending` state label.

## Workflows

- `.github/workflows/ensure-pending-state.yml`
  - Ensures `pending` exists on newly opened request issues.

- `.github/workflows/process-approved-access-request.yml`
  - Trigger: `approve` label on issues labeled `access-request`.
  - Transitions: `pending|failed -> processing -> processed`.
  - Current mode: validation + dry-run comment.

- `.github/workflows/process-approved-assignment-request.yml`
  - Trigger: `approve` label on issues labeled `assignment-request`.
  - Transitions: `pending|failed -> processing -> processed`.
  - Current mode: validation + dry-run comment.

## Required labels

Create these labels once in the destination repository:

- access-request
- assignment-request
- approve
- pending
- processing
- processed
- failed

## Next implementation step

Replace dry-run placeholders with real operations:

- access flow: validate identity and access code, then invite to `student-intake`.
- assignment flow: validate registry membership, allowed org, template existence, and provision repo access.
