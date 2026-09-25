# Infrastructure, Data Standards, and Data Management for Federated Learning in Biological Countermeasures

**Status:** draft for review · **Scope:** platform, data, and governance standards for a multi-party federated learning (FL) program supporting medical/biological countermeasure R&D (vaccines, therapeutics, diagnostics, surveillance analytics).

This document defines *core goals*, the *standards* that operationalize them, and the *controls* that make the pipelines safe, secure, and reliable. It is deliberately organization-agnostic: each section states the goal, the standard, the control, and how the control is verified.

---

## 1. Why federated, and what that changes

Biological countermeasure data is high-value and rarely poolable: it sits in hospital systems, national labs, public-health agencies, CROs, and commercial sponsors, under different legal bases (HIPAA/Common Rule, GDPR, national biosecurity law, export control, sponsor IP). Federated learning lets a model travel to the data instead of the reverse.

That inverts the usual assumptions and creates the problems this standard exists to solve:

| Centralized ML assumption | Federated reality |
|---|---|
| One schema, one curation team | N schemas, N curation cultures, no shared eyes on raw data |
| You can look at the data to debug | You can never look; you debug through statistics and contracts |
| One trust boundary | Every participant is both a dependency and a potential adversary |
| Uptime is one platform's problem | Availability is the *product* of all participants' reliability |
| Data quality is inspectable | Data quality must be *asserted, tested remotely, and attested* |
| Model output is the only artifact | Gradients, metrics, and even loss curves are potential data leaks |

FL reduces raw-data movement risk. It does **not** reduce — and in some respects raises — model-mediated disclosure risk, data-quality risk, and dual-use risk. Design for that explicitly.

---

## 2. Core goals (the things you are actually managing to)

Five goals, each with measurable objectives. Everything later in the document traces to one of these.

### G1 — Scientific validity
A model is only useful if its performance transfers to sites that never contributed to it.

- **O1.1** Every released model reports performance on ≥2 held-out sites not used in training, with subgroup breakdowns (age, sex, geography, assay platform, specimen type).
- **O1.2** Every release is reproducible from recorded inputs: pinned code, container digests, round schedule, aggregation config, seeds, per-site data snapshot hashes.
- **O1.3** No release without a documented, pre-registered evaluation plan; post-hoc metric selection is a release blocker.

### G2 — Data protection and sovereignty
Participants must be able to join without surrendering control of raw data or violating their legal basis.

- **O2.1** Raw records never leave a participant's boundary. Only model deltas, encrypted aggregates, and approved summary statistics cross it.
- **O2.2** Every cross-boundary flow has a named legal basis and data-use agreement reference recorded in metadata.
- **O2.3** Privacy budget (if using DP) is tracked per data subject population, not per run, and is enforced by the platform rather than by convention.

### G3 — Security and integrity
Assume a participant, a network path, or a dependency is compromised.

- **O3.1** No single participant can move the global model beyond configured bounds in a single round.
- **O3.2** Every artifact — container, dataset snapshot, model delta, aggregate, release — is signed and verifiable back to an identity.
- **O3.3** Mean time to isolate a misbehaving participant: minutes, via a documented and *rehearsed* procedure.

### G4 — Reliability and operability
Countermeasure work is time-critical; the pipeline must degrade gracefully, not stall.

- **O4.1** Training completes with a defined quorum (e.g. ≥70% of enrolled sites, cohort-weighted), not all sites.
- **O4.2** Any round is resumable from checkpoint; no round loses more than one round of work.
- **O4.3** Defined SLOs per pipeline stage, with error budgets that gate feature work.

### G5 — Responsible use and accountability
Biological countermeasure work is inherently dual-use adjacent. Governance is part of the infrastructure, not a document beside it.

- **O5.1** Every model has an approved intended-use statement and an explicit out-of-scope list before first release.
- **O5.2** A standing review body signs off on new data domains, new model classes, and any release to a wider audience.
- **O5.3** Complete, tamper-evident audit trail: who trained what, on which data, under which approval, and who consumed the output.

**Non-goals** (state these; they prevent scope drift): the platform does not design biological agents or interventions, does not perform wet-lab automation control, and does not make clinical or regulatory decisions autonomously. It produces evidence for humans who do.

---

## 3. Reference architecture

Five planes. The discipline is that each plane has one job and a stated trust boundary.

