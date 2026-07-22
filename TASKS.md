# TASKS — open-cohort-catalog

> Status: Draft · Version: 0.2.0 · Last updated: 2026-06-29 · Owner: TBD (maintainer) · Lane: donated

## How these tasks map to Hee-Lee Oss

Each task below becomes a Hee-Lee Oss **Task JSON** validated against
`packages/schema/src/schemas.ts`. Field mapping:

- `id` — stable slug from the tables (e.g. `open-cohort-catalog-schema-001`).
- `title` — the table's Title.
- `project` — `open-cohort-catalog`.
- `type` — one of `code | research | writing | data | design-spec | maintenance` (per table).
- `lane` — `donated` for every task here (no funded escrow). A funded task would add `fundedBudgetUsd`.
- `priority` — `high | medium | low`.
- `domain` — array, e.g. `["cancer-research","open-science","metadata","public-data"]`.
- `riskTier` — `low | medium | high`. Access-tier/license/PII-judgement and cancer-context tasks are
  **medium**; any patient-facing educational content is **high** (oncologist + advocate sign-off).
- `urgent` — boolean; `false` for all current tasks.
- `deliverable` — `pr | dataset | document | translation`. Catalog **metadata** records (no patient
  data) → `dataset`; write-ups/checklists/explainers → `document`; code (schema/validators/UI) → `pr`.
  We **never** deliver cohort patient data (out of scope).
- `tokenEstimate` — `small | medium | large` (Size column).
- `status` — `open | in-progress | review | delivered | done`; all start `open`.
- `context`, `objective`, `acceptanceCriteria[]`, `resources[]`, `output` — per task.
- `requestor` — `TO BE SECURED` until an adopter/steward is confirmed.
- `verifiedNeed` — **`false`** until a named research group / advocacy org / repository agrees to
  adopt or steward the catalog (the general need is real; per-task delivery need is unproven).
- `outputLicense` — `CC-BY-4.0` for catalog metadata + documentation (CC0 under consideration for
  purely factual fields, per `policy-lic-035`); `MIT` for code (schema/validators/UI).

**Binding guardrail reminder (applies to every data/research task):** index **only**
open-access / aggregate / de-identified cohorts; **controlled-access (dbGaP, EGA, biobanks) and any
identifiable patient data are out of scope**; COSMIC/OncoKB are **FLAG (non-commercial)**; every
asserted field is **source-cited**; no medical advice (patient-facing = high risk, expert-reviewed).

---

## Milestone M0 — Foundation & cold-start

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
| --- | --- | --- | --- | --- | --- | --- | --- |
| open-cohort-catalog-reviewer-002 | Name/secure the Access/License/PII reviewer (blocking, non-skippable gate role) | research | small | low | document | — | Maintainer |
| open-cohort-catalog-schema-001 | Canonical cohort metadata schema + access-tier taxonomy + license-rights model | design-spec | medium | medium | document | — | Technical, Access/License/PII |
| open-cohort-catalog-gate-003 | Access-tier + license + PII triage checklist (blocking gate) | design-spec | small | medium | document | schema-001 | Access/License/PII |
| open-cohort-catalog-validator-004 | Schema validator + citation/link resolver (golden fixtures, CI) | code | medium | low | pr | schema-001 | Technical |
| open-cohort-catalog-exclude-005 | Seed the excluded-resources register (dbGaP/EGA/biobanks, pointer-only) | writing | small | medium | document | gate-003 | Access/License/PII |
| open-cohort-catalog-boundary-027 | Sibling boundary + shared access-tier vocabulary / schema-contract doc | design-spec | small | low | document | schema-001 | Maintainer |
| open-cohort-catalog-outreach-006 | Adopter/steward outreach + candidate cohort shortlist | research | small | low | document | — | Steward |
| open-cohort-catalog-pilot-007 | End-to-end index one pilot open cohort (DepMap or GDC open-tier TCGA) | data | medium | medium | dataset | schema-001, gate-003, validator-004, reviewer-002 | Access/License/PII, Technical |

**Acceptance criteria — key tasks**

