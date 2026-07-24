# open-cohort-catalog

> Cancer research generates an enormous, growing body of **open** data — gene-expression series, open-tier somatic mutation calls, proteogenomics, cell-line dependency maps, and aggregate incidence/outc  ·  **Risk tier:** med  ·  **Status:** planning

Cancer research generates an enormous, growing body of **open** data — gene-expression series, open-tier somatic mutation calls, proteogenomics, cell-line dependency maps, and aggregate incidence/outcome statistics — scattered across dozens of repositories (GDC, GEO, EMBL-EBI, PRIDE, DepMap, SEER, IARC, and more). The practical problem is **not** that the data is missing; it is that a researcher (or an informed patient, advocate, or student) cannot easily answer four questions about any given cohort: *What is in it? Who can use it, and under what license? Is it truly open, or is it controlled-access behind an application? How do I cite it and trust its provenance?* The access terms are inconsistent, the licenses are heterogeneous (public-domain US government data sits next to non-commercial custom terms and click-through registration), and the boundary between **open-access** and **controlled-access** tiers *within the same resource* (e.g. TCGA has both) is routinely misunderstood — a misunderstanding that, at worst, leads people to treat identifiable or controlled data as if it were freely reusable.

**Definition of shipped:** license + PII), fully source-cited (completeness ≥ 90/100), schema-valid, CI-green, reviewer-approved, published in the catalog, AND demonstrably usable by a beneficiary** — i.e. the catalog (or the record) is adopted/queried/cited by a named third party, with the Steward's outco

This is a **Hee-Lee Oss** good-deed project. Contributors pull a task, do it with their own coding agent, and open a PR. Get started: https://github.com/Hee-Lee-Oss-Projects/hee-lee-oss-downloads

## Plan
- [PLAN.md](./PLAN.md) — robust enterprise plan (vision, architecture, roadmap, risks; includes an applied-improvements appendix + review sign-off)
- [TASKS.md](./TASKS.md) — schema-mapped task backlog
- [tasks/](./tasks/) — ready-to-pull task JSON(s)

## Contribute
```bash
hee-lee-oss browse
hee-lee-oss next --repo Hee-Lee-Oss-Projects/open-cohort-catalog --no-fork
```

## Licensing & review
- Open license (see PLAN.md).
- Risk tier **med** — deeds are *delivered, not merged*; a domain reviewer (and expert sign-off for any high-stakes content) must approve before merge.

> Planning stage; no adopting partner secured yet (`verifiedNeed: false` on delivery-dependent tasks).