```
┌───────────────────────────────────────────────────────────────────────┐
│ GOVERNANCE PLANE                                                      │
│  enrollment & identity · data-use agreements · approvals & sign-off   │
│  privacy-budget ledger · model registry · immutable audit log         │
└───────────────────────────────────────────────────────────────────────┘
                 │ policy decisions, attestations, approvals
┌───────────────────────────────────────────────────────────────────────┐
│ ORCHESTRATION PLANE  (coordinator — untrusted with raw data)          │
│  round scheduling · participant selection · secure aggregation        │
│  robust aggregation & bounds enforcement · checkpointing              │
└───────────────────────────────────────────────────────────────────────┘
        ▲ signed deltas / encrypted shares          ▼ signed global model
┌───────────────────────────────────────────────────────────────────────┐
│ PARTICIPANT PLANE  (per site, inside the site's boundary)             │
│  local ingest → harmonization → QC gates → snapshot → local train    │
│  local eval · egress filter · local audit log                        │
└───────────────────────────────────────────────────────────────────────┘
┌───────────────────────────────────────────────────────────────────────┐
│ EVALUATION PLANE  (independent of training)                           │
│  held-out site benchmarks · drift & subgroup monitors · red team      │
└───────────────────────────────────────────────────────────────────────┘
┌───────────────────────────────────────────────────────────────────────┐
│ CONSUMPTION PLANE                                                     │
│  model serving · decision support · reporting · feedback capture      │
└───────────────────────────────────────────────────────────────────────┘
```

Architectural rules:

1. **The coordinator is untrusted with data.** It sees only what secure aggregation and egress filtering allow. Design as if it will be breached.
2. **Egress is a chokepoint, not a convention.** Each participant runs an egress filter that enforces an allowlist of artifact types and shapes. Anything unrecognized is dropped and alerted, not passed.
3. **Evaluation must not report to training.** Different owners, different credentials, separate held-out data. A team that can tune against the benchmark does not have a benchmark.
4. **Reproducibility is infrastructure.** Content-addressed artifacts, pinned container digests, a run manifest per round. If a result cannot be regenerated, it cannot be released.
5. **Governance is enforced in code paths.** An approval that exists only in a wiki page is not a control.

---

## 4. Data standards

The hardest part of federated work is not the learning; it is agreeing what a row means at every site.

### 4.1 Adopt standards, do not invent them

Bind to established representations wherever one exists, and record which version you bound to:

| Domain | Use |
|---|---|
| Clinical / EHR | HL7 FHIR resources; OMOP CDM for observational analytics |
| Clinical trials / submissions | CDISC SDTM, ADaM, SEND |
| Laboratory / diagnostics | LOINC for tests, SNOMED CT for findings, UCUM for units |
| Sequence and genomic data | INSDC/GA4GH conventions; VCF; GA4GH Phenopackets, DRS, Passports |
| Imaging | DICOM (with de-identification profile explicitly named) |
| Assay and experiment description | MIAPPE / MINSEQE-style minimum-information checklists; Investigation–Study–Assay structure |
| Terminology for pathogens/taxa | NCBI Taxonomy identifiers, not free-text names |
| Provenance | W3C PROV; in-toto attestations for pipeline steps |
| Findability | Persistent identifiers (DOI/ARK), DCAT catalog records, FAIR principles as the acceptance bar |

Rule: **no free-text where a code system exists**, and every coded field carries `(system, version, code)`. Unmapped values go to an explicit `unmapped` queue with the original string retained locally — never silently coerced.

### 4.2 The Feature Contract

The central artifact of a federated program. One versioned, machine-readable contract per model family, authored centrally, enforced locally.

Each field specifies:

- logical name, semantic type, and unit (UCUM)
- source standard and code system with version
- permissible values / range, and the action on violation (reject row, null with flag, clamp)
- missingness policy: is missing informative, and how is it encoded
- normalization/derivation, stated as executable code, not prose
- required or optional, and behavior when absent
- sensitivity class and whether it may influence egressed statistics

Contracts are semantically versioned. A breaking change forces a new model lineage rather than a silent reinterpretation of history. Every training round records the contract version it ran under; a site whose validator reports a different version is excluded from the round.

### 4.3 Harmonization happens at the edge

Each participant runs the same containerized harmonization job: source → contract-conformant local dataset. Benefits: site-specific mapping logic stays with the people who understand the source system; the platform sees a single schema; mapping bugs are diagnosable from conformance reports rather than from raw data.

Each site maintains a **mapping specification** — a reviewed, version-controlled document plus code that says exactly how its local codes become contract fields. Treat unreviewed mapping changes as a production change, because they are.

