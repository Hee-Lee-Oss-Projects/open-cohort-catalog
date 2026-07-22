# Competitive & Improvement Analysis — open-cohort-catalog

Scope reviewed: `PLAN.md` (v0.1.0, 2026-06-28) + `TASKS.md` (M0 tables and acceptance criteria).
Project thesis: a searchable, license-aware, access-tier-first **index of open cancer cohorts**
(metadata only; no patient data). Risk: medium core / high patient layer.

---

## 1. Correctness & completeness review of PLAN.md

The plan is unusually disciplined: access-tier-first, provenance-on-every-assertion, mixed-tier
"open-portion-only", zero-tolerance false-"open" metric, NC sources (COSMIC/OncoKB) flagged not
silently included, synthetic-only fixtures. Those are the right spine. Concrete gaps and errors:

**A. "Cohort" vs "dataset" is never actually defined — and the project name presumes the harder one.**
The plan slides between "cohort," "study," "dataset," and "resource" (e.g. DepMap and GEO *series* are
datasets/resources, not cohorts; TCGA is a program; AACR GENIE is a registry). A **cohort** is a
defined group of subjects; a **dataset** is a data artifact. The unit of cataloging drives dedup,
counts, and cross-refs. Right now `aggregateCounts {patients, samples}` assumes a patient cohort, but
DepMap (cell lines) and GLOBOCAN (population statistics) have no patient cohort at all. **Fix:** define
the indexed unit explicitly (recommend a typed `entryType`: `patient-cohort | population-statistics |
cell-model-collection | molecular-dataset`) and make `aggregateCounts` shape depend on it. Without
this, the schema will misrepresent half the seed sources.