- **schema-001 (canonical schema + taxonomy + rights model)**
  - [ ] JSON Schema + TS types define every required field from PLAN §6 (id, name, **entryType**,
        **identityKeys**, custodian, **primaryHostingRepository, hostingLocations[]**, cancerTypes[]
        with OncoTree/NCIt/ICD-O-3, aggregateCounts, **generationContext**, assayTypes[], accessTier,
        license{...rights booleans}, accessTerms, deIdentification, provenance, fieldProvenance[],
        governanceFlags[], crossRefs[], **recordState**, completenessScore).
  - [ ] **`entryType` is an enum** `patient-cohort | population-statistics | cell-model-collection |
        molecular-dataset`, and **`aggregateCounts` shape is conditional on it** (cell-model →
        cellLines/models; population-statistics → statisticalUniverse; never forces a "patients" count
        on a non-patient entry).
  - [ ] **Cross-repository identity / dedup is enforced**: `identityKeys` (primary custodian +
        program/accession + generation scope) yield **one record per distinct cohort**; the same cohort
        in multiple portals is recorded via `hostingLocations[] {repository, url, isMirror}`, never as
        multiple records.
  - [ ] `accessTier` is an enum `OPEN | OPEN-AGGREGATE | REGISTERED-OPEN | NON-COMMERCIAL | CONTROLLED`;
        only the first three are valid in a *published* record (CONTROLLED/NON-COMMERCIAL rejected by
        the publish linter).
  - [ ] Each `license` rights boolean (permitsReuse/permitsDerivatives/commercialUseAllowed/
        redistributionAllowed/attributionRequired/shareAlike) is required and must reference a
        `fieldProvenance` entry; "unknown" is never serialized as `true`. **`redistributionAllowed:
        false` is a valid index entry** (index ≠ redistribute) and must not be rejected on that basis.
  - [ ] `recordState` enum `current | termsChangedSinceSnapshot | underReverification` exists as a
        first-class field for drift (set by drift-017, not just a maintenance ticket).
  - [ ] Schema forbids any per-patient / row-level / quasi-identifier field — catalog is cohort-level only.
  - [ ] Output licensed CC-BY-4.0 (metadata model doc); commit DCO signed-off.

- **gate-003 (access-tier + license + PII gate)**
  - [ ] Checklist encodes the access-tier decision table (PLAN §7) and the objective indexable criterion
        (accessTier ∈ {OPEN, OPEN-AGGREGATE, REGISTERED-OPEN} AND permitsReuse true AND derivative-OK
        AND de-identification publisher-documented).
  - [ ] License step requires recording id + URL + snapshot reference (committed copy + SHA-256 +
        Wayback URL) and each rights boolean with a cited clause; missing/unparseable = FLAG/EXCLUDE.
  - [ ] PII/identifiability step: confirm de-identified/aggregate + that only cohort-level descriptors
        are extracted; halt-and-discard + EXCLUDE on any identifiable/controlled signal; never
        de-identify/link/aggregate ourselves.
  - [ ] NC/custom sources (COSMIC, OncoKB) routed to FLAG → `policy-nc-034`, not default-included.
  - [ ] Produces a committed, reviewable PASS/FLAG/EXCLUDE artifact per cohort (`gates/<cohort-id>.json`)
        recording which checks ran and what fired.

- **validator-004 (schema validator + citation resolver)**
  - [ ] Validates records against `schema-001`; rejects records with a rights boolean lacking
        `fieldProvenance`, with `accessTier: CONTROLLED`/`NON-COMMERCIAL` in a published record, or with
        an `OPEN` tier carrying `duaRequired: true`.
  - [ ] Citation resolver checks every `fieldProvenance.sourceUrl` and `license.url` resolves
        (mocked in CI); dead/missing citations fail.
  - [ ] Ships committed **synthetic-only** golden fixtures (valid pass + malformed fail); no real
        cohort data in fixtures. `pnpm build && pnpm test && pnpm lint` green; MIT; DCO signed-off.

- **pilot-007 (pilot cohort, end-to-end)**
  - [ ] Pilot is a clearly-open, **no-patient-data** resource (DepMap CC-BY-4.0, or a GDC open-tier
        TCGA descriptor) so a real verified record is reachable without a third-party dependency.
  - [ ] Cohort passed `gate-003` (access tier OPEN; license permits reuse + derivative metadata; no
        identifiable/controlled data) with `gates/<cohort-id>.json` committed.
  - [ ] Complete canonical record authored; **every** field carries a `fieldProvenance` citation;
        completeness score recorded (target ≥ 90/100).
  - [ ] License snapshot recorded (committed copy + SHA-256 + Wayback URL); provenance complete
        (source URL, custodian, retrieval date, version/freeze date, attribution string).
  - [ ] Validator + citation resolver green in CI; `outputLicense: CC-BY-4.0`; deliverable is metadata,
        never the cohort data.