### 4.4 Metadata: what must accompany every dataset snapshot

Dataset-level:

- persistent identifier, snapshot hash, creation time, contract version
- provenance: source systems, extraction window, harmonization image digest
- cohort definition (inclusion/exclusion as executable query, not prose)
- population descriptors sufficient for representativeness assessment, at a granularity that does not itself identify anyone
- collection context: assay platforms, instruments and versions, specimen handling, known batch structure
- legal basis, consent scope, data-use agreement reference, retention and deletion date
- sensitivity classification and export-control determination
- known limitations and defects, written by the site

Record-level (only what the contract permits): stable pseudonymous key, timestamps at a defined resolution, batch identifiers, quality flags.

**Batch structure is first-class metadata.** In multi-site biological data, site, instrument, reagent lot, and collection period are the leading source of spurious signal. If you cannot name the batch variables, you cannot claim the model learned biology rather than logistics.

### 4.5 Quality gates (pass/fail, automated, reported as metadata)

Every snapshot passes these before a site may join a round:

1. **Schema conformance** — types, units, code systems, required fields.
2. **Referential integrity** — keys resolve; no orphan records.
3. **Range and plausibility** — physiologically and physically implausible values flagged.
4. **Completeness** — missingness per field within contract thresholds.
5. **Duplication** — within-site duplicate detection; cross-site duplicate *risk* assessed via privacy-preserving methods, never by exchanging identifiers.
6. **Distributional stability** — drift versus the site's prior snapshot, on approved summary statistics only.
7. **Label quality** — provenance of labels, adjudication procedure, inter-rater agreement where applicable.
8. **Leakage check** — no field that encodes the outcome or post-outcome information (a recurring, expensive failure mode).

Gate results are signed and published to the catalog as a **conformance report**. Failing sites are excluded automatically, with a reason, not quietly down-weighted.

### 4.6 Data management lifecycle

Define and automate, per data domain: intake and eligibility review → classification → harmonization → snapshot and registration → active use → drift monitoring → retention expiry → verified deletion (including derived artifacts and models where the agreement requires it) → deletion attestation.

Two commonly skipped pieces: **deletion must propagate** to snapshots, caches, checkpoints, and audit-exempt copies, with an attestation per participant; and **withdrawal of consent** needs a defined procedure — document up front whether it implies retraining, since retroactively removing one subject's influence from a trained model is expensive and sometimes infeasible.

---

## 5. Safety, security, and privacy controls

### 5.1 Threat model (FL-specific, state it explicitly)

| Threat | Vector | Primary controls |
|---|---|---|
| Data poisoning | A participant submits corrupted or mislabeled data | QC gates; robust aggregation; per-site influence monitoring; held-out evaluation |
| Model poisoning / backdoor | Crafted updates to implant behavior | Update-norm clipping; robust aggregators (trimmed mean, median, Krum-family); anomaly scoring per site per round; backdoor probes in the eval suite |
| Sybil / collusion | Multiple identities from one actor | Strong enrollment vetting; identity attestation; cohort-size floors; participation caps |
| Gradient inversion / memorization | Reconstructing records from updates or model | Secure aggregation so no individual update is visible; differential privacy with an enforced budget; large minimum cohort sizes; no per-site metric egress below thresholds |
| Membership inference | Querying the released model | DP; output regularization; rate limiting and query auditing at serving |
| Metric-channel leakage | Loss curves, per-site metrics, error analyses | Egress filter; aggregate-only reporting with minimum cell sizes and suppression |
| Supply-chain compromise | Malicious dependency or image | Pinned digests; SBOM per image; signed builds; in-toto attestations; hermetic builds; verify signatures at the participant before execution |
| Coordinator compromise | Aggregator exfiltrates or manipulates | Secure aggregation; participant-side verification of global model signatures; hardware-backed attestation where available; append-only transparency log of round configs |
| Insider misuse | Legitimate access, illegitimate purpose | Least privilege; two-person control on releases and policy changes; immutable audit logs reviewed on a schedule |
| Dual-use misuse of outputs | Legitimate model, harmful application | Intended-use gating; access tiering; release review by the governance body; usage monitoring and revocation |

### 5.2 Controls that must be in the platform, not in a runbook

