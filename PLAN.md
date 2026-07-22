# PLAN — open-cohort-catalog

> Status: Draft · Version: 0.2.0 · Last updated: 2026-06-29 · Owner: TBD (maintainer) ·
> Lane: donated · Track: 8b — Cancer research, broadly · Risk tier: **medium** (core catalog) /
> **high** (any patient-facing educational layer)

## Executive summary

Cancer research generates an enormous, growing body of **open** data — gene-expression series,
open-tier somatic mutation calls, proteogenomics, cell-line dependency maps, and aggregate
incidence/outcome statistics — scattered across dozens of repositories (GDC, GEO, EMBL-EBI,
PRIDE, DepMap, SEER, IARC, and more). The practical problem is **not** that the data is missing;
it is that a researcher (or an informed patient, advocate, or student) cannot easily answer four
questions about any given cohort: *What is in it? Who can use it, and under what license? Is it
truly open, or is it controlled-access behind an application? How do I cite it and trust its
provenance?* The access terms are inconsistent, the licenses are heterogeneous (public-domain US
government data sits next to non-commercial custom terms and click-through registration), and the
boundary between **open-access** and **controlled-access** tiers *within the same resource* (e.g.
TCGA has both) is routinely misunderstood — a misunderstanding that, at worst, leads people to
treat identifiable or controlled data as if it were freely reusable.

This project builds a **searchable index (catalog) of open cancer cohorts and their access
terms/licenses** — a metadata layer *about* cohorts, never the patient data itself. Each catalog
entry is a structured, **fully source-cited** record: cancer type(s), aggregate sample/patient
counts, assay modalities, hosting repository, **access tier**, **license** (with an explicit
permits-reuse / permits-derivatives / commercial-use / redistribution breakdown), de-identification
basis, citation requirement, and provenance. The deliverable is the **catalog + a small search
interface + the verification tooling**, contributed back as open artifacts. The headline,
non-negotiable design constraint is the **cancer data guardrail (see §7)**: the catalog indexes
**only open-access / aggregate / de-identified** cohorts; **controlled-access** resources (dbGaP,
EGA, individual-level biobanks) and **any identifiable patient data** are **out of scope** and are
never ingested, mirrored, or described at the record level — they are recorded only as a short
"excluded / controlled — see custodian" pointer so users are not misled into thinking they are open.

Risk tier is **medium** for the researcher-facing catalog. The dominant risks are not engineering
risks; they are **factual, legal, and ethical**: mis-stating a cohort's access tier (the most
dangerous error — implying controlled or identifiable data is open), mis-classifying a license
(treating COSMIC/OncoKB non-commercial terms as freely reusable), or letting any patient-level or
identifiable datum into a "metadata-only" catalog. Any **patient-facing** educational layer is
treated as **high risk** and gated behind **oncologist + patient-advocate review** with an explicit
"not medical advice" frame. The plan front-loads the access-tier + license + PII gate as a hard,
non-skippable, per-cohort blocking step, and treats provenance-on-every-assertion as a build
requirement rather than a nicety.

## Problem & beneficiaries

**Who is helped.**
- **Cancer researchers and bioinformaticians** — the primary beneficiary. They lose substantial
  time discovering which open cohorts exist for a given cancer type, what assays they contain, and
  whether the license permits their intended (often including commercial-adjacent or
  redistribution-heavy) use. A trustworthy, machine-readable index shortens "can I use this, and
  how?" from hours of portal-spelunking to a query.
- **Patient advocates and advocacy organizations** — who increasingly need to understand the open
  data landscape for a specific (often rare) cancer to push for data sharing and to engage with
  researchers credibly.
- **Students, educators, and reproducibility/methods developers** — who need small, clearly-licensed,
  genuinely-open cohorts for teaching and for safe method development.
- **Informed patients and families** (via a *separate, expert-reviewed, education-only* layer) —
  who benefit from plain-language explanation of what "open cohort," "access tier," and "de-identified"
  mean, **never** clinical guidance.
- **The data custodians themselves** — accurate, attributed indexing increases discoverable,
  correctly-licensed reuse of resources they already funded and curated.

**The verified need.** The *general* need — a license-and-access-aware index of open cancer data —
is well established (it is repeatedly raised in open-science and FAIR-data literature, and partial
prior art exists, e.g. repository-specific search, but no license-first, access-tier-first,
provenance-complete cross-repository index focused on cancer). We treat the general need as real.
However, the **specific, per-partner verified need is `TO BE SECURED`**: we have not yet confirmed a
named research group, advocacy organization, or repository that has agreed to **adopt, host, or
maintain** the catalog as a recognized resource. Until a named beneficiary confirms they will use
and/or steward it, all tasks carry **`verifiedNeed: false`**. This honesty is load-bearing:
"delivered, not merged" requires the catalog to actually be *used by a beneficiary*, not merely
produced.

**Partner / requestor.** `TO BE SECURED`. Candidate channels (none assumed): an academic cancer
bioinformatics group; a rare-cancer foundation or patient-advocacy data initiative; an existing
open-science indexing effort (e.g. a FAIR/registry community) willing to host or cross-link;
a cancer data commons community. M0 includes explicit partner-outreach work; no partner is assumed.

## Goals and non-goals

**Goals**
- Produce a **canonical cohort metadata schema** purpose-built for cancer cohorts, with an
  **access-tier taxonomy** and a **license-rights breakdown** as first-class, required fields. The
  schema models a **typed `entryType`** (patient cohort vs population-statistics vs cell-model vs
  molecular dataset) so it does not falsely presume every entry is a patient cohort, and enforces a
  **cross-repository cohort-identity / dedup rule** (one record per cohort; repositories are locations).
- Become the **canonical cancer access-tier vocabulary + shared schema contract** that the Hee-Lee Oss
  cancer siblings reuse, rather than a fourth overlapping cancer-dataset catalog (boundary in
  "Competitive landscape & differentiation").
- Build a **searchable catalog** of verified **open** cancer cohorts — each entry fully source-cited
  (provenance on every assertion) and gated through the access-tier + license + PII checklist.
- Make **access-tier classification** (OPEN / OPEN-AGGREGATE / REGISTERED-OPEN / NON-COMMERCIAL /
  CONTROLLED) an auditable, non-skippable gate so the catalog **cannot** mislabel controlled or
  identifiable data as open.
- Ship a small, dependency-light **search/browse interface** (static, no patient data, no backend
  that stores anything sensitive) plus **validation tooling** that re-checks each record against the
  schema and flags stale/changed source terms.
- Provide a curated, license-classified **candidate cohort backlog** (`COHORT-CATALOG.md`) that
  feeds per-cohort indexing tasks via the gate.
- Provide an optional, **expert-reviewed, education-only** patient/advocate-facing explainer layer
  ("how to read this catalog," "what access tiers mean") under a high-risk gate.

**Non-goals**
- We do **not** host, mirror, download in bulk, transform, re-derive, or republish any cohort's
  underlying patient data. The deliverable is **metadata about cohorts**, plus tooling.
- We do **not** index controlled-access resources (dbGaP, EGA, individual-level biobanks) or **any**
  identifiable patient data — these are out of scope, not "best-effort included."
- We do **not** provide **medical advice**, treatment guidance, prognosis, or any clinical
  interpretation. Any patient-facing text is education-only, "not medical advice," and expert-reviewed.
- We do **not** rank, score, or editorially recommend cohorts by scientific quality; we describe
  them factually.
- We do **not** perform re-identification, linkage, or any analysis that could increase
  re-identification risk — even on aggregate data.
- We do **not** auto-publish; a human reviews and submits every entry and every contribution.
- We do **not** accept tasks whose primary benefit is a for-profit entity's private data pipeline.

## Success metrics (outcomes)

Outcome-based and beneficiary-centric. Vanity metrics ("entries written") are explicitly excluded;
an entry only counts when it is **verified, gated, and usable**, and impact only counts when it is
**externally observable**.

**Counting rule (anti-gaming).** All "cohorts indexed" / "across N repositories" counts are over
**distinct canonical cohorts** under the cross-repository identity rule (§6), **not** records. The
same cohort appearing in GDC, cBioPortal, Xena, and Synapse is **one** indexed cohort with multiple
`hostingLocations[]`; it can never be counted five times to inflate the metric. The **adopter/steward
metric is the actual gate on "delivered"** — entry count is secondary to a confirmed beneficiary.

