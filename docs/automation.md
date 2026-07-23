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
  - Trigger: request issue opened, reopened, or edited.
  - Handles both form types in one workflow to avoid duplicate submit-triggered runs.
  - Loads access codes from the private `student-registry` repo.
  - Redacts the submitted access code immediately.
  - If invalid, comments, closes the issue, and marks it `failed`.
  - If valid, marks it `validated` and comments that staff can approve.
  - Fully automated assignment path on submission (no `approve` step).
  - Assignment submit path moves state labels from `pending` to `processing` to either `failed` or `processed`.
  - First check: issue author identity must exist in `student-registry/data/students.json`.
  - If user is not in registry, marks `failed`, comments guidance, and closes the issue.
  - If user is valid, validates org/template-prefix and creates repository from template.
  - Template derivation is trailing-hyphen-insensitive: `final` and `final-` both resolve to `final-startercode`.
  - Grants `maintain` access to the requesting student.
  - On wrong org/repo-prefix/template, fails with a broad retry message.
  - Failed requests can be retried by editing the issue fields and saving.

- `.github/workflows/process-request-label-actions.yml`
  - Single review-command workflow for access requests.
  - Access approve: staff comments `/approve` on a `validated` access request, then automation invites requester to `student-intake` and comments next steps.
  - Access reinvite: staff comments `/reinvite` on a processed access request, then automation re-sends invitation and comments instructions.
  - Assignment requests are intentionally excluded from this workflow and are handled on submit by `validate-access-request.yml`.

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

- access flow: validates access code, then invite/reinvite happens on labels.
- assignment flow: submit-driven, identity-first, template-based repo provisioning.

## Access code source

- The access code is stored in the private `student-registry` repo at `config/access-codes.json`.
- All workflows in this repository use one secret: `COURSE_AUTOMATION_TOKEN`.
