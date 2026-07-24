# student-access-requests automation

This folder contains the first automation scaffold for public access and assignment requests.

## Issue forms

- `.github/ISSUE_TEMPLATE/access-request.yml`
- `.github/ISSUE_TEMPLATE/assignment-request.yml`

- Access template auto-applies type labels and the `pending` state label.
- Assignment template auto-applies `assignment-request` and `pending` immediately on submit.

## Workflows

- `.github/workflows/ensure-pending-state.yml`
  - Ensures `pending` exists on newly opened request issues.

- `.github/workflows/validate-access-request.yml`
  - Trigger: request issue opened.
  - Handles both form types in one workflow to avoid duplicate submit-triggered runs.
  - Loads access codes from the private `student-registry` repo.
  - Redacts the submitted access code immediately.
  - If invalid, comments, closes the issue, and marks it `failed`.
  - If valid, marks it `validated` and leaves the issue open for manual staff processing.
  - Fully automated assignment path on submission (no `approve` step).
  - Assignment submit path moves state labels from `pending` to `processing` to either `failed` or `processed`.
  - First check: issue author identity must exist in `student-registry/data/students.json`.
  - If user is not in registry, marks `failed`, comments guidance, and closes the issue.
  - If user is valid, validates org/template-prefix and creates repository from template.
  - Template derivation is trailing-hyphen-insensitive: `final` and `final-` both resolve to `final-startercode`.
  - Grants `maintain` access to the requesting student.
  - On wrong org/repo-prefix/template, fails with a broad retry message.
  - Failed requests are retried by submitting a new issue.

- `.github/workflows/process-request-label-actions.yml`
  - Manual `workflow_dispatch` workflow for access requests.
  - Processes all open `access-request` issues with the `validated` label.
  - Approves each by inviting requester to `student-intake`, posting next steps, and moving labels to `processed`.
  - Assignment requests are excluded from this workflow and are handled on submit by `validate-access-request.yml`.

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

Current implementation status:

- access flow: validates access code on issue open, then staff runs a manual workflow to process validated requests.
- assignment flow: submit-driven, identity-first, template-based repo provisioning.

## Access code source

- The access code is stored in the private `student-registry` repo at `config/access-codes.json`.
- All workflows in this repository use one secret: `COURSE_AUTOMATION_TOKEN`.