**B. Dedup / granularity / "same cohort, many repositories" is unaddressed.** TCGA appears in GDC,
cBioPortal, the legacy ICGC portal, Genomic Data Commons, and Xena; GENIE appears in Synapse *and*
cBioPortal. The plan has `crossRefs[]` but **no canonical-identity / dedup rule** ("one record per
cohort, repositories are locations"). Without it the catalog will double-count and contradict the
"≥40 cohorts across ≥5 repositories" metric (you can hit it trivially by indexing the same cohort five
times). **Fix:** define cohort identity keys and a "primary custodian vs mirror" model; the count
metric should be distinct cohorts, not records.

**C. "Open" vs license-permits-derivatives conflation — the core risk is subtler than the plan admits.**
The plan's indexable criterion requires `permitsDerivatives: true` "for the metadata we publish."
But factual metadata (counts, assay names, license id) is generally **not copyrightable** and needs no
license grant, while the *source's own license* may forbid redistributing **their** curated metadata
verbatim. AACR GENIE's terms explicitly prohibit **redistribution of the data**
(https://www.aacr.org/professionals/research/aacr-project-genie/aacr-project-genie-frequently-asked-questions/)
— fine for an index, but the plan should state the legal theory clearly: *we publish independently-verified
factual descriptors + a citation, not the custodian's copyrighted prose or data.* Otherwise the
"CC-BY vs CC0 for our metadata" open question is unanswerable. This is a real correctness gap, not pedantry.

**D. Access-terms currency / drift is under-specified operationally.** Good that snapshots + Wayback +
freshness SLA exist. But access tiers and licenses **change** (ICGC's legacy DCC portal is being
retired and 25K data moved; GEO per-series terms vary; GDC reorganizes). The plan says "drift
detection" but never says **how** a metadata-only catalog detects an upstream terms change without an
authoritative machine-readable terms feed (most sources have none). **Fix:** specify the drift signal
(HTTP content hash of the terms URL + periodic human re-verification on the SLA), and make "terms
changed since snapshot" a first-class record state, not just a maintenance task.

**E. Scope overlap with siblings is asserted but not operationalized.** The plan references
`open-data-datasheets` for house conventions but does **not** differentiate against the named cancer
siblings (`cancer-dataset-datasheets`, `ewing-open-data-catalog`, `cancer-data-dictionaries`). Risk of
three Hee-Lee Oss projects all "indexing cancer datasets." The clean split (make it explicit in §3/Scope):
this project = **breadth index keyed on access-tier + license** (the "is it open, and on what terms?"
layer); `cancer-dataset-datasheets` = **depth** (rich per-dataset datasheet narrative à la
Datasheets-for-Datasets); `ewing-open-data-catalog` = **one disease vertical** (could consume this
schema); `cancer-data-dictionaries` = **variable/field-level** harmonization. Without a written
boundary + a shared schema contract, these will duplicate effort and diverge on tier vocabulary.

**F. Weak / gameable metrics.**
- "≥40 verified entries / ≥3 external reuse events" — reuse events are the real outcome but "3" is
  arbitrary and the *adopter* metric (≥1 steward) is the actual gate on "delivered." Reuse-event
  definition is good (externally verifiable) but the bar should be **adopter-first**, not entry-count-first.
- "0 false-open errors" is measured by "independent reviewer disagreement rate on an audited sample" —
  but a sample can miss the rare catastrophic error. Add **100% second-reviewer on any `REGISTERED-OPEN`
  or `mixed-tier` record** (the classes most likely to be mislabeled), not just a sample.
- No metric for **coverage representativeness** (rare cancers, non-US sources, pediatric). A catalog of
  40 easy US-gov open resources is low-value; the plan's own beneficiary case (rare-cancer advocates)
  needs a coverage/diversity metric.

**G. Coverage / source-list gaps.** The seed list is solid but omits high-value open cancer resources:
**cBioPortal public studies** (~300 studies, mostly open;
https://www.cbioportal.org/datasets), **UCSC Xena**, **Genomic Data Commons** non-TCGA programs
(CGCI, HCMI, MMRF open tiers), **ICGC 25K legacy open data**, **CCLE**, **Genomics of Drug Sensitivity
in Cancer (GDSC)**, **cptac**, **Human Tumor Atlas Network (HTAN)**, **proteomics via ProteomeXchange**.
Also missing from the *excluded* register: **All of Us**, **MVP**, **Genomics England** are named but
**TCGA controlled (germline/BAM)**, **GTEx**, and **TOPMed** should be explicit. cBioPortal is a notable
omission given it is the closest thing to an existing open-cancer-cohort browser.

**H. Minor correctness items.**
- `REGISTERED-OPEN` + GENIE: GENIE is click-through registered **and** has a no-redistribution clause —
  the schema's `redistributionAllowed` will be `false` for GENIE, which is correct, but the plan should
  confirm a `REGISTERED-OPEN` record with `redistributionAllowed:false` is still *indexable* (it is —
  index ≠ redistribute). Make that explicit so the linter doesn't reject it.
- DepMap is **CC-BY-4.0 cell-line** data (not patient data) — correct, and a good cold-start pilot, but
  it stresses point A (it has no "patients").
- `OPEN-AGGREGATE` (GLOBOCAN/SEER*Explorer/CDC WONDER): IARC GLOBOCAN terms are **not** simply "open";
  they require attribution and have use conditions — verify, don't assume "aggregate = open."
- The schema lacks a field for **funder / consent basis / data-generation date range**, useful for
  beneficiaries and cheap to capture.

---

## 2. Competitive landscape (real competitors / adjacent, cited)

**Direct-adjacent (cancer dataset/cohort discovery):**

- **NCI Cancer Research Data Commons (CRDC) / General Commons.** The 800-lb gorilla for *cancer*
  specifically. Explicitly spans open + controlled; General Commons portal searches submitted studies
  with no login, then routes controlled access via dbGaP/DCF.
  (https://datacommons.cancer.gov/cancer-research-data-commons,
  https://datacommons.cancer.gov/repository/general-commons,
  https://pmc.ncbi.nlm.nih.gov/articles/PMC11063687/)
  *Strengths:* authoritative, funded, comprehensive, the source of truth for tier status.
  *Gaps:* organized by node/repository not by a normalized cross-source **license-rights breakdown**;
  no single "is this open and on exactly what license terms?" answer across non-NCI sources (GEO, EBI,
  COSMIC, OncoKB); US/NCI-centric.

- **cBioPortal public studies list.** ~300 curated cancer genomics studies, ~100k+ samples, mostly open.
  (https://www.cbioportal.org/datasets, https://pmc.ncbi.nlm.nih.gov/articles/PMC3956037/)
  *Strengths:* the closest existing "browse open cancer cohorts" surface; harmonized; widely used.
  *Gaps:* it's an *analysis portal*, not a license/access-terms catalog — tier and license terms aren't
  first-class facets; only covers what's been ingested into cBioPortal; no provenance-per-assertion.

- **ICGC / ICGC ARGO data portal.** Two-tier (open: SSMs, CNV, expression, methylation; controlled via
  DACO). (https://docs.icgc-argo.org/docs/data-access/icgc-25k-data,
  https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3395593/)
  *Strengths:* clear tier model, global tumor coverage. *Gaps:* single-consortium; legacy DCC portal
  retiring (drift risk illustration); not a cross-repository index.

**Generic dataset/registry catalogs (broader, shallower on cancer tiers):**

- **OmicsDI (EMBL-EBI).** Aggregates omics datasets (proteomics/genomics/transcriptomics/metabolomics)
  into a unified search with common metadata. (https://www.omicsdi.org/,
  https://omicsdi.readthedocs.io/en/latest/introduction.html)
  *Strengths:* real cross-repository aggregation, API, established. *Gaps:* discovery-oriented; **license
  and access-tier are not normalized rights facets**; not cancer-focused; no per-assertion provenance.

- **re3data.org.** Registry of ~3000+ research data *repositories* (not datasets/cohorts), with policy/
  access metadata at the repo level. (https://www.re3data.org/)
  *Strengths:* canonical repository registry, access-category metadata. *Gaps:* repository granularity,
  not cohort; no cancer cohort records; no per-cohort license-rights breakdown.

- **FAIRsharing.org.** Curated registry of databases, standards, and policies (incl. licences) with
  stable IDs. (https://www.nature.com/articles/s41587-019-0080-8,
  https://www.re3data.org/repository/r3d100010142)
  *Strengths:* community-maintained, links DBs↔standards↔policies, license metadata. *Gaps:* database-level
  not cohort-level; broad not cancer; license info is descriptive, not a normalized rights matrix.

- **Google Dataset Search / DataCite.** Crawls schema.org/Dataset markup across the web.
  (https://datasetsearch.research.google.com via
  https://support.datacite.org/docs/how-do-i-expose-my-datasets-to-google-dataset-search,
  https://arxiv.org/pdf/2006.06894)
  *Strengths:* enormous reach, zero-curation. *Gaps:* only as good as publishers' markup; **no verified
  access tier, no rights normalization, no cancer focus, no provenance auditing** — exactly the trust
  gap this project targets.

- **Maelstrom Research Catalogue.** Population-cohort (epidemiology) catalogue: studies, variables,
  harmonization potential. (https://www.maelstrom-research.org/page/catalogue,
  https://pmc.ncbi.nlm.nih.gov/articles/PMC6057635/)
  *Strengths:* mature cohort-catalog model, variable-level search, harmonization. *Gaps:* epidemiology
  not cancer-molecular; not license/access-tier-first; many catalogued cohorts are controlled.

- **CINECA cohort network + synthetic cohorts.** Federated human-cohort infrastructure (GA4GH Beacon/
  Search), with freely usable **synthetic** cohorts (incl. cancer use case).
  (https://www.cineca-project.eu/cineca-synthetic-datasets,
  https://www.cineca-project.eu/virtual-platform/federated-data-access)
  *Strengths:* federated discovery standards, synthetic data for tooling. *Gaps:* access is fundamentally
  controlled/federated; not an open-cohort license index.

- **dbGaP & EGA.** Authoritative controlled-access catalogs/browsers for individual-level genomic data.
  (https://academic.oup.com/nar/article/45/D1/D819/2605794,
  https://www.ncbi.nlm.nih.gov/books/NBK570242/)
  *Role here:* these are the **excluded** set — the project should *point* to them, not index them.
  Their existence is why an "is it open vs controlled?" disambiguation layer has value.

- **OHDSI/OMOP & EHDEN data-partner catalogue.** ~187 EHDEN data partners mapping to OMOP; OHDSI network
  ~810M patient records, federated. (https://www.ehden.eu/datapartners/,
  https://www.ohdsi.org/web/wiki/doku.php?id=resources:data_network)
  *Strengths:* huge federated EHR network, a model for a data-partner directory. *Gaps:* observational
  EHR not open cancer-molecular cohorts; data are behind partners, not open.

**Takeaway:** No incumbent offers a **cancer-specific, cross-repository, access-tier-first, license-rights-
normalized, per-assertion-provenanced index of *open* cohorts.** CRDC is authoritative but NCI-scoped and
not a normalized rights matrix; OmicsDI/FAIRsharing/re3data/Google are broad but tier/license-shallow;
cBioPortal is the closest browse surface but is an analysis tool, not a terms catalog.

---

## 3. Gaps we can fill

1. **A normalized, cited license-rights matrix** (`permitsReuse / derivatives / commercial /
   redistribution / attribution / shareAlike`, each with a clause citation) across heterogeneous sources
   — nobody publishes this in machine-readable, comparable form for cancer cohorts.
2. **An authoritative open-vs-controlled disambiguation** for *mixed-tier* resources (TCGA, ICGC, GDC),
   stating "the open portion is X under license Y; the controlled portion is out of scope, contact Z" —
   the single most error-prone thing users get wrong.
3. **Per-assertion provenance + immutable snapshots** (SHA-256 + Wayback) so the answer is auditable and
   survives upstream drift — a trust property no incumbent provides.
4. **Cross-repository identity** for cohorts that appear in 3-5 portals (one record, many locations) —
   collapses the duplication users currently navigate manually.
5. **An explicit "excluded but pointed-to" register** so controlled resources are *visibly* controlled,
   not silently absent (prevents the "I assumed it was downloadable" failure).
6. **NC-license clarity** (COSMIC, OncoKB) as a first-class FLAG state with the actual commercial/
   redistribution constraints spelled out — usually buried in custom T&Cs.

---

## 4. Differentiators to win

- **Access-tier + license-rights as the primary key**, not a footnote — the catalog answers "is this
  open, and exactly what may I do with it?" which CRDC/cBioPortal/OmicsDI answer only partially.
- **Provenance-complete and auditable** (clause-level citations + snapshots): defensible, contributor-
  scalable, and survives source drift. This is the trust moat vs Google Dataset Search's crawl.
- **Cancer-scoped + tier-rigorous**: deep enough to encode mixed-tier and NC nuances generic registries
  flatten.
- **Static, data-free, MIT+CC-BY artifact**: forkable, embeddable, no infrastructure to adopt — lowers the
  bar for a steward to host it vs standing up a CRDC node.
- **vs Hee-Lee Oss siblings (make this explicit and load-bearing):** this is the **breadth/access-tier layer**;
  `cancer-dataset-datasheets` is the **depth/datasheet-narrative layer**; `ewing-open-data-catalog` is a
  **single-disease vertical** that should *consume this schema*; `cancer-data-dictionaries` is the
  **variable-level** layer. One shared access-tier vocabulary + schema contract across all four turns
  overlap into a stack instead of four competitors. **Win condition: be the canonical access-tier
  vocabulary the siblings reuse.**

---

## 5. Claude API leverage

**Where Claude clearly helps (with provenance + human gate):**
1. **Draft cohort metadata extraction from public landing pages / data dictionaries / papers** — Claude
   reads the GEO series page, GDC project page, DepMap release notes, or a methods section and proposes
   the structured record (counts, assays, modalities, custodian, citation string), **emitting a candidate
   `fieldProvenance` (source URL + the exact quoted clause) for every field**. Human verifies; Claude never
   commits. Huge time-saver on the most tedious step.
2. **Normalize heterogeneous access/license language into the controlled vocabulary** — map free-text
   terms ("freely available for academic use," "requires registration," "no commercial use") into the
   enum + the rights-boolean matrix **as a proposal with the quoted basis**, flagging ambiguity rather
   than guessing. Especially valuable for NC/custom terms.
3. **Harmonize cancer-type coding** — propose OncoTree/NCIt/ICD-O-3 codes from free-text disease names,
   with confidence + the mapping rationale, for domain-reviewer confirmation (helps rare-cancer coverage).
4. **Drift triage** — diff a fresh terms page against the stored snapshot and summarize *what changed*
   (tier? license? redistribution clause?), routing material changes to a human re-verification task.
5. **Consistency/lint assistant** — explain *why* a record fails the access-tier linter (e.g. "OPEN but
   `duaRequired:true`") in reviewer-friendly language; draft the excluded-register pointer text.

**Where Claude must NOT decide (hard guardrails):**
- **Final access-tier and license determinations stay human-verified.** Claude proposes + cites; the
  Access/License/PII reviewer decides. A model-only "OPEN" must never reach the published catalog
  (the P0 failure mode). Treat Claude output as `FLAG-for-review`, never `PASS`.
- **Never fabricate terms or citations.** Every Claude-asserted clause must resolve to a real, quoted
  source span; a record fails if the cited clause can't be re-found. Forbid Claude from inferring a
  permissive default when terms are silent ("unknown ≠ true" must bind the model too).
- **Never ingest controlled/identifiable data.** Claude is pointed only at *public, cohort-level*
  descriptor pages — never at controlled data, per-patient rows, or anything behind a DUA; if a page
  surfaces identifiable content, halt-and-discard (no summarizing it into the record).
- **No de-identification / re-identification / linkage reasoning** by Claude — out of scope entirely.
- **No medical advice** — Claude must not generate clinical interpretation; patient-facing text stays
  oncologist+advocate-gated.
(Reference Anthropic model/pricing/caching guidance via the `claude-api` skill before implementing; the
extraction step benefits from prompt caching the schema + gate rubric.)

---

## 6. Ten concrete optimizations

1. **Define `entryType`** (`patient-cohort | population-statistics | cell-model-collection |
   molecular-dataset`) and make `aggregateCounts` shape conditional on it — fixes the DepMap/GLOBOCAN
   "no patients" inconsistency (gap A).
2. **Add a cohort-identity / dedup rule** ("one record per cohort; repositories are locations") and make
   the coverage metric count *distinct cohorts*, not records (gap B).
3. **Publish the legal theory for the metadata itself** ("independently-verified factual descriptors +
   citation, not the custodian's copyrighted prose/data"), resolving CC-BY-vs-CC0 (gaps C, §16).
4. **Write the sibling boundary + shared schema contract** into Scope: tier vocabulary owned here, reused
   by `ewing-open-data-catalog` / `cancer-dataset-datasheets` / `cancer-data-dictionaries` (gap E, §4).
5. **Make second-reviewer mandatory (not sampled) for `REGISTERED-OPEN` and `mixed-tier` records** — the
   classes most likely to be mislabeled (gap F).
6. **Add a coverage/representativeness metric** (rare cancers, pediatric/AYA, non-US sources, modality
   spread) so the catalog isn't 40 easy US-gov resources (gap F/G).
7. **Specify the drift signal concretely**: store a content hash of each terms URL, re-check on the SLA,
   and add a first-class `termsChangedSinceSnapshot` record state (gap D).
8. **Emit schema.org/Dataset + DCAT JSON-LD** for every record so the catalog is itself discoverable via
   Google Dataset Search/DataCite — distribution for free, and d-foods the FAIR claim (turns a competitor
   into a channel).
9. **Ship a read-only access-terms API + JSON export** (not just a static site) so downstream tools and
   siblings can query "is cohort X open?" programmatically (see §7).
10. **Expand the seed list** to include cBioPortal public studies, UCSC Xena, GDC non-TCGA open programs
    (HCMI/CGCI/MMRF), CCLE/GDSC, HTAN, ProteomeXchange/PRIDE cancer sets; and make the excluded register
    explicitly name TCGA-controlled/germline, GTEx, TOPMed (gap G).

---

## 7. Parallel & perpendicular spin-offs

- **Cross-cohort access-terms API** (`GET /cohort/{id}/terms` → tier + rights matrix + provenance) — the
  catalog's most reusable output; lets pipelines gate on license before touching data.
- **"Is this cohort open?" MCP server** — wraps the access-terms API as a tool any agent/IDE can call to
  check tier+license before suggesting a download; natural Anthropic-ecosystem distribution and a sharp,
  demoable artifact.
- **Shared Hee-Lee Oss access-tier vocabulary package** consumed by `ewing-open-data-catalog` (disease vertical),
  `cancer-dataset-datasheets` (depth datasheets reference this record by id), and `cancer-data-dictionaries`
  (variable-level layer links up to the cohort record). Turns four projects into one stack.
- **License-snapshot/drift watcher as a standalone service** — useful well beyond cancer (any FAIR catalog
  needs "did the terms change?"); could be contributed to FAIRsharing/re3data communities.
- **schema.org/Dataset enrichment bot** — emits markup for under-described open cancer resources, improving
  their Google Dataset Search visibility (a public good that also seeds the catalog).
- **An "excluded but here's the door" controlled-access concierge** — for dbGaP/EGA/CRDC controlled
  studies, a pointer record with the *exact* application path (DACO, dbGaP authorized access), so users
  aren't misled into thinking the data is open *or* unavailable.

---

## 8. Open questions for the maintainer

1. **What is the indexed unit of record** — cohort, study, dataset, or resource — and how do
   multi-repository appearances dedup to one canonical record?
2. **What is the legal basis** for publishing our *own* metadata (CC-BY vs CC0), given some sources
   (GENIE) forbid redistributing *their* data/metadata? (Drives §16 license decision.)
3. **Sibling boundaries:** is this project the *owner* of the shared access-tier vocabulary that
   `ewing-open-data-catalog` / `cancer-dataset-datasheets` / `cancer-data-dictionaries` consume? If not,
   how do we avoid four divergent tier taxonomies?
4. **Coverage target:** breadth-first (many easy open US-gov resources) or depth-first (fewer, including
   hard rare-cancer / non-US / NC-flagged resources)? The beneficiary case argues depth/diversity.
5. **Is cBioPortal in or out of scope** as a source — it's the closest existing browse surface and a
   notable omission from the seed list.
6. **Drift detection authority:** absent machine-readable terms feeds, what is the concrete change-signal
   and re-verification cadence per source?
7. **Adopter-first sequencing:** should M0 prioritize securing the steward *before* indexing 40 entries,
   given "delivered" hinges on adoption, not entry count?
