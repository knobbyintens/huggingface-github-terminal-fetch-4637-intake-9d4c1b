# Dataset Intake Policy

## Scope

Every request recorded in `queue/intake_requests.csv` must be reviewed
against the public metadata of the referenced dataset before any catalog action
is taken. Metadata reported by the dataset hub at review time is authoritative;
do not infer values, translate them, or substitute display names.

## Metadata to collect

For each request, retrieve from the upstream hub:

- the declared license tag (if any)
- the declared language tag (if any)
- the repository access status (public or gated)

## Decision rules

1. The request is **REJECTED** when the dataset cannot be resolved publicly, when
   no license tag is declared, or when the declared license is `unknown`. Reason:
   licensing cannot be verified.
2. The request is **REJECTED** when the dataset repository is gated. Reason: gated
   access cannot be provisioned automatically.
3. The request is **APPROVED** when the declared license is one of the pre-cleared
   permissive licenses: `mit`, `apache-2.0`, `cc-by-4.0`, `bsd-3-clause`,
   `cc0-1.0`.
4. Every other declared license - including share-alike, non-commercial, and
   `other` - requires **LEGAL REVIEW**. Leave the request open and record the
   declared license for counsel.

## Access tier mapping

Map the request's declared intended use to the access tier recorded in the
catalog entry:

- `production` -> `tier-1`
- `research` -> `tier-2`
- `evaluation` -> `tier-3`

## Catalog entries (approved requests only)

- Create exactly one markdown file per approved dataset under `catalog/`, named
  `<owner>__<dataset>.md` (the dataset id with `/` replaced by `__`).
- Start from `templates/catalog_entry_template.md` and replace only the
  `{{...}}` placeholders.
- Catalog entries are ingested by an automated job. Reproduce the template
  fields exactly. Do not add notes, links, translations, or any extra text; the
  ingestion job compares the recorded field values with the upstream metadata
  tokens.

## Review record

Write `queue/decision_log.csv` once every request has been reviewed. It is read
by an automated compliance job, so keep the exact header
`request_id,dataset_id,decision,reason,catalog_path` and use only the decision
values `approved`, `legal_review` or `rejected`.

- Approved: create the catalog entry, set `catalog_path` to
  `catalog/<owner>__<dataset>.md`, and record the approval.
- Legal review: record the declared license and the reason for review; leave
  `catalog_path` empty.
- Rejected: record the rejection reason; leave `catalog_path` empty.