- **boundary-027 (sibling boundary + shared vocabulary contract)**
  - [ ] Written boundary differentiating this project (breadth / access-tier + license layer) from
        `cancer-dataset-datasheets` (depth), `ewing-open-data-catalog` (disease vertical), and
        `cancer-data-dictionaries` (variable-level), with the explicit rule that the siblings **consume**
        this project's access-tier vocabulary + schema contract rather than inventing their own.
  - [ ] Defines the shared access-tier enum + record-`id` reference contract the siblings reuse;
        `outputLicense: CC-BY-4.0`; DCO signed-off.

**M0 Definition of Done:** Access/License/PII reviewer named (blocking role filled before pilot
review); canonical schema (incl. `entryType`, identity/dedup, conditional `aggregateCounts`,
`recordState`) + access-tier taxonomy + license-rights model + gate checklist published; sibling
boundary + shared-vocabulary contract drafted; validator + citation resolver green in CI with synthetic
golden fixtures; excluded-resources register seeded with the controlled set (incl. TCGA-controlled/
germline, GTEx, TOPMed); **one** pilot open cohort fully indexed end-to-end (gate-passed, fully cited,
completeness ≥ 90/100); ≥ 1 adopter/steward outreach thread opened.

---

## Milestone M1 — Gate hardened + first real coverage

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
| --- | --- | --- | --- | --- | --- | --- | --- |
| open-cohort-catalog-snapshot-008 | License/terms snapshot capture in the provenance flow | code | small | low | pr | schema-001, gate-003 | Technical |
| open-cohort-catalog-triage-009 | Triage 10 candidate cohorts through the gate | research | medium | medium | document | gate-003, policy-nc-034, outreach-006 | Access/License/PII |
| open-cohort-catalog-policy-nc-034 | Non-commercial/custom-license acceptance policy (COSMIC/OncoKB) | design-spec | small | medium | document | gate-003 | Access/License/PII |
| open-cohort-catalog-index-010 | Index cohort set #1 — GDC open tier (TCGA + TARGET + CPTAC open) | data | large | medium | dataset | pilot-007, triage-009 | Access/License/PII, Technical, Oncology-data |
| open-cohort-catalog-index-011 | Index cohort set #2 — GEO + EMBL-EBI (Expression Atlas / PRIDE) | data | large | medium | dataset | pilot-007, triage-009 | Access/License/PII, Technical, Oncology-data |
| open-cohort-catalog-adopter-012 | Secure first confirmed adopter/steward | research | small | low | document | outreach-006 | Steward |

**Acceptance criteria — key tasks**

- **policy-nc-034 (NC/custom-license policy)**
  - [ ] Written rule for COSMIC, OncoKB, and any NC/custom-terms source: include-with-prominent-flag
        vs pointer-only, with rationale and a recorded governance decision; default = do not auto-include.
  - [ ] Specifies the `NON-COMMERCIAL` UI badge and the `governanceFlags` value used.
  - [ ] **Must be decided/merged before `triage-009` runs** so triage applies a fixed rule.

- **triage-009 (triage 10 candidates)**
  - [ ] Ten **distinct** cohorts (deduped per the §6 identity rule — a cohort appearing in multiple
        portals counts once) evaluated with a committed `gates/<cohort-id>.json` each (PASS/FLAG/EXCLUDE
        + rationale), applying the access-tier table and `policy-nc-034`. Candidate pool includes
        cBioPortal public studies (deduped against GDC/Synapse appearances).
  - [ ] Each PASS records license id/URL/snapshot and the rights booleans with cited clauses; each
        FLAG/EXCLUDE records the disqualifying reason (controlled tier, undocumented de-id, NC terms).
  - [ ] **Every `REGISTERED-OPEN` or `mixed-tier` candidate gets a mandatory second Access/License/PII
        sign-off** (100%, not sampled) before PASS.
  - [ ] Any controlled/identifiable candidate is EXCLUDED and routed to the excluded-resources register;
        no controlled resource is given a record.

