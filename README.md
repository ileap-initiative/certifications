# iLEAP Certification Catalog

This repository is the official catalog of iLEAP certifications. Certifications are issued by the [SINE Foundation](https://sine.foundation) with the support of [Smart Freight Centre (SFC)](https://www.smartfreightcentre.org), under the [iLEAP Certification process](https://sinefoundation.notion.site/iLEAP-Certification-Pilot-2fe407933aab80e78c67ee2495f6e1b2).

Each certification is stored as an immutable JSON file. The git history is a log of when records were added, not the source of truth for certification history — that is provided by the records themselves.

---

## Repository structure

```
certifications/    One JSON file per granted certification
revocations/       One JSON file per revoked certification
tools/             One JSON file per tool and tool provider
schema/            JSON Schema definitions for validation
examples/          Annotated example files (one per record type)
  certifications/
  revocations/
  tools/
```

---

## Certifications

Each file in `certifications/` represents a certification that was granted to a tool provider. Files are named `<provider-slug>-<date>.json` (e.g., `emisia-2026-03-05.json`).

A certification record contains:
- **`uuid`** — globally unique identifier for this record
- **`provider`** — tool provider details, including their SFC Certificate Number
- **`certification`** — [iLEAP Technical Specifications](https://specs.ileap.global) standard, version, date, validity, and Data Transaction coverage
- **`test_evidence`** — references to the ACT and PACT conformance test runs that evidence the certification, including any exceptions (skipped or non-blocking test cases) with rationale

See `examples/certifications/example.json` for a fully annotated example.

### Data Transactions

| DT | Description | Requirement |
|----|-------------|-------------|
| DT1 | ShipmentFootprint | Mandatory |
| DT2 | TOC / HOC | Mandatory |
| DT3 | TransportActivityData | Optional |
| DT4 | — | Out of scope |

DT1 and DT2 must be `tested` for a certification to be valid. The schema enforces this.

### Validity

A certification is valid for 1 year from its `certification.date`, provided the provider's SFC Certification is renewed before `provider.sfc_certification_renewal_date`. If the SFC certification lapses, the iLEAP certification is no longer valid regardless of `valid_until`.

---

## Revocations

Certifications can become invalid before their `valid_until` date — for example if a provider's SFC Certification lapses, if their implementation is found to no longer conform, or if they voluntarily withdraw. When this happens, a revocation record is added to `revocations/` rather than modifying or deleting the original certification. This preserves a complete, auditable history of all certifications ever granted and the reasons they were ended.

Files in `revocations/` are named `<provider-slug>-<revocation-date>.json` (e.g., `revocations/emisia-2026-09-15.json`). The original certification file is never modified.

A revocation record references the certification being revoked by its `uuid` and `filepath`, and includes the revocation date, reason, and optional notes.

Possible revocation reasons: `sfc_cert_lapsed` · `conformance_lost` · `voluntary_withdrawal` · `administrative`

See `examples/revocations/example.json` for a fully annotated example.

---

## Checking whether a certification is currently valid

A certification in `certifications/` is currently valid if **all** of the following hold:

1. No matching file exists in `revocations/`
2. Today's date is before `certification.valid_until`
3. The provider's SFC Certification has been renewed (check `provider.sfc_certification_renewal_date`)

---

## Adding a new certification

1. Run the iLEAP ACT tool against the provider's API with the `--certification` flag
2. Obtain the ACT run UUID from the database and the PACT run ID from the ACT output
3. Create a new JSON file in `certifications/` following the naming convention and validated against `schema/certification.json`
4. Open a pull request — a member of the iLEAP team will review and merge

## Revoking a certification

1. Create a new JSON file in `revocations/` with the same filename as the certification being revoked
2. Populate `uuid`, `filepath`, `revocation_date`, `reason`, and optionally `notes`
3. Open a pull request

---

## Schema validation

Both schemas are in `schema/` and follow [JSON Schema draft 2020-12](https://json-schema.org/draft/2020-12/schema).

To validate a file locally using [ajv-cli](https://github.com/ajv-validator/ajv-cli):

```sh
npx ajv-cli validate -s schema/certification.json -d certifications/emisia-2026-03-05.json
npx ajv-cli validate -s schema/revocation.json -d revocations/emisia-2026-03-05.json
```