- **Identity and attestation.** Workload identity for every node; short-lived credentials; mutual TLS between all planes; remote attestation of the participant runtime before it receives the global model.
- **Secure aggregation.** The coordinator receives only sums over a cohort above the minimum size. Individual updates are never in the clear on the coordinator.
- **Differential privacy where the data subject population warrants it.** Budget accounting in a ledger the platform enforces; report `(ε, δ)`, the accounting method, and the unit of privacy (record, patient, site) with every release. An unstated privacy unit makes ε meaningless.
- **Bounds enforcement.** Per-round caps on update norm and on any single cohort's contribution, so O3.1 holds by construction.
- **Egress filtering.** Allowlist of artifact types and shapes per participant, with suppression rules for small cells. Default deny.
- **Isolation and revocation.** One action removes a participant from future rounds and marks affected artifacts for review. Rehearse it quarterly; an unrehearsed kill switch is a hypothesis.
- **Secrets and key management.** HSM/KMS-backed; documented rotation; no long-lived credentials in CI or images.
- **Tamper-evident audit log.** Append-only, hash-chained, covering enrollment, approvals, round configs, participation, gate outcomes, releases, and access to outputs.

### 5.3 Biosecurity- and dual-use-specific guardrails

- **Intake review for data domains.** New domains get a written dual-use assessment before onboarding, not after.
- **Tiered access to outputs.** Models and derived analyses are classified; higher tiers need named approvers, stronger identity, and usage logging.
- **Institutional review integration.** Wire the platform to existing oversight (IRB/ethics, institutional biosafety, export-control review) so approval status is a machine-readable precondition for a run rather than an email.
- **Release review as a gate.** A standing multidisciplinary body (scientific, security, legal/regulatory, ethics) approves first release of each model and any widening of access. Their decision is recorded in the registry and enforced by the release pipeline.
- **Publication and disclosure policy.** Decide in advance how methods and weights are shared, and under what restrictions.
- **Incident response with a biosecurity annex.** Standard IR plus: who is notified when an output is misused or a dual-use concern is raised, and what the containment options are (revoke access, withdraw model, notify participants and authorities).

---

## 6. Reliability engineering

### 6.1 Make partial participation normal

Sites will be down, slow, mid-migration, or failing QC. Design so this is routine:

- **Quorum-based rounds.** Proceed at a defined participation floor; record who participated.
- **Asynchronous or buffered aggregation** where round-time variance is high; bound staleness explicitly.
- **Deadline + straggler policy** stated in config, not decided ad hoc.
- **Checkpoint every round**, content-addressed, so recovery costs one round.
- **Idempotent, retryable stages** with exponential backoff; at-least-once delivery with deduplication by artifact hash.
- **Cohort-weighted aggregation** with caps, so a large site cannot dominate and a small site cannot be erased.

### 6.2 SLOs and error budgets

Set service levels per stage and enforce them with an error budget that gates feature work:

| Stage | Example SLI | Example SLO |
|---|---|---|
| Snapshot freshness | Age of latest conforming snapshot per site | ≥95% of sites <7 days |
| QC gate | Gate run completes and reports | ≥99% within 2h of snapshot |
| Round completion | Rounds reaching quorum | ≥98% per week |
| Round latency | p95 wall-clock per round | Within program target |
| Recovery | Time to resume after coordinator failure | <30 min, verified by drill |
| Release integrity | Releases with complete, verifiable manifest | 100% (no budget) |
| Serving | Availability and p95 latency of decision support | Per consumption-plane target |

Integrity and privacy SLOs get **no error budget**. They are not tunable against velocity.

### 6.3 Testing, continuous evaluation, and drift

- **Unit and contract tests** for harmonization: a shared synthetic conformance suite every site must pass before enrollment and after any mapping change.
- **Synthetic-data end-to-end rehearsals** of the whole loop, including failure injection (a site that drops, a site that sends garbage, a site that sends adversarial updates).
- **Chaos and adversarial drills.** Scheduled exercises: drop a site mid-round, corrupt a delta, replay a stale update, simulate coordinator loss. Failures found in a drill cost far less than failures found in a release.
- **Continuous evaluation** on held-out sites for every candidate, including subgroup and batch-confound analyses.
- **Drift monitoring** on inputs (approved statistics only), outputs, and performance where labels arrive later. Define the action on drift — retrain, roll back, or restrict use — before drift occurs.
- **Shadow deployment** before any model influences a decision.
- **Rollback as a first-class path**, tested, with a named owner.

### 6.4 Change management

Everything is code and review: contracts, mapping specs, aggregation configs, privacy parameters, gate thresholds, release criteria. Two-person review for anything touching privacy parameters, egress rules, or release gates. Participants get advance notice and a compatibility window for contract changes; the platform refuses to mix contract versions in one round.