| Metric | Baseline | Target (first 6 months) |
| --- | --- | --- |
| **Distinct** open cohorts indexed with a **complete, gate-passed, fully-cited** record (completeness ≥ 90/100; deduped per §6 identity rule) | 0 | ≥ 40 verified **distinct** cohorts |
| Coverage representativeness (not just easy US-gov open data) — rare cancers, pediatric/AYA, non-US sources, modality spread | n/a | ≥ 30% of indexed cohorts are rare-cancer, pediatric/AYA, **or** non-US-sourced; ≥ 3 distinct modalities represented |
| Access-tier classification accuracy (independent reviewer disagreement rate on tier) | n/a | **0 false-"open"** errors; < 5% any-field disagreement on audited sample |
| License-rights accuracy (reviewer disagreement on permits-derivatives / commercial / redistribution) | n/a | **0** records with an unverified "permits-derivatives: true"; < 5% any-field disagreement |
| Cohorts incorrectly admitted that were actually controlled/identifiable (must be zero) | n/a | **0** (any occurrence is a P0 incident) |
| Confirmed adopter/steward (a named group using or hosting the catalog) | 0 | ≥ 1 secured |
| External reuse signal (catalog cited, forked, linked, or queried by a third party with evidence) | 0 | ≥ 3 verifiable external reuse events |
| Staleness: indexed records re-validated within their refresh window | n/a | 100% of records carry a freshness date; ≥ 90% re-validated on schedule |
| Patient/advocate explainer reviewed + signed off by oncologist **and** advocate (if shipped) | n/a | 100% of patient-facing pages have both sign-offs before publish |

**Attribution of outcomes.** A "reuse event" must be **externally verifiable** (a citation, a
linking page, a fork/PR, a documented query from a named group). Self-reported use does not count.
The Steward records each accepted outcome as a committed `outcomes/<cohort-id-or-event>.json`
(channel, URL/permalink, timestamp, completeness score).

**Quantifying "complete, usable record" (so DoDs are checkable).** Each entry gets a
**completeness score (0–100)**: fraction of required canonical fields populated **and** each populated
field carrying a resolvable provenance citation, **and** the access-tier + license gate artifact
attached. A record scores 0 until the gate passes (no partial-credit for ungated records). Target:
every published record ≥ 90/100. A record asserting `accessTier: OPEN` without a cited clause
evidencing open access **fails** regardless of other fields.

## Scope

**In scope**
- A **canonical cancer-cohort metadata schema** (fields in §6) with an access-tier taxonomy and a
  license-rights breakdown, a typed **`entryType`**, and a **cross-repository cohort-identity rule**.
- A **shared access-tier vocabulary + schema contract** published as the reusable layer for the Hee-Lee Oss
  cancer siblings (`cancer-dataset-datasheets`, `ewing-open-data-catalog`, `cancer-data-dictionaries`),
  with an explicit written boundary so the four projects compose into a stack, not four overlapping
  catalogs (see "Competitive landscape & differentiation").
- A **verified catalog** of open / open-aggregate / registered-open cancer cohorts, each entry
  source-cited and gate-passed, emitted as machine-readable records (JSON/CSV) + human-readable pages.
- A **static, no-data search/browse UI** over the catalog (client-side; indexes metadata only).
- **Gate + validation tooling**: the access-tier/license/PII checklist as a reviewable artifact, plus
  scripts that validate records against the schema, check link/citation resolvability, and flag drift.
- The **candidate cohort backlog** (`COHORT-CATALOG.md`) with first-pass license/access triage signals.
- An **excluded-resources register**: a short, explicit list of controlled/identifiable resources
  (dbGaP/EGA/biobanks) recorded **only** as "excluded — controlled, contact custodian," with **no**
  record-level metadata extraction — so users are warned, not misled.
- *(Gated, optional)* an **education-only** patient/advocate explainer layer (high-risk; see §8).

**Out of scope**
- The cohort data itself (no hosting/mirroring/bulk download/transformation/re-derivation/republishing).
- **Controlled-access** resources (dbGaP, EGA, individual-level biobanks) and **any identifiable
  patient data** — never ingested or described at record level.
- Cohorts whose license is unclear/unverifiable, or whose de-identification basis is undocumented →
  flagged/excluded, never guessed.
- Non-commercial / custom-restrictive resources (e.g. COSMIC, OncoKB) are **flagged for a governance
  decision**, not silently included or excluded (see §7).
- Scientific quality ranking, benchmarking, or editorial recommendation of cohorts.
- Medical advice, clinical interpretation, prognosis, or treatment guidance.
- Re-identification, record linkage, or any analysis raising re-identification risk.
- Automated, unattended publishing.

## Competitive landscape & differentiation

**No incumbent offers a cancer-specific, cross-repository, access-tier-first, license-rights-normalized,
per-assertion-provenanced index of *open* cohorts.** The neighbours each solve a different problem:

| Incumbent | What it is | Where it stops (our opening) |
| --- | --- | --- |
| **NCI Cancer Research Data Commons (CRDC) / General Commons** | The authoritative, funded cancer data commons; spans open + controlled; General Commons searches submitted studies, routes controlled access via dbGaP/DCF | Organized by node/repository, **not** a normalized cross-source license-rights breakdown; US/NCI-centric; no single "is this open, and on exactly what terms?" answer across non-NCI sources (GEO, EBI, COSMIC, OncoKB). Treat CRDC as an **authoritative tier-status source we cite**, not a competitor to mirror. |
| **cBioPortal public studies** (~300 curated studies, mostly open) | The closest existing "browse open cancer cohorts" surface; harmonized, widely used | An **analysis portal**, not a license/access-terms catalog — tier and license are not first-class facets; only what's been ingested; no per-assertion provenance. (Now a **seed source**, §7.) |
| **OmicsDI (EMBL-EBI)** | Real cross-repository omics dataset aggregation with an API | Discovery-oriented; **license/access-tier not normalized rights facets**; not cancer-focused; no per-assertion provenance |
| **re3data.org / FAIRsharing.org** | Canonical registries of repositories / databases+standards+policies, with access/license metadata | **Repository/database granularity, not cohort**; broad not cancer; license info is descriptive, not a normalized, clause-cited rights matrix |
| **dbGaP / EGA** | Authoritative **controlled-access** catalogs for individual-level genomic data | These are the **excluded set** — we *point* to them (exact application path), never index them. Their existence is precisely why an "open vs controlled?" disambiguation layer has value. |

**Our differentiator (the moat).** A **cancer-scoped, cross-repository access-tier + normalized
license-rights matrix** (`permitsReuse / derivatives / commercial / redistribution / attribution /
shareAlike`, each with **clause-level provenance**) plus **immutable snapshots** (SHA-256 + Wayback) —
the auditable **"is it open, and exactly what may I do with it?"** layer no incumbent provides. It is
static, data-free, MIT+CC-BY, and forkable, so a steward can host it without standing up a CRDC node.

**Differentiation vs the Hee-Lee Oss cancer siblings (operationalized boundary — load-bearing).** To avoid
four overlapping cancer catalogs, the split is **written and contractual**, not implied:

- **`open-cohort-catalog` (this project) = the breadth / access-tier + license layer.** Owns the
  **access-tier taxonomy and license-rights schema contract**; answers "is it open and on what terms?"
- **`cancer-dataset-datasheets` = the depth layer.** Rich per-dataset Datasheets-for-Datasets narrative;
  **references this project's record by `id`** rather than re-deriving tier/license.
- **`ewing-open-data-catalog` = a single-disease vertical.** **Consumes this schema and vocabulary**
  for the Ewing-sarcoma slice rather than inventing its own tier enum.
- **`cancer-data-dictionaries` = the variable/field-level layer.** Links **up** to the cohort record;
  harmonizes variables, not access terms.

**Win condition:** become the **canonical access-tier vocabulary the three siblings reuse** (one
shared enum + schema contract across all four), turning overlap into a stack. A shared-vocabulary
package and the boundary doc are tracked as deliverables (see Roadmap / `TASKS.md`).

## Solution approach & architecture

This is a **content/data-curation project with light software**: a schema + a verified metadata
catalog + a static search UI + validation/gate tooling. It is **not** a data pipeline that moves
patient data; the only bytes it touches are **publicly documented cohort-level descriptors** (counts,
assay names, license text, access terms), never patient records.