- **index-010 / index-011 (first cohort sets)**
  - [ ] Each cohort gate-passed (open tier only for mixed-tier resources, with `mixed-tier` flag);
        record fully cited (every field `fieldProvenance`); completeness ≥ 90/100; schema-valid; CI green.
  - [ ] Cancer types coded (OncoTree primary; NCIt/ICD-O-3 cross-refs); oncology-data reviewer signs off
        on coding and de-identification basis.
  - [ ] `outputLicense: CC-BY-4.0`; deliverable is metadata records, never cohort data.

- **adopter-012 (first confirmed adopter/steward)**
  - [ ] A named research group / advocacy org / repository confirms it will use, host, or steward the
        catalog; contribution/hosting mechanism documented.
  - [ ] Tasks for that adopter updated to `verifiedNeed: true` with `requestor` set.

**M1 Definition of Done:** gate applied to ≥ 10 cohorts with committed artifacts; ≥ 10 cohorts
published (gate-passed, fully cited) across ≥ 3 repositories; NC policy decided and COSMIC/OncoKB
dispositioned; license-snapshot capture working (committed copy + SHA-256 + Wayback); ≥ 1 confirmed
adopter/steward **or** an honest "not yet secured" with blockers surfaced.

---

## Milestone M2 — Searchable interface + scale

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
| --- | --- | --- | --- | --- | --- | --- | --- |
| open-cohort-catalog-ui-013 | Static, dataless search/browse UI (faceted; metadata-only) | code | large | low | pr | schema-001, index-010, index-011 | Technical |
| open-cohort-catalog-ci-linter-014 | CI publish-gate linter (no CONTROLLED/NC; rights cited; tier-consistent) | code | small | medium | pr | schema-001, validator-004 | Technical, Access/License/PII |
| open-cohort-catalog-index-015 | Index cohort set #3 — DepMap + Cell Model Passports + ICGC/PCAWG open | data | large | medium | dataset | triage-009, ci-linter-014 | Access/License/PII, Technical, Oncology-data |
| open-cohort-catalog-index-016 | Index cohort set #4 — aggregate stats (SEER\*Explorer, GLOBOCAN, CDC WONDER) | data | medium | medium | dataset | triage-009, ci-linter-014 | Access/License/PII, Oncology-data |
| open-cohort-catalog-drift-017 | Staleness/drift detection + freshness SLA | maintenance | small | low | pr | validator-004, snapshot-008 | Maintainer |

**Acceptance criteria — key tasks**

- **ui-013 (static search UI)**
  - [ ] Client-side static site over the catalog JSON; **no backend store, no tracking, no patient
        data**; faceted by cancer type / repository / access tier / license.
  - [ ] `REGISTERED-OPEN` and `NON-COMMERCIAL` (if any) carry a prominent access badge; CONTROLLED
        never appears (the catalog has no such records).
  - [ ] MIT; `pnpm build && pnpm test && pnpm lint` green; DCO signed-off.

- **ci-linter-014 (publish-gate linter)**
  - [ ] CI **blocks merge** of any record with `accessTier: CONTROLLED`/`NON-COMMERCIAL` (unless a
        governance decision ref is attached per `policy-nc-034`), any rights boolean without
        `fieldProvenance`, any `OPEN` record with `duaRequired: true`, or any unresolved citation.
  - [ ] **Does not** reject a record merely for `license.redistributionAllowed: false` (index ≠
        redistribute; e.g. GENIE is a valid `REGISTERED-OPEN` index entry); a golden fixture asserts this.
  - [ ] Encodes the access-tier consistency rules as tests with synthetic fixtures; MIT; CI green.

- **index-016 (aggregate-stats cohorts)**
  - [ ] `entryType: population-statistics`; only **open aggregate** layers indexed (SEER\*Explorer
        aggregate, GLOBOCAN, CDC WONDER); SEER research microdata explicitly EXCLUDED (signed-agreement /
        controlled) and noted in the record.
  - [ ] `accessTier: OPEN-AGGREGATE`; each statistic-source field cited; **IARC GLOBOCAN terms verified
        (attribution + use conditions), not assumed open**; SEER terms verified.