---

## 7. Governance and roles

| Role | Owns |
|---|---|
| Program steering | Core goals, participation terms, escalation |
| Data stewards (per site) | Local data, mapping spec, legal basis, QC outcomes |
| Contract owner | Feature contracts and their versioning |
| Platform/SRE | Orchestration, SLOs, reliability, incident response |
| Security | Threat model, controls, attestation, IR |
| Privacy/legal | Legal bases, agreements, DP parameters, retention |
| Independent evaluation | Held-out benchmarks, subgroup analysis, red teaming |
| Release review body | Intended use, dual-use assessment, release and access decisions |
| Model owner | A model's lifecycle, documentation, retirement |

Every model carries a **model card** (intended use, out-of-scope uses, training population, contract version, participants and rounds, metrics with subgroup breakdowns, privacy parameters and unit, known failure modes, monitoring plan, retirement criteria) and every dataset snapshot carries a **datasheet** (the metadata of §4.4 plus limitations in the steward's own words). No release without both.

---

## 8. Implementation sequence

Do not build the whole stack before the first result. Order that de-risks fastest:

1. **Weeks 0–6 — Agree the contract and governance skeleton.** One narrow use case, one feature contract, the intended-use statement, the release-review body, and the participation agreement. Write the threat model down.
2. **Weeks 4–10 — Edge harmonization and QC, on synthetic data.** Ship the containerized harmonizer and gate suite; every candidate site passes the conformance suite before touching real records.
3. **Weeks 8–16 — Orchestration with security in from the start.** Identity, attestation, signed artifacts, secure aggregation, egress filter, audit log, checkpointing. Retrofitting secure aggregation later means re-approving every agreement.
4. **Weeks 12–20 — Two- or three-site pilot on real data.** Target a boring model and a real held-out evaluation. The goal is a trustworthy end-to-end result, not a good metric.
5. **Weeks 18–28 — Reliability and adversarial hardening.** SLOs, drills, robust aggregation tuning, DP budget ledger, rollback rehearsal.
6. **Weeks 24+ — Scale participants, then scale model ambition.** In that order. Each new site is a conformance run and an agreement, not a config line.

**Signs the program is off track:** raw data being requested "just to debug"; per-site metrics circulating informally; mapping changes without review; a benchmark the training team can see; a kill switch never exercised; a model card written after the model shipped; privacy parameters chosen to hit an accuracy target.

---

## 9. Standards and frameworks to bind to

- **Security/risk:** NIST SP 800-53, NIST Cybersecurity Framework, NIST AI Risk Management Framework, NIST SP 800-207 (zero trust), ISO/IEC 27001, ISO/IEC 42001 (AI management systems), SLSA + in-toto for supply chain.
- **Privacy:** ISO/IEC 27701, applicable data-protection law per jurisdiction, de-identification standards per data type (e.g. DICOM confidentiality profiles).
- **Data:** FAIR principles, HL7 FHIR, OMOP CDM, CDISC, LOINC/SNOMED CT/UCUM, GA4GH specifications, W3C PROV, DCAT.
- **Software/ML quality:** IEEE/ISO software lifecycle practice, model risk management discipline, and — where outputs approach clinical or regulatory use — the relevant medical-device software and good-practice regimes (e.g. IEC 62304, ISO 14971, GxP data-integrity expectations such as ALCOA+).
- **Biosafety/biosecurity:** institutional biosafety and dual-use research oversight processes applicable to the participants, plus national biosecurity and export-control regimes.

Pick the specific instruments that apply to your jurisdictions and participants, and record the mapping from each control in §5 to the framework clause it satisfies — that mapping is what makes audits and onboarding cheap.

---

## 10. One-page summary

- Federated learning solves data movement; it does not solve semantics, quality, security, or dual-use. Budget most of the effort for those.
- **Feature contracts** and **edge harmonization** are the backbone of data standards. Automated **quality gates** with signed conformance reports are what make remote data trustworthy without being visible.
- Treat the **coordinator as untrusted**, make **egress an enforced chokepoint**, and enforce **bounds** so no single participant can move the model far.
- **Reliability means partial participation is normal**: quorum rounds, checkpoints, idempotent stages, SLOs with error budgets, and rehearsed failure drills.
- **Governance in the code path**: intended-use gating, release review, model cards and datasheets, tamper-evident audit logs, and a rehearsed isolation procedure.
- Integrity and privacy objectives get **no error budget**. Everything else is negotiable against delivery speed.