**Pipeline (per cohort)**
1. **Discovery & candidacy** — identify a cohort, its hosting repository, and its custodian from the
   candidate backlog (`COHORT-CATALOG.md`) or outreach.
2. **Access-tier + license + PII gate (blocking)** — classify the access tier; verify the license and
   its rights breakdown from primary source text; confirm the cohort is de-identified/aggregate and
   that **no identifiable or controlled-tier data** is in scope. PASS / FLAG / EXCLUDE with a committed
   artifact. *No record is authored before this passes.*
3. **Metadata authoring** — populate the canonical cohort record from **publicly documented** sources
   only (repository landing pages, data dictionaries, publications, custodian DUAs/terms). **Every
   asserted field carries a provenance citation** (source URL + retrieval date + the specific claim).
4. **Provenance + license snapshot** — capture source URL, custodian, retrieval timestamp, dataset
   version/freeze date, license id + URL + a **snapshot** of the license/terms text (committed copy +
   SHA-256 + Wayback save URL), required citation string.
5. **Validation** — schema validation, citation/link resolvability check, access-tier consistency
   check (e.g. an `OPEN` record may not carry `duaRequired: true`), and a completeness score.
6. **Review** — Access/License/PII reviewer (mandatory) + technical reviewer; domain (oncology data)
   reviewer for ambiguous tiers; oncologist + advocate for any patient-facing text.
7. **Publish & track** — a human merges the record into the catalog; the static UI rebuilds; the
   Steward records adoption/reuse outcomes.

**Unit of cataloging — `entryType` (resolves "cohort vs dataset").** The project name says "cohort,"
but the seed sources are heterogeneous: DepMap indexes **cell lines** (no patients), GLOBOCAN/SEER are
**population statistics** (no cohort), GEO *series* are **datasets**, TCGA is a *program*, GENIE is a
*registry*. To stop the schema from misrepresenting half the seed sources, every record declares a
typed **`entryType`**: `patient-cohort | population-statistics | cell-model-collection |
molecular-dataset`. The **shape of `aggregateCounts` is conditional on `entryType`** (a
`cell-model-collection` carries `cellLines`/`models`, not `patients`; `population-statistics` carries
the statistical universe, not a sample roster). "Cohort" is used as the umbrella term for an indexed
entry; the typed field carries the precise meaning.