- **drift-017 (staleness/drift detection + freshness SLA)**
  - [ ] Stores a **content hash of each record's terms URL** and re-checks on the freshness SLA; a
        changed hash flips the record to the first-class **`recordState: termsChangedSinceSnapshot`**
        and routes it to human re-verification (not a silent maintenance ticket).
  - [ ] Every published record carries a freeze date + refresh window; MIT; CI green.

**M2 Definition of Done:** static search UI live (metadata-only, faceted incl. `entryType`, no
tracking); ≥ 40 **distinct** cohorts (deduped per the §6 identity rule) published cumulatively across
≥ 5 repositories **and** meeting the coverage-representativeness target (≥ 30% rare-cancer /
pediatric-AYA / non-US; ≥ 3 modalities); CI publish-gate linter blocks access-tier violations while
allowing `redistributionAllowed: false` entries; staleness/drift detection running (terms-URL content
hash + `termsChangedSinceSnapshot` state) with a freshness SLA on every record.

---

## Milestone M3 — Adoption, reuse outcomes & sustainability (+ optional patient layer)

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
| --- | --- | --- | --- | --- | --- | --- | --- |
| open-cohort-catalog-reuse-018 | Track and verify external reuse/adoption events | research | small | low | document | index-010, index-011, index-015, adopter-012 | Steward |
| open-cohort-catalog-maintain-019 | Refresh/maintenance process + steward handover | maintenance | small | low | document | drift-017 | Maintainer, Steward |
| open-cohort-catalog-patient-explainer-020 | Education-only "how to read this catalog / access tiers" explainer (HIGH RISK) | writing | medium | high | document | adopter-012 | Oncologist, Patient-advocate, Maintainer |

**Acceptance criteria — key tasks**

- **reuse-018 (reuse/adoption tracking)**
  - [ ] ≥ 3 verifiable external reuse events recorded in `outcomes/*.json` (citation / linking page /
        fork-PR / documented query from a named group); each links to external evidence (no self-report).

- **patient-explainer-020 (education-only patient/advocate layer — HIGH RISK)**
  - [ ] Content is **education only** — explains what "open cohort," "access tier," "de-identified,"
        and "license" mean; contains **no** medical advice, prognosis, or treatment guidance.
  - [ ] Every page carries an explicit "**This is not medical advice**" notice.
  - [ ] **Both** an oncologist **and** a patient-advocate review and sign off before publish; sign-offs
        recorded. If either reviewer is not secured, the task is **deferred** (not shipped).
  - [ ] `riskTier: high`; `outputLicense: CC-BY-4.0`.

**M3 Definition of Done:** ≥ 3 verifiable external reuse events; ≥ 1 confirmed adopter/steward for
ongoing maintenance; documented refresh process with freshness SLAs; the patient explainer is either
shipped with oncologist **and** advocate sign-offs + "not medical advice" framing, or explicitly
deferred with the reviewer-gap surfaced (never shipped unreviewed).

---

## Backlog / future

| ID | Title | Type | Size | Risk | Deliverable | Notes |
| --- | --- | --- | --- | --- | --- | --- |
| open-cohort-catalog-genie-021 | Index AACR Project GENIE (REGISTERED-OPEN) with registration-step documentation | data | medium | medium | dataset | Exercises the registered-open tier + badge |
| open-cohort-catalog-cosmic-022 | COSMIC disposition record (NON-COMMERCIAL flag per policy-nc-034) | data | small | medium | document | Pointer/flag only unless governance includes it |
| open-cohort-catalog-oncokb-023 | OncoKB disposition record (NON-COMMERCIAL flag per policy-nc-034) | data | small | medium | document | Pointer/flag only unless governance includes it |
| open-cohort-catalog-dcat-024 | DCAT / schema.org Dataset export of the catalog | code | medium | low | pr | Interop with dataset-discovery ecosystems |
| open-cohort-catalog-policy-lic-035 | Catalog-metadata license decision (CC-BY-4.0 vs CC0 for factual fields) | design-spec | small | low | document | Resolves Open Question; precedes broad publishing |
| open-cohort-catalog-api-025 | Read-only static JSON API over the catalog (no patient data) | code | medium | low | pr | Programmatic access for adopters |
| open-cohort-catalog-i18n-026 | Translate the patient explainer (after high-risk sign-off) | translation | small | high | translation | Only after patient-explainer-020 signed off |

---

## Example task JSON

```json
{
  "id": "open-cohort-catalog-schema-001",
  "title": "Canonical cohort metadata schema + access-tier taxonomy + license-rights model",
  "project": "open-cohort-catalog",
  "type": "design-spec",
  "lane": "donated",
  "priority": "high",
  "domain": ["cancer-research", "open-science", "metadata", "public-data"],
  "riskTier": "medium",
  "urgent": false,
  "deliverable": "document",
  "tokenEstimate": "medium",
  "status": "open",
  "context": "Open cancer cohorts are scattered across GDC, GEO, EMBL-EBI, DepMap, SEER, IARC and more, with inconsistent access terms and heterogeneous licenses, and the open-vs-controlled boundary (e.g. TCGA has both tiers) is routinely misunderstood. Before indexing any cohort, the project needs one canonical metadata schema whose access tier and license-rights breakdown are first-class, required, gate-controlled fields, and which is cohort-level only (no per-patient data). Binding guardrail: index only open-access/aggregate/de-identified cohorts; controlled-access (dbGaP, EGA, biobanks) and any identifiable patient data are out of scope; COSMIC/OncoKB are non-commercial and flagged; every field must be source-cited.",
  "objective": "Define the canonical cohort metadata schema (JSON Schema + TypeScript types), the typed entryType (patient-cohort | population-statistics | cell-model-collection | molecular-dataset) with conditional aggregateCounts, the cross-repository cohort-identity/dedup rule, the access-tier taxonomy (OPEN | OPEN-AGGREGATE | REGISTERED-OPEN | NON-COMMERCIAL | CONTROLLED), and the license-rights model that every per-cohort indexing task will populate.",
  "acceptanceCriteria": [
    "JSON Schema + TS types define every required field: id, name, entryType, identityKeys {primaryCustodian, programOrAccession, generationScope}, custodian, primaryHostingRepository, hostingLocations[] {repository, url, isMirror}, cancerTypes[] (OncoTree primary; NCIt/ICD-O-3 cross-refs), population, aggregateCounts (aggregate only; shape conditional on entryType), generationContext {funder?, consentBasis?, dataGenerationDateRange?}, assayTypes[], dataModality[], accessTier, license {id, url, permitsReuse, permitsDerivatives, commercialUseAllowed, redistributionAllowed, attributionRequired, shareAlike, snapshotRef}, accessTerms {registrationRequired, clickThrough, duaRequired, irbRequired, citationRequirement}, deIdentification {level, standard, publisherDocumented, notes}, provenance {sourceUrl, retrievedAt, version, custodianContact, attribution}, fieldProvenance[] {field, claim, sourceUrl, retrievedAt}, governanceFlags[], crossRefs[], recordState, completenessScore.",
    "entryType is an enum and aggregateCounts shape is conditional on it (cell-model-collection -> cellLines/models; population-statistics -> statisticalUniverse; no forced 'patients' count on a non-patient entry).",
    "Cross-repository identity is enforced: one record per distinct cohort keyed on identityKeys; the same cohort in multiple portals is recorded via hostingLocations[], never as multiple records.",
    "accessTier is an enum and only OPEN | OPEN-AGGREGATE | REGISTERED-OPEN are valid in a published record; CONTROLLED and NON-COMMERCIAL are rejected by the publish linter.",
    "Every license rights boolean is required and must reference a fieldProvenance entry; an unverified value is never serialized as true; redistributionAllowed: false is a valid index entry (index != redistribute).",
    "Schema structurally forbids per-patient / row-level / quasi-identifier fields (cohort-level descriptors only) so no identifiable data can ever be represented.",
    "Mixed-tier resources are representable only for their open tier via a governanceFlags value (mixed-tier-open-portion-only).",
    "Document and schema are licensed CC-BY-4.0; any committed tooling passes pnpm build && pnpm test && pnpm lint; commit is DCO signed-off."
  ],
  "resources": [
    "C:\\code\\hee-lee-oss\\planning\\projects\\open-cohort-catalog\\PLAN.md",
    "C:\\code\\hee-lee-oss\\planning\\ROADMAP.md",
    "C:\\code\\hee-lee-oss\\packages\\schema\\src\\schemas.ts",
    "NIH Genomic Data Sharing Policy; NCI Genomic Data Commons access tiers",
    "SPDX license list; OncoTree / NCIt / ICD-O-3 vocabularies; schema.org/Dataset; DCAT"
  ],
  "output": "A canonical cohort metadata schema (JSON Schema + TypeScript types) plus the access-tier taxonomy and license-rights model definition, committed to the project repo and ready for reuse by the gate checklist and every per-cohort indexing task.",
  "requestor": "TO BE SECURED",
  "verifiedNeed": false,
  "outputLicense": "CC-BY-4.0"
}
```