**Cross-repository cohort identity / dedup rule (so counts aren't gameable).** **One canonical record
per distinct cohort; repositories are *locations*, not separate cohorts.** TCGA appears in GDC,
cBioPortal, Xena, and the legacy ICGC portal; GENIE appears in Synapse *and* cBioPortal — these are
**one** record each with multiple `hostingLocations[]`, not five records. The schema defines explicit
**identity keys** (primary custodian + program/cohort accession + data-generation scope) and a
**primary-custodian vs mirror** model. All coverage metrics count **distinct cohorts**, never records,
closing the "index the same cohort five times to hit ≥40/≥5" loophole.

**Canonical cohort metadata model (source of truth).** One internal record per **distinct** cohort;
all outputs (JSON, CSV, catalog page, search index) are *projections* of it. Required fields:
- `id`, `name`, `aka[]`
- `entryType` — enum `patient-cohort | population-statistics | cell-model-collection | molecular-dataset`
- `identityKeys` `{primaryCustodian, programOrAccession, generationScope}` — the dedup key (see rule above)
- `custodian` (publishing body), `primaryHostingRepository`, `hostingLocations[]` `{repository, url,
  isMirror}` (e.g. GDC, GEO, EMBL-EBI, PRIDE, DepMap, cBioPortal, Xena, Synapse, SEER, IARC) — mirrors
  recorded as locations of the **same** record, never as new records
- `cancerTypes[]` (coded: NCIt / OncoTree / ICD-O-3 where available, plus free-text), `population`
  (pediatric / AYA / adult / mixed / n-a-for-cell-models)
- `aggregateCounts` — **shape conditional on `entryType`**: `{patients?, samples?, cellLines?, models?,
  statisticalUniverse?, asOf}` — **aggregate counts only; never per-patient rows**
- `assayTypes[]` (e.g. WES, WGS, RNA-seq, methylation, proteomics, WSI/imaging, clinical-aggregate)
- `dataModality[]`
- `generationContext` `{funder?, consentBasis?, dataGenerationDateRange?}` — cheap-to-capture context
  useful to beneficiaries (each field still cited via `fieldProvenance`; optional, never guessed)
- `accessTier` — enum: `OPEN | OPEN-AGGREGATE | REGISTERED-OPEN | NON-COMMERCIAL | CONTROLLED`
  (only the first three are indexable; `NON-COMMERCIAL` → FLAG; `CONTROLLED` → EXCLUDE)
- `license` `{id (SPDX or custom-id), url, permitsReuse, permitsDerivatives, commercialUseAllowed,
  redistributionAllowed, attributionRequired, shareAlike, snapshotRef}` — booleans **must** each be
  backed by a cited clause; unknown ≠ true
- `accessTerms` `{registrationRequired, clickThrough, duaRequired, irbRequired, citationRequirement}`
- `deIdentification` `{level, standard (e.g. HIPAA Safe Harbor / Expert Determination / aggregate-only),
  publisherDocumented:boolean, notes}`
- `provenance` `{sourceUrl, retrievedAt, version/freezeDate, custodianContact, attribution}`
- `fieldProvenance[]` — **per-assertion** citations: `{field, claim, sourceUrl, retrievedAt}`
- `governanceFlags[]` (e.g. `nc-license`, `registered-access`, `mixed-tier-open-portion-only`)
- `crossRefs[]` (related dbGaP/EGA accession **as a pointer only**, related portals)
- `recordState` — enum `current | termsChangedSinceSnapshot | underReverification` — a **first-class**
  state (not just a maintenance ticket) flipped when drift detection sees the source terms change (§15)
- `completenessScore {value}`

**Index ≠ redistribute (REGISTERED-OPEN / no-redistribution sources are still indexable).** A record
may have `license.redistributionAllowed: false` (e.g. AACR Project GENIE forbids redistributing the
*data*) and still be **indexable**: we publish independently-verified factual **descriptors + a
citation**, not the custodian's data or copyrighted prose. The publish linter must therefore treat
`redistributionAllowed: false` as *valid for an index entry* and never reject it on that basis.

**Mixed-tier resources rule.** For resources with both open and controlled tiers (e.g. TCGA via the
GDC), the record describes **only the open-access tier**, sets `governanceFlags: [mixed-tier-open-portion-only]`,
and explicitly names the controlled portion as out of scope. The catalog never extends a record to
controlled data.

**Tech stack.** TypeScript, ESM, pnpm workspaces (Hee-Lee Oss convention). The schema is JSON Schema +
TypeScript types. Validators/gate tooling are small Node packages with minimal dependencies. Catalog
records are JSON (one file per cohort) with generated CSV. The search UI is a **static client-side
site** (e.g. prebuilt search index over the JSON; no server that stores user input, no patient data,
no tracking). Everything runs locally or in CI; **no runtime service holds sensitive data** (there is
none to hold).

**Interfaces / outputs.** (a) `cohort.schema.json` + TS types; (b) `catalog/*.json` records + a
generated `catalog.csv`; (c) a static search/browse site; (d) gate artifacts `gates/<cohort-id>.json`;
(e) `outcomes/*.json` adoption/reuse ledger. Code is MIT; catalog **metadata** is CC-BY-4.0 (or CC0
where we author purely factual metadata — decided in `policy-lic-*`), with upstream attribution preserved.

**Key decisions.**
- **Access-tier-first, license-first.** A record cannot exist without a gate-passed access tier and a
  cited license-rights breakdown. This is the spine of the project, not a column.
- **Metadata-only, provenance-complete.** Every assertion is cited; the catalog is auditable end to end.
- **Mixed-tier = open-portion-only.** Never let a record bleed into a controlled tier.
- **Static, dataless UI.** No backend that could accumulate sensitive data; the search index is over
  public metadata.

**AI assistance (Claude) — drafting only, never deciding.** Claude is used to accelerate the most
tedious steps, always behind the human gate, and only against **public, cohort-level descriptor pages**:
1. **Cohort-metadata extraction** — Claude reads a public GEO series page, GDC/cBioPortal project page,
   or DepMap release notes and **proposes** the structured record, emitting a candidate
   `fieldProvenance` (source URL + the exact quoted clause) **for every field**. Human verifies; Claude
   never commits.
2. **Access/license language normalization** — map free-text terms ("freely available for academic
   use," "requires registration," "no commercial use") into the `accessTier` enum + rights-boolean
   matrix **as a quoted proposal with the basis span**, flagging ambiguity rather than guessing.
3. **Drift triage** — diff a fresh terms page against the stored snapshot and summarize *what changed*
   (tier? license? redistribution clause?), routing material changes to human re-verification (§15).
4. **Cancer-type coding + lint explanation** — propose OncoTree/NCIt/ICD-O-3 codes (with rationale,
   for domain-reviewer confirmation) and explain *why* a record failed the access-tier linter.

**Hard AI guardrails (bind the model exactly as they bind a contributor):**
- **Tier and license determinations are always `FLAG-for-review`, never `PASS`.** Claude proposes +
  cites; the Access/License/PII reviewer decides. A model-only "OPEN" must **never** reach the
  published catalog (the P0 failure mode).
- **Never fabricate terms or citations.** Every Claude-asserted clause must resolve to a real, quoted
  source span; "unknown ≠ true" binds the model — it may not infer a permissive default from silence.
- **Never ingest controlled/identifiable data.** Claude is pointed only at public cohort-level
  descriptors — never controlled data, per-patient rows, or anything behind a DUA. If a page surfaces
  identifiable content, **halt-and-discard** (do not summarize it into a record).
- **No de-identification / re-identification / linkage reasoning, and no medical advice** by Claude;
  patient-facing text stays oncologist + advocate-gated.
(Consult the Anthropic model/pricing/caching guidance via the `claude-api` skill before implementing;
the extraction step benefits from prompt-caching the schema + gate rubric.)

## Data, licensing & compliance

**THIS IS THE CRITICAL SECTION, AND IT LEADS WITH THE BINDING CANCER GUARDRAILS.**

### Binding cancer-data guardrails (non-negotiable)
1. **Open-access / aggregate / de-identified ONLY.** The catalog indexes only cohorts that are
   open-access, open-aggregate, or free-registered-open **and** de-identified/aggregate.
2. **Controlled-access is OUT OF SCOPE.** dbGaP, EGA, and individual-level biobanks (UK Biobank,
   Genomics England, All of Us, and equivalents) are **excluded** — never ingested, never described
   at record level, only listed in the excluded-resources register as "controlled — contact custodian."
3. **No identifiable patient data, ever.** Not in the catalog, not in examples, not in CI fixtures,
   not in logs. The catalog stores **cohort-level descriptors only** (aggregate counts, assay names,
   license/terms text, provenance) — **no per-patient rows, no quasi-identifier combinations**.
4. **Per-source license verification is mandatory.** TCGA/GDC **open tier** and GEO **open** records
   are treated as open (with cited terms); **COSMIC** and **OncoKB** are **non-commercial / custom →
   FLAG** for a governance decision (never default-included). Every license is verified from primary
   text, not assumed.
5. **No medical advice.** Patient-facing content is **education only**, carries an explicit
   "**not medical advice**" notice, and ships only after **oncologist + patient-advocate** review
   (riskTier **high**).
6. **Provenance on every assertion.** Every catalog field is backed by a resolvable citation.

### Access-tier taxonomy (the gate's core decision)
| Tier | Meaning | Catalog action |
| --- | --- | --- |
| `OPEN` | Public, no registration; data + derivative metadata reusable | **Index** |
| `OPEN-AGGREGATE` | Open aggregate statistics only (e.g. incidence dashboards) | **Index** (aggregate-only) |
| `REGISTERED-OPEN` | Free but click-through/registration; de-identified aggregate (e.g. AACR Project GENIE) | **Index**, flag `registered-access` |
| `NON-COMMERCIAL` | Custom/NC terms (e.g. COSMIC, OncoKB) | **FLAG** → governance decision; not auto-included |
| `CONTROLLED` | dbGaP/EGA/biobank application/DUA/IRB for individual-level data | **EXCLUDE** → register pointer only |

### Source licenses & terms (initial classification — each re-verified at the gate)
**Accepted (open; reuse + derivative metadata permitted):**
- **NIH/NCI Genomic Data Commons — open tier** (TCGA, TARGET, CPTAC open-access data): US-government /
  NIH Genomic Data Sharing open tier; open data carry no access restriction. Record the open tier only.
- **NCBI GEO** (Gene Expression Omnibus): open; NCBI value-added content is US-gov public domain;
  submitter-deposited data are generally reusable — verify per series for any stated restriction.
- **EMBL-EBI resources** — **Expression Atlas** (CC-BY-4.0), **PRIDE** (open, commonly CC0),
  **ArrayExpress/BioStudies** (mostly open; verify per study) — verify each study's stated terms.
- **DepMap** (Broad; cell-line dependency/omics, **not patient data**): CC-BY-4.0. (`entryType:
  cell-model-collection` — counts are cell lines/models, not patients.)
- **Cell Model Passports** (Sanger; cell-line metadata): open — verify exact license at gate.
- **cBioPortal public studies** (~300 curated, mostly-open cancer genomics studies): the closest
  existing "browse open cancer cohorts" surface and a primary **seed source**; many studies mirror
  cohorts hosted elsewhere (GDC, Synapse) → apply the cross-repository identity rule and verify each
  study's stated terms (some carry source-specific or no-redistribution clauses).
- **ICGC open-access tier / PCAWG open data**: open tier accepted; controlled tier excluded.
- **SEER\*Explorer / IARC GLOBOCAN / CDC WONDER** — **aggregate** incidence/mortality statistics
  (`entryType: population-statistics`): open-aggregate, **terms verified not assumed** — IARC GLOBOCAN
  is **not** simply "open" (it carries attribution requirements and use conditions; verify per the
  GLOBOCAN/Cancer-Today terms); SEER terms verified; CDC is US-gov public domain. Note: SEER **research
  microdata** requires a signed agreement → that microdata is `REGISTERED`/`CONTROLLED` and **out of
  scope**; only the open aggregate layer is indexed.

**Flagged (non-commercial / custom → governance decision, not default-included):**
- **COSMIC** (Wellcome Sanger Institute): free for **non-commercial** use with registration;
  commercial use requires a paid license; redistribution restricted → **FLAG**.
- **OncoKB** (MSK): proprietary; academic/research license with restrictions; commercial restricted;
  redistribution restricted → **FLAG**.

**Excluded (controlled / identifiable → register pointer only, never indexed):**
- **dbGaP** controlled studies; **EGA** studies; **TCGA/ICGC controlled tiers** explicitly incl.
  **germline + raw sequence/BAMs**; **GTEx** and **TOPMed** (controlled individual-level tiers);
  **individual-level biobanks** (UK Biobank, Genomics England, All of Us, MVP).

**Objective "indexable" criterion.** A cohort is indexable **only if** its `accessTier ∈ {OPEN,
OPEN-AGGREGATE, REGISTERED-OPEN}` **and** `license.permitsReuse: true` with a cited clause/URL,
**and** `license.permitsDerivatives: true` for the *metadata we publish* (or the metadata is bare
factual data we may state regardless), **and** de-identification/aggregate status is publisher-documented.
Any missing evidence, unparseable license, or undocumented de-identification = **FLAG/EXCLUDE**, never
default-allow. NC/custom licenses are decided by the written NC policy (`policy-nc-*`).

**Provenance model.** Every record stores source URL, custodian, retrieval timestamp, version/freeze
date, license id + URL + a committed **snapshot** (text copy + SHA-256 + Wayback save URL) of the
license/terms as they stood at retrieval, the required citation string, **and** a per-assertion
`fieldProvenance[]` linking each populated field to the claim that supports it. A bare link is **not**
sufficient evidence for any rights boolean.

**Privacy / PII stance.** The catalog is **metadata-only and patient-data-free by construction**. The
gate nonetheless runs an explicit PII/identifiability check: confirm the cohort is published as
de-identified/aggregate; confirm **we are extracting only cohort-level descriptors**; reject any
candidate where indexing would require touching per-patient rows or where de-identification is
undocumented. We **never** de-identify, re-identify, link, or aggregate patient data ourselves — that
would be transforming the data (out of scope). If discovery surfaces any identifiable datum, work
halts, the datum is discarded, and the cohort is routed to EXCLUDE with the concern surfaced.

**Legal theory for our own metadata (resolves the CC-BY-vs-CC0 question).** What we publish is
**independently-verified factual descriptors** (counts, assay names, license id, access tier) **plus a
citation** — *not* the custodian's copyrighted prose or their data. Bare facts are generally not
copyrightable and need no license grant from the source; conversely, a source license that forbids
redistributing **their** data/metadata (e.g. GENIE) does not bar us from stating independently-checked
facts about it with attribution. This theory is what makes a source's `redistributionAllowed: false`
compatible with an index entry, and is the basis for choosing our metadata license (`policy-lic-035`).

**Attribution.** Every record attributes the custodian per the source terms, links to the source, and
states the catalog metadata — not the data — is our contribution. Catalog metadata is **CC-BY-4.0**
(CC0 considered for purely factual fields per `policy-lic-*`); code (schema/validators/UI) is **MIT**.

## Quality, review & risk gates

**Risk tier: medium** for the researcher-facing catalog; **high** for any patient-facing educational
layer (per CLAUDE.md and the cancer track guardrails).

**Required review before a deed is "done":**
- **Access/License/PII reviewer (mandatory, every cohort)** — the **hard gate**. Confirms the access
  tier (zero tolerance for a false "open"), the license-rights breakdown (each boolean cited), and that
  the cohort is de-identified/aggregate with no identifiable/controlled data in scope. No record ships
  without this sign-off. Must be filled **before the M0 pilot is reviewed** (blocking prerequisite).
- **Mandatory second reviewer for the error-prone classes (not sampled).** Because a sampled audit can
  miss a rare catastrophic mislabel, **every `REGISTERED-OPEN` record and every `mixed-tier` record
  gets a second independent Access/License/PII sign-off** — 100%, not a sample. These are the classes
  most likely to be mis-tiered (registered-but-open vs controlled; open-portion vs controlled-portion).
- **Technical reviewer** — confirms schema validity, citation/link resolvability, validator CI green,
  and that the static UI exposes no data beyond public metadata.
- **Oncology-data domain reviewer** — engaged for ambiguous tiers, mixed-tier resources, or any cohort
  where cancer-type coding / de-identification basis needs expert judgment (escalates to medium/high).
- **Oncologist + patient-advocate (high-risk gate)** — **both** sign off on any patient-facing
  educational text before publish; text carries an explicit "not medical advice" notice.

**Test fixtures & golden files (so "CI green" means something).** Tooling ships with committed,
**synthetic** fixtures only (never real cohort extracts):
- **Schema validator** — golden valid records that must pass and malformed records (e.g. `OPEN` +
  `duaRequired: true`; a rights boolean with no `fieldProvenance`) that must fail.
- **Citation resolver** — fixtures asserting that every `fieldProvenance` and `license.url` resolves
  (mocked in CI) and that records with dead/missing citations fail.
- **Access-tier consistency linter** — rules encoded as tests (e.g. `CONTROLLED` must never appear in
  the published catalog; `NON-COMMERCIAL` must carry a governance-decision reference).

**Definition of Shipped.** A cohort record is *shipped* when it is **gate-passed (access tier +
license + PII), fully source-cited (completeness ≥ 90/100), schema-valid, CI-green, reviewer-approved,
published in the catalog, AND demonstrably usable by a beneficiary** — i.e. the catalog (or the record)
is adopted/queried/cited by a named third party, with the Steward's outcome artifact recorded.
Producing a record is **not** shipped; recorded beneficiary use is. For any patient-facing page,
shipped additionally requires **both** oncologist and advocate sign-offs on file.

## Roadmap & milestones

**M0 — Foundation & cold-start (thin)**
- Goal: define the schema + access-tier taxonomy + gate, prove the end-to-end flow on **one** pilot
  cohort, and open partner outreach.
- **Cold-start de-risking.** To avoid building a catalog nobody adopts, the pilot uses the
  lowest-risk, clearly-open cohort (a US-gov / CC-BY, no-patient-data resource such as **DepMap** or a
  **GDC open-tier TCGA** descriptor) so a real, verifiable record is reachable without depending on a
  third party; and at least one outreach thread is opened toward a potential adopter/steward.
- Exit criteria: (1) canonical cohort schema + access-tier taxonomy + license-rights model published,
  including the typed **`entryType`** (patient-cohort / population-statistics / cell-model-collection /
  molecular-dataset), conditional `aggregateCounts`, and the **cross-repository cohort-identity / dedup
  rule** (one record per distinct cohort; repositories are `hostingLocations`); (2) access-tier/license/PII
  gate checklist (incl. the NC policy) exists and is applied to one cohort; (3) schema validator +
  citation resolver green in CI with golden fixtures; (4) **one** cohort fully indexed end-to-end
  (gate-passed, fully cited, completeness ≥ 90/100); (5) ≥ 1 partner/adopter outreach thread opened;
  (6) excluded-resources register seeded with the controlled set (dbGaP/EGA/biobanks + TCGA-controlled/
  germline, GTEx, TOPMed); (7) the **sibling boundary + shared access-tier vocabulary/schema-contract
  doc** drafted (this project owns the vocabulary the siblings reuse).

**M1 — Gate hardened + first real coverage**
- Goal: make the gate rigorous and index a credible first set across repositories.
- Exit criteria: (1) gate codified as a reviewable artifact and applied to ≥ 10 **distinct** cohorts,
  with **mandatory second-reviewer** on every `REGISTERED-OPEN` / `mixed-tier` record; (2) ≥ 10
  **distinct** cohorts published (gate-passed, fully cited, deduped per the identity rule) spanning ≥ 3
  repositories; (3) NC policy decided and COSMIC/OncoKB dispositioned per it; (4) license-snapshot
  capture working (committed copy + SHA-256 + Wayback); (5) **seed list expanded** to include
  cBioPortal public studies (+ candidate UCSC Xena, GDC non-TCGA open programs, CCLE/GDSC, HTAN,
  ProteomeXchange cancer sets) on the candidate backlog; (6) ≥ 1 confirmed adopter/steward **or** an
  honest "not yet secured" with blockers surfaced.

**M2 — Searchable interface + scale**
- Goal: ship the static search/browse UI and scale coverage via the candidate backlog.
- Exit criteria: (1) static search UI live over the catalog (metadata-only, no patient data, no
  tracking), faceted by cancer type / repository / access tier / license / `entryType`; (2) ≥ 40
  **distinct** cohorts (deduped per the identity rule) published cumulatively across ≥ 5 repositories,
  **meeting the coverage-representativeness target** (≥ 30% rare-cancer / pediatric-AYA / non-US;
  ≥ 3 modalities); (3) staleness/drift detection running with a **concrete signal** — a stored content
  hash of each terms URL, re-checked on the SLA, flipping the record to the first-class
  `termsChangedSinceSnapshot` state on change; (4) automated CI gate that **blocks** any record
  violating the access-tier rules (no CONTROLLED, no uncited rights boolean), while correctly
  **allowing** `redistributionAllowed: false` index entries; (5) every record emits **schema.org/Dataset
  + DCAT JSON-LD** so the catalog is itself discoverable (turning Google Dataset Search into a channel).

**M3 — Adoption, reuse outcomes & sustainability + (optional) patient layer**
- Goal: demonstrate real beneficiary use, a maintenance model, and — if and only if expert review is
  secured — a high-risk education-only patient/advocate explainer layer.
- Exit criteria: (1) ≥ 3 verifiable external reuse events; (2) ≥ 1 confirmed adopter/steward for
  ongoing maintenance; (3) documented refresh/maintenance process with freshness SLAs; (4) **if
  shipped**, the patient-facing explainer layer has oncologist **and** advocate sign-offs and the
  "not medical advice" frame on every page (otherwise explicitly deferred, not quietly dropped).

Dependencies: M1 depends on the M0 schema+gate; M2 UI depends on M1's record corpus; M3 depends on a
body of published, adopted records; the patient layer depends on securing the high-risk reviewers.

## Work breakdown

The itemized, schema-mapped backlog lives in `TASKS.md`, organized by the milestones above, each with
a task table (`ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer`), acceptance
criteria for the most important tasks, and a milestone Definition of Done. A backlog of
sized-but-unscheduled tasks and one complete, schema-valid example Task JSON are included there. The
**candidate cohort backlog** that feeds per-cohort indexing tasks lives in `COHORT-CATALOG.md`;
`TASKS.md` includes a catalog-triage task and concrete example per-cohort tasks drawn from real
repositories, each mapped to the Task JSON schema and each requiring its own committed gate artifact
before work proceeds.

## Governance, roles & stakeholders

- **Maintainer (Owner):** TBD — owns the schema, gate, tooling, and backlog.
- **Access/License/PII reviewer:** TBD (name `TO BE SECURED`) — **mandatory, non-skippable** gatekeeper;
  no record ships without this sign-off. Filled **before** the M0 pilot is reviewed. Must be able to
  read genomic-data-sharing policies, open-data licenses (SPDX/CC/OGL + custom NC terms like
  COSMIC/OncoKB), and reason about de-identification/access tiers. May rotate among ≥ 2 qualified
  reviewers to avoid a bottleneck, but ≥ 1 qualified reviewer must exist at all times or indexing halts.
- **Technical reviewer(s):** rotation verifying schema validity, citations, validators, and the UI.
- **Oncology-data domain reviewer(s):** engaged per-cohort for ambiguous tiers / mixed-tier resources /
  cancer-type coding (medium/high tier).
- **Oncologist + patient-advocate reviewers (high-risk):** `TO BE SECURED` — **both** required before
  any patient-facing content ships. Until secured, the patient layer is **not built** (deferred, not faked).
- **Steward (last-mile owner):** TBD — owns adopter relationships and confirms beneficiary use (the
  "delivered" signal); records the outcome ledger.
- **Partner / requestor:** `TO BE SECURED` — a named research group, advocacy org, or repository that
  adopts/hosts/maintains the catalog. Until named, all tasks carry `verifiedNeed: false`.

## Dependencies & integrations

- **Repositories / custodians (sources, not integrations that move data):** NIH/NCI Genomic Data
  Commons (GDC: TCGA/TARGET/CPTAC open tier, + candidate non-TCGA open programs HCMI/CGCI/MMRF), NCBI
  GEO, EMBL-EBI (Expression Atlas, PRIDE, ArrayExpress/BioStudies), **cBioPortal public studies**,
  **UCSC Xena**, Broad DepMap, Sanger Cell Model Passports, candidate CCLE/GDSC and HTAN, ICGC/PCAWG
  (open tier), SEER (aggregate), IARC GLOBOCAN, CDC WONDER, AACR Project GENIE (registered-open; Synapse
  + cBioPortal locations of one cohort). Each verified per-source; multi-repository appearances deduped
  to one record per the §6 identity rule.
- **Authoritative tier-status reference (cite, don't mirror):** NCI Cancer Research Data Commons (CRDC) /
  General Commons — the funded source of truth for open-vs-controlled status we reference at the gate.
- **Flagged sources:** COSMIC, OncoKB (NC/custom → governance decision).
- **Excluded sources (register pointer only):** dbGaP, EGA, TCGA/ICGC controlled tiers (germline/BAMs),
  GTEx, TOPMed, individual-level biobanks (UK Biobank, Genomics England, All of Us, MVP).
- **Standards / vocabularies:** SPDX license identifiers; NCIt / OncoTree / ICD-O-3 cancer-type coding;
  schema.org/Dataset & DCAT for cross-walk; Datasheets-for-Datasets concepts; FAIR principles.
- **Tooling:** Wayback Machine (license snapshots), SHA-256 hashing, a static-site/search-index builder.
- **Hee-Lee Oss pieces:** Task JSON schema (`packages/schema`), the donated-lane CLI workspace/PR flow
  (`packages/cli`), good-deed definition + refusal guardrails. **No funded-lane/runner dependency**
  (this project is donated lane).

## Risks & mitigations

| Risk | Likelihood | Impact | Mitigation | Owner |
| --- | --- | --- | --- | --- |
| Mislabeling a controlled/identifiable cohort as "open" (the worst error) | Low | Critical | Access-tier gate as a blocking step; CONTROLLED can never enter the published catalog (CI-enforced); mandatory reviewer; zero-tolerance metric | Access/License/PII reviewer |
| Mis-classifying a license (treating COSMIC/OncoKB NC terms as freely reusable) | Medium | High | Per-source verification from primary text; rights booleans require cited clauses; NC policy decides flagged sources; default exclude on doubt | Access/License/PII reviewer |
| Any identifiable patient datum entering a "metadata-only" catalog | Low | Critical | Metadata-only by construction; PII check in the gate; halt-and-discard on any identifiable signal; synthetic-only fixtures | Access/License/PII reviewer |
| Patient-facing content drifts into medical advice | Medium | High | Patient layer gated behind oncologist + advocate sign-off; "not medical advice" on every page; default = don't build it | Oncologist + advocate |
| No adopter secured → catalog built but unused (fails "delivered") | Medium | High | M0 outreach; Steward role; `verifiedNeed:false` until secured; pilot chosen for standalone usefulness | Steward |
| Source terms / versions change, making records stale or wrong | Medium | Medium | License snapshot + freeze date; drift detection; freshness SLA; maintenance tasks | Maintainer |
| Scope creep into hosting/transforming/re-deriving cohort data | Medium | High | Explicit non-goal; reviewers reject any data-movement work; metadata-only invariant | Maintainer |
| Mixed-tier resource record bleeds into the controlled tier | Medium | High | "Open-portion-only" rule + `mixed-tier` flag; reviewer checks tier boundary | Access/License/PII reviewer |
| Cancer-type miscoding misleads users (esp. rare cancers) | Medium | Medium | Coded vocabularies (NCIt/OncoTree/ICD-O-3) + domain reviewer; free-text kept secondary | Oncology-data reviewer |
| Static UI accidentally collects/exposes sensitive input | Low | Medium | Dataless static site; no backend store; no tracking; reviewed | Technical reviewer |

## Security & privacy

- **Threat surface is small by design** (no patient data, no runtime data store, static UI). The main
  surfaces are CI and the published metadata/UI.
- **Patient data:** there is none in the system — this is the central privacy control. The catalog is
  cohort-level metadata only; identifiability is prevented by construction and re-checked at the gate.
- **Secrets handling:** tooling needs no credentials by default. Any repository API token used for
  link-checking is supplied by the human and must **never** be written into logs, receipts, or
  committed files (per CLAUDE.md). No secrets in fixtures.
- **Provenance integrity:** license snapshots are hashed (SHA-256) and Wayback-archived so a later
  source change cannot silently rewrite history.
- **Abuse/misuse prevention:** refuse and flag any task that would (a) launder a controlled/non-open
  resource as "open," (b) attempt re-identification or linkage, (c) bulk-mirror or re-derive cohort
  data, (d) steer the catalog toward surveillance or targeting individuals, or (e) primarily serve a
  for-profit data pipeline. The catalog stays descriptive, source-verified, and open.

## Sustainability & maintenance

- **Post-delivery ownership:** the Steward maintains adopter relationships and the outcome ledger; the
  Maintainer keeps the schema, validators, gate, and UI current with source/standard changes.
- **Freshness SLA + concrete drift signal:** each record carries a freeze date and a refresh window
  (e.g. annual, or on a detected source change). Because most sources have **no machine-readable terms
  feed**, drift is detected by storing a **content hash of each terms URL** and re-checking it on the
  SLA; a changed hash flips the record to the first-class **`termsChangedSinceSnapshot`** state (not
  merely a maintenance ticket) and routes it to human re-verification (Claude may pre-triage *what*
  changed, §6 architecture). Confirmed-stale records become `maintenance` tasks.
- **Outcome tracking:** the Steward records adoption and external reuse events against the success
  metrics, reviewed each milestone. Access-tier/license errors are tracked as incidents (target zero).
- **Community contribution:** once the schema + gate are stable, external contributors can propose
  cohorts via the gate; no contribution is published without the mandatory reviewer sign-off.

## Adjacent opportunities

Natural extensions once the catalog + schema contract are stable (each preserves the metadata-only,
no-patient-data, human-gated invariants; none is a commitment in the first 6 months):

- **Cross-cohort access-terms API** (`GET /cohort/{id}/terms` → access tier + normalized rights matrix
  + provenance). The catalog's most reusable output: lets downstream pipelines and the Hee-Lee Oss siblings
  **gate on license before touching data**, served as static JSON (no patient data, no backend store).
- **"Is this cohort open?" MCP server** — wraps the access-terms API as a tool any agent/IDE can call
  to check tier + license **before** suggesting a download; a sharp, demoable, Anthropic-ecosystem
  distribution channel. Returns the cited rights matrix, never a download.
- **Shared Hee-Lee Oss access-tier vocabulary package** consumed by `ewing-open-data-catalog`,
  `cancer-dataset-datasheets`, and `cancer-data-dictionaries` — the concrete artifact that turns the
  four projects into one stack (the "win condition" above).
- **License-snapshot / drift watcher as a standalone utility** — "did the terms change?" is useful to
  any FAIR catalog; potentially contributable to the FAIRsharing/re3data communities.
- **Controlled-access concierge** — for excluded dbGaP/EGA/CRDC studies, a pointer record carrying the
  *exact* application path (DACO, dbGaP authorized access) so users are neither misled into thinking
  the data is open **nor** that it is unavailable.

## Open questions

- **What is the indexed unit of record**, and how do multi-repository appearances dedup to one
  canonical cohort? (Resolved in v0.2: typed `entryType` + cross-repository identity rule, §6 — kept
  here as the standing governance question if new source shapes appear.)
- **What is the legal basis** for publishing our *own* metadata (CC-BY vs CC0), given some sources
  (e.g. GENIE) forbid redistributing *their* data/metadata? (v0.2 states the legal theory in §7;
  `policy-lic-035` finalizes CC-BY-4.0 vs CC0 for purely factual fields.)
- **Sibling boundaries:** is this project the *owner* of the shared access-tier vocabulary the cancer
  siblings consume — and how is divergence into four tier taxonomies prevented? (v0.2 default: yes,
  owner; boundary doc + shared schema-contract package are M0/Roadmap deliverables.)
- **Coverage target:** breadth-first (many easy US-gov open resources) vs depth/diversity-first (rare
  cancer, non-US, NC-flagged)? (v0.2 adds a coverage-representativeness metric pushing toward diversity.)
- **Is cBioPortal in scope** as a seed source? (v0.2 default: yes — added to §7 seed sources, deduped
  via the identity rule since it mirrors cohorts hosted elsewhere.)
- **Drift-detection authority:** absent machine-readable terms feeds, what is the concrete change-signal
  and re-verification cadence per source? (v0.2 default: terms-URL content hash + SLA re-check +
  `termsChangedSinceSnapshot` state; cadence set per-source in the freshness SLA.)
- **Adopter-first sequencing:** should M0 secure the steward *before* indexing toward 40 entries, given
  "delivered" hinges on adoption, not entry count? (v0.2 makes the adopter the gating metric.)
- Which named research group, advocacy org, or repository will be the first **confirmed
  adopter/steward**? (Blocks `verifiedNeed: true`.)
- Catalog metadata license: **CC-BY-4.0** vs **CC0** for purely factual fields — decided in `policy-lic-*`
  (default CC-BY-4.0 with upstream attribution preserved).
- NC/custom sources (COSMIC, OncoKB): include with a prominent NC flag, or exclude and only pointer them?
  (Default: FLAG → governance decision; do not auto-include.)
- For `REGISTERED-OPEN` resources (e.g. AACR Project GENIE), is documenting the registration step
  sufficient, or should the catalog distinguish "registered-open" prominently in the UI? (Default: yes,
  prominent badge.)
- Cancer-type coding: standardize on **OncoTree** (oncology-friendly) with NCIt/ICD-O-3 cross-refs, or
  lead with NCIt? (Default: OncoTree primary, cross-referenced.)
- Will the optional patient-facing layer be built at all in the first 6 months, given it requires
  securing oncologist + advocate reviewers? (Default: deferred until reviewers secured.)

## References

- Hee-Lee Oss work rules — `C:\code\hee-lee-oss\CLAUDE.md`
- Good Deed Definition + risk tiers — `C:\code\hee-lee-oss\docs\good-deed-definition.md`
- Task JSON schema — `C:\code\hee-lee-oss\packages\schema\src\schemas.ts`
- Portfolio roadmap (Track 8b: open-cohort-catalog) — `C:\code\hee-lee-oss\planning\ROADMAP.md`
- Sibling plan (house conventions) — `C:\code\hee-lee-oss\planning\projects\open-data-datasheets\PLAN.md`
- NIH Genomic Data Sharing Policy; NCI Genomic Data Commons (GDC) data access tiers
- NCBI GEO; EMBL-EBI (Expression Atlas, PRIDE, ArrayExpress/BioStudies) terms
- Broad DepMap (CC-BY-4.0); Sanger Cell Model Passports; ICGC/PCAWG data access
- SEER\*Explorer; IARC GLOBOCAN / Cancer Today; CDC WONDER
- AACR Project GENIE access terms; COSMIC license; OncoKB terms of use
- SPDX license list; OncoTree / NCIt / ICD-O-3 cancer-type vocabularies; FAIR principles;
  Datasheets for Datasets (Gebru et al.); Creative Commons CC0 / CC-BY 4.0

---

## Appendix A — Improvements applied

The following 25 improvements were identified during planning and are **already applied** in the body
above (not deferred). Each cites where it landed.

1. **Access-tier taxonomy made a first-class, gated enum** (§6/§7) — OPEN / OPEN-AGGREGATE /
   REGISTERED-OPEN / NON-COMMERCIAL / CONTROLLED, with explicit catalog actions, so "open vs
   controlled" is an auditable decision, not prose.
2. **Zero-tolerance "false-open" metric** (§4) — false-"open" classification is a P0 incident with a
   hard target of 0, making the worst failure mode measurable.
3. **Mixed-tier "open-portion-only" rule** (§6, §11 risks) — TCGA/ICGC-style resources are recorded
   only for their open tier, with a `mixed-tier` flag, preventing record bleed into controlled data.
4. **Per-assertion provenance (`fieldProvenance[]`)** (§6/§7) — provenance is enforced per field, not
   per record, satisfying "provenance on every assertion."
5. **License-rights breakdown booleans, each cited** (§6/§7) — permitsReuse / permitsDerivatives /
   commercialUseAllowed / redistributionAllowed / attribution / shareAlike, with "unknown ≠ true."
6. **COSMIC & OncoKB explicitly FLAGGED non-commercial** (§7) — named in the source classification,
   routed to a governance decision rather than silently included.
7. **Excluded-resources register** (§5/§7) — dbGaP/EGA/biobanks listed as "controlled — contact
   custodian" pointers (no record-level metadata), so users are warned, not misled into thinking
   controlled data is unavailable or, worse, that it is open.
8. **Metadata-only-by-construction privacy stance** (§7/§14) — identifiability prevented structurally
   (no per-patient rows ever), with the gate re-checking it; synthetic-only fixtures.
9. **Patient-facing layer isolated as high-risk and default-deferred** (§5/§8/§10) — gated behind
   oncologist **and** advocate sign-off + "not medical advice"; not built until reviewers secured.
10. **Completeness score that returns 0 until the gate passes** (§4) — no partial credit for ungated
    records; a "permits-derivatives" without a cited clause fails.
11. **Access-tier consistency linter as CI tests** (§8/§10) — e.g. `OPEN` may not carry
    `duaRequired: true`; `CONTROLLED` may never appear in the published catalog (CI-blocked).
12. **License snapshot with SHA-256 + Wayback** (§7/§15) — terms captured immutably so later source
    changes can't silently rewrite the record's basis.
13. **Freshness SLA + drift detection** (§9/§15) — every record carries a freeze date + refresh
    window; changed sources auto-flag into maintenance tasks.
14. **Static, dataless search UI** (§6/§14) — no backend store, no tracking; eliminates an entire class
    of privacy/abuse surface for a health-adjacent tool.
15. **Standardized cancer-type coding** (§6/§16) — OncoTree primary with NCIt/ICD-O-3 cross-refs +
    domain-reviewer check, reducing miscoding (critical for rare cancers).
16. **Honest `verifiedNeed: false` until an adopter is secured** (§2/§4/§11) — and a Steward role whose
    job is to convert "produced" into "used," aligning with "delivered, not merged."
17. **Cold-start pilot chosen for standalone usefulness** (§9) — a clearly-open, no-patient-data
    resource (DepMap / GDC open tier) so M0 yields a real record without a third-party dependency.
18. **Mandatory reviewer is a blocking prerequisite, not a parallel hire** (§8/§11) — must be filled
    before the M0 pilot is reviewed; indexing halts if no qualified reviewer exists.
19. **Refusal/abuse guardrails specialized to this domain** (§14) — explicitly refuses laundering
    controlled data as open, re-identification, bulk mirroring/re-derivation, and for-profit pipelines.
20. **SEER microdata vs aggregate distinction** (§7) — only SEER\*Explorer aggregate is indexed; SEER
    research microdata (signed agreement) is correctly classified out of scope.
21. **REGISTERED-OPEN tier for click-through resources** (§7) — AACR Project GENIE et al. handled
    precisely (free + registered + de-identified aggregate), not lumped with either OPEN or CONTROLLED.
22. **Outcome ledger with externally-verifiable reuse events** (§4/§15) — self-reported use excluded;
    each outcome is a committed artifact with channel + permalink + timestamp.
23. **Candidate backlog as a triage funnel** (§5/§10) — `COHORT-CATALOG.md` feeds per-cohort tasks
    only after the gate, so scale never bypasses verification.
24. **CC-BY-4.0 metadata / MIT code split with upstream attribution preserved** (§6/§7) — and an open
    question to decide CC0 for purely factual fields, avoiding accidental over-claiming of rights.
25. **DCAT/schema.org/Dataset cross-walk + FAIR alignment** (§6/§12) — so the catalog is interoperable
    with existing dataset-discovery ecosystems rather than a silo.

---

## Review sign-off

**Reviewer:** Planning author (senior staff engineer + TPM), self-review pass · **Date:** 2026-06-28

**Completeness check (PLAN_SPEC 17 sections):** All 17 required H2 sections present and in order —
Executive summary; Problem & beneficiaries; Goals and non-goals; Success metrics (outcomes); Scope;
Solution approach & architecture; Data, licensing & compliance; Quality, review & risk gates; Roadmap
& milestones; Work breakdown; Governance, roles & stakeholders; Dependencies & integrations; Risks &
mitigations; Security & privacy; Sustainability & maintenance; Open questions; References. ✔
(v0.2 adds two **additive** non-required H2 sections — "Competitive landscape & differentiation" and
"Adjacent opportunities" — without removing or reordering any required section.)

**Correctness fixes applied during review:**
- Confirmed the **Data, licensing & compliance** section **leads** with the binding cancer guardrails
  (open/aggregate/de-identified only; controlled out; NC FLAG; no medical advice; provenance) before
  any source classification. ✔
- Verified **no metric or DoD** rewards mere production; every "shipped"/success signal requires
  gate-pass + beneficiary use. ✔
- Verified the **mixed-tier rule** and **CI access-tier linter** are consistent across §6, §7, §8,
  §10, §11 (CONTROLLED can never enter the published catalog). ✔
- Verified **COSMIC/OncoKB → FLAG** and **dbGaP/EGA/biobanks → EXCLUDE** are stated consistently in
  §5, §7, §11, §12. ✔
- Confirmed `TASKS.md` mapping: catalog metadata → `deliverable: dataset`/`document`, code → `pr`,
  `verifiedNeed: false`, `lane: donated`, risk tiers (medium core / high patient-facing). ✔
- Confirmed patient-facing layer is **default-deferred** and never ships without oncologist **and**
  advocate sign-off + "not medical advice." ✔

**Residual items requiring a human decision** (tracked in §16 Open questions): confirm an
adopter/steward (unblocks `verifiedNeed`); finalize metadata license (CC-BY-4.0 vs CC0); decide
COSMIC/OncoKB inclusion-with-flag vs pointer-only; decide whether the patient layer is in the first
6-month window. **No fabricated partner or verified need has been asserted anywhere.**

**Verdict:** Plan is internally consistent, guardrail-compliant, and ready for maintainer review.

---

## Changelog — v0.2 (analysis merged)

This version merges the findings of `COMPETITIVE-ANALYSIS.md` (source of truth) into the plan,
surgically and additively. No guardrail was weakened; no partner, adopter, or verified need was
invented. Cancer guardrails (open/aggregate/de-identified only; controlled out; per-source license
verify; provenance-on-every-assertion; no medical advice) are preserved and, in places, strengthened.

**Correctness / schema / license / safety fixes applied (analysis §1, §6):**
- **`entryType` defined** (`patient-cohort | population-statistics | cell-model-collection |
  molecular-dataset`) with **conditional `aggregateCounts`**, so DepMap (cell lines) and GLOBOCAN
  (population stats) are no longer forced into a patient-cohort shape (gap A). [§6, Goals, M0]
- **Cross-repository cohort-identity / dedup rule** added (one record per distinct cohort; repositories
  are `hostingLocations[]`); all coverage metrics now count **distinct cohorts**, closing the
  "index the same cohort five times to hit ≥40/≥5" gaming hole (gap B). [§4, §6, M0/M1/M2]
- **Legal theory for our own metadata** stated (independently-verified factual descriptors + citation,
  not the custodian's data/prose), making `redistributionAllowed: false` compatible with an index entry
  and grounding the CC-BY-vs-CC0 decision (gap C). [§7, §6, Open questions]
- **Index ≠ redistribute** clarified so the publish linter accepts `REGISTERED-OPEN` /
  no-redistribution records (e.g. GENIE) instead of wrongly rejecting them (gap H). [§6, M2]
- **Concrete drift signal** specified (terms-URL content hash + SLA re-check + first-class
  `termsChangedSinceSnapshot` record state) (gap D). [§6, §15, M2]
- **GLOBOCAN/IARC "aggregate ≠ automatically open"** caveat added (attribution + use conditions verified,
  not assumed); **`generationContext`** (funder/consent/date-range) field added (gap H). [§7, §6]
- **cBioPortal public studies added as a seed source**; excluded register expanded with TCGA-controlled/
  germline, GTEx, TOPMed, MVP (gap G). [§7, §12, M1]
- **Mandatory (not sampled) second reviewer** for `REGISTERED-OPEN` and `mixed-tier` records — the
  classes most likely to be mislabeled (gap F). [§8, M1]
- **Coverage-representativeness metric** added (rare-cancer / pediatric-AYA / non-US / modality spread)
  so the catalog isn't 40 easy US-gov resources (gap F/G). [§4, M2]

**Sibling differentiation operationalized (analysis §1E, §4):**
- New **"Competitive landscape & differentiation"** section with a **written boundary + shared schema
  contract**: this project = breadth / access-tier + license layer; `cancer-dataset-datasheets` = depth;
  `ewing-open-data-catalog` = disease vertical (consumes this schema); `cancer-data-dictionaries` =
  variable-level. **Win condition: be the canonical access-tier vocabulary the siblings reuse.**

**Strategy integrated (analysis §2–§7):**
- **Competitive landscape** documented (NCI CRDC, cBioPortal, OmicsDI, re3data/FAIRsharing, dbGaP/EGA)
  with the differentiator = a cancer-scoped cross-repository **access-tier + normalized license-rights
  matrix with clause-level provenance + immutable snapshots** — the auditable "is it open, and exactly
  what may I do?" layer.
- **Claude API leverage folded into architecture** (cohort-metadata extraction with cited
  `fieldProvenance`; access/license-language normalization into the enum as a quoted proposal; drift
  triage; coding/lint assist) — with hard guardrails: tier/license determinations are always
  **FLAG-for-review, never PASS**; never fabricate terms/citations ("unknown ≠ true" binds the model);
  **never ingest controlled/identifiable data**; no de-id/re-id/linkage; no medical advice.
- **Optimizations folded into the Roadmap** (entryType + dedup in M0; second-reviewer + seed-list
  expansion in M1; concrete drift signal + coverage target + schema.org/DCAT JSON-LD in M2).
- New **"Adjacent opportunities"** section (cross-cohort access-terms API; "is this cohort open?" MCP
  server; shared access-tier vocabulary package; drift-watcher utility; controlled-access concierge).
- **Open questions merged** with the analysis's maintainer questions, each annotated with its v0.2
  resolution or default.

**Preserved unchanged:** vision, beneficiary case, the binding cancer guardrails and their ordering,
the access-tier taxonomy, metadata-only/no-patient-data invariant, mixed-tier open-portion-only rule,
synthetic-only fixtures, `verifiedNeed: false` honesty, Appendix A. `COMPETITIVE-ANALYSIS.md` left
unchanged. Version bumped 0.1.0 → 0.2.0.