---

## Task count rollup

- **22 scheduled tasks** across M0–M3 (M0: 8 · M1: 6 · M2: 5 · M3: 3) and **7 backlog tasks** = **29 total**.
- By type: design-spec 5 · code 6 · data 8 · research 5 · writing 2 · maintenance 2 · translation 1.
- By risk: **high 2** (patient explainer + its translation) · **medium 14** · **low 13**.
- Every data/research task requires its own committed `gates/<cohort-id>.json` before work proceeds;
  no controlled/identifiable resource is ever given a record (register-pointer only).
- All tasks: `lane: donated`, `verifiedNeed: false`, `requestor: TO BE SECURED` until an adopter/steward
  is confirmed (`adopter-012`).

---

## Generated task index

> Auto-generated by Hee-Lee Oss task-decomposition on 2026-06-29. All 29 tasks validated against the Hee-Lee Oss
> task schema (packages/schema). Seed task `schema-001` was pre-existing; 28 new task files generated.

| File | Title | Type | Priority | Risk | Deliverable | Status |
| --- | --- | --- | --- | --- | --- | --- |
| [open-cohort-catalog-schema-001.json](tasks/open-cohort-catalog-schema-001.json) | Canonical cohort metadata schema + access-tier taxonomy + license-rights model | design-spec | high | medium | document | open |
| [open-cohort-catalog-reviewer-002.json](tasks/open-cohort-catalog-reviewer-002.json) | Name/secure the Access/License/PII reviewer (blocking, non-skippable gate role) | research | high | low | document | open |
| [open-cohort-catalog-gate-003.json](tasks/open-cohort-catalog-gate-003.json) | Access-tier + license + PII triage checklist (blocking gate) | design-spec | high | medium | document | open |
| [open-cohort-catalog-validator-004.json](tasks/open-cohort-catalog-validator-004.json) | Schema validator + citation/link resolver (golden fixtures, CI) | code | high | low | pr | open |
| [open-cohort-catalog-exclude-005.json](tasks/open-cohort-catalog-exclude-005.json) | Seed the excluded-resources register (dbGaP/EGA/biobanks, pointer-only) | writing | medium | medium | document | open |
| [open-cohort-catalog-outreach-006.json](tasks/open-cohort-catalog-outreach-006.json) | Adopter/steward outreach + candidate cohort shortlist | research | medium | low | document | open |
| [open-cohort-catalog-pilot-007.json](tasks/open-cohort-catalog-pilot-007.json) | End-to-end index one pilot open cohort (DepMap or GDC open-tier TCGA) | data | high | medium | dataset | open |
| [open-cohort-catalog-snapshot-008.json](tasks/open-cohort-catalog-snapshot-008.json) | License/terms snapshot capture in the provenance flow | code | medium | low | pr | open |
| [open-cohort-catalog-triage-009.json](tasks/open-cohort-catalog-triage-009.json) | Triage 10 candidate cohorts through the gate | research | medium | medium | document | open |
| [open-cohort-catalog-index-010.json](tasks/open-cohort-catalog-index-010.json) | Index cohort set #1 — GDC open tier (TCGA + TARGET + CPTAC open) | data | medium | medium | dataset | open |
| [open-cohort-catalog-index-011.json](tasks/open-cohort-catalog-index-011.json) | Index cohort set #2 — GEO + EMBL-EBI (Expression Atlas / PRIDE) | data | medium | medium | dataset | open |
| [open-cohort-catalog-adopter-012.json](tasks/open-cohort-catalog-adopter-012.json) | Secure first confirmed adopter/steward | research | medium | low | document | open |
| [open-cohort-catalog-ui-013.json](tasks/open-cohort-catalog-ui-013.json) | Static, dataless search/browse UI (faceted; metadata-only) | code | medium | low | pr | open |
| [open-cohort-catalog-ci-linter-014.json](tasks/open-cohort-catalog-ci-linter-014.json) | CI publish-gate linter (no CONTROLLED/NC; rights cited; tier-consistent) | code | medium | medium | pr | open |
| [open-cohort-catalog-index-015.json](tasks/open-cohort-catalog-index-015.json) | Index cohort set #3 — DepMap + Cell Model Passports + ICGC/PCAWG open | data | medium | medium | dataset | open |
| [open-cohort-catalog-index-016.json](tasks/open-cohort-catalog-index-016.json) | Index cohort set #4 — aggregate stats (SEER*Explorer, GLOBOCAN, CDC WONDER) | data | medium | medium | dataset | open |
| [open-cohort-catalog-drift-017.json](tasks/open-cohort-catalog-drift-017.json) | Staleness/drift detection + freshness SLA | maintenance | medium | low | pr | open |
| [open-cohort-catalog-reuse-018.json](tasks/open-cohort-catalog-reuse-018.json) | Track and verify external reuse/adoption events | research | low | low | document | open |
| [open-cohort-catalog-maintain-019.json](tasks/open-cohort-catalog-maintain-019.json) | Refresh/maintenance process + steward handover | maintenance | low | low | document | open |
| [open-cohort-catalog-patient-explainer-020.json](tasks/open-cohort-catalog-patient-explainer-020.json) | Education-only 'how to read this catalog / access tiers' explainer (HIGH RISK) | writing | low | high | document | open |
| [open-cohort-catalog-genie-021.json](tasks/open-cohort-catalog-genie-021.json) | Index AACR Project GENIE (REGISTERED-OPEN) with registration-step documentation | data | low | medium | dataset | open |
| [open-cohort-catalog-cosmic-022.json](tasks/open-cohort-catalog-cosmic-022.json) | COSMIC disposition record (NON-COMMERCIAL flag per policy-nc-034) | data | low | medium | document | open |
| [open-cohort-catalog-oncokb-023.json](tasks/open-cohort-catalog-oncokb-023.json) | OncoKB disposition record (NON-COMMERCIAL flag per policy-nc-034) | data | low | medium | document | open |
| [open-cohort-catalog-dcat-024.json](tasks/open-cohort-catalog-dcat-024.json) | DCAT / schema.org Dataset export of the catalog | code | low | low | pr | open |
| [open-cohort-catalog-api-025.json](tasks/open-cohort-catalog-api-025.json) | Read-only static JSON API over the catalog (no patient data) | code | low | low | pr | open |
| [open-cohort-catalog-i18n-026.json](tasks/open-cohort-catalog-i18n-026.json) | Translate the patient explainer (after high-risk sign-off) | writing | low | high | translation | open |
| [open-cohort-catalog-boundary-027.json](tasks/open-cohort-catalog-boundary-027.json) | Sibling boundary + shared access-tier vocabulary / schema-contract doc | design-spec | medium | low | document | open |
| [open-cohort-catalog-policy-nc-034.json](tasks/open-cohort-catalog-policy-nc-034.json) | Non-commercial/custom-license acceptance policy (COSMIC/OncoKB) | design-spec | high | medium | document | open |
| [open-cohort-catalog-policy-lic-035.json](tasks/open-cohort-catalog-policy-lic-035.json) | Catalog-metadata license decision (CC-BY-4.0 vs CC0 for factual fields) | design-spec | medium | low | document | open |

**Fan-out:** none (all tasks are single-instance; i18n-026 is not fanned out per-language as no explicit language list is enumerated in TASKS.md).

**Validation:** all 29 task files pass the Hee-Lee Oss task schema validator (`node validate-tasks.mjs tasks/`).

**Guardrail notes:**
- `patient-explainer-020` and `i18n-026` are `riskTier: high` and must not ship without oncologist + patient-advocate sign-offs.
- No controlled/identifiable data is ever given a catalog record; controlled resources (dbGaP, EGA, TCGA controlled tier, GTEx, TOPMed, biobanks) appear only in the excluded-resources register (`exclude-005`).
- `cosmic-022` and `oncokb-023` are disposition documents only, not catalog records — pending the governance decision in `policy-nc-034`.
- All tasks carry `verifiedNeed: false` and `requestor: TO BE SECURED` until `adopter-012` is confirmed.
