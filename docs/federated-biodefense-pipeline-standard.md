# Infrastructure, Data Standards, and Data Management for Federated Learning in Biological Countermeasures

**Status:** draft for review
**Evidence current as of:** 25 September 2026
**Scope:** platform, data, and governance standards for a multi-party federated learning (FL) program supporting medical and biological countermeasure R&D — vaccines, therapeutics, diagnostics, and surveillance analytics.

This document sets out core goals, the data and infrastructure standards that operationalize them, and the controls that make the pipelines safe, secure, and reliable. Claims are cited. Where the literature is contested or a policy is in flux, that is stated rather than smoothed over.

---

## 0. How to read this document, and what it is not

**On evidence.** Every substantive claim below carries a citation. Where a source could not be read in full, or where sources conflicted, the text says so inline. Three recurring markers:

- **[verified]** — primary source or DOI resolution confirmed directly.
- **[secondary]** — confirmed via the official page's indexed content and at least two independent corroborating sources, but the primary document was not read in full.
- **[unresolved]** — sources conflict, or the claim could not be corroborated. Treat as a research task, not a fact.

**On regulatory currency.** Section 8 documents a policy landscape that moved substantially in 2025–2026. Several frameworks widely assumed to be in force are not. Any compliance artifact built from this document must pin exact versions and dates, and must be re-verified before it is relied upon — the half-life of Section 8 is short.

**Non-goals.** This platform produces evidence for humans who make decisions. It does not design biological agents or interventions, does not control wet-lab automation, and does not make clinical or regulatory determinations autonomously. Section 8.3 explains why the boundary between computational design work and regulated life-sciences research is currently the most consequential open question for a program of this kind — and why it must be resolved at intake rather than assumed.

---

## 1. What federated learning actually buys, and what it does not

### 1.1 The case, with numbers

Biomedical data is rarely poolable: it sits in hospital systems, national laboratories, public-health agencies, CROs, and commercial sponsors under incompatible legal bases. Federated learning moves the model to the data instead. The strongest evidence that this produces real scientific value:

- **EXAM** — 20 hospitals across five continents, predicting COVID-19 oxygen requirements from vitals, labs, and chest radiographs. Average AUC >0.92 at 24 and 72 hours, a 16% improvement in average AUC across sites and a **38% average increase in generalizability** over models trained at a single site ([Dayan et al., *Nature Medicine* 2021;27:1735–1743](https://www.nature.com/articles/s41591-021-01506-3)). [secondary]
- **FeTS** — the largest FL study to date: **71 sites across 6 continents**, n=6,314 glioblastoma cases, yielding **33% improvement** in delineating the surgically targetable tumor and **23%** for complete tumor extent over a publicly-trained model ([Pati et al., *Nature Communications* 2022;13:7346](https://www.nature.com/articles/s41467-022-33407-5); note the [Author Correction](https://www.nature.com/articles/s41467-023-36188-7)). [secondary]
- **MELLODDY** — ten competing pharmaceutical companies, 2.6bn confidential activity data points across 21m compounds, each realizing model improvements without exposing proprietary data ([Heyndrickx et al., *J. Chem. Inf. Model.* 2024](https://pubs.acs.org/doi/10.1021/acs.jcim.3c00799)). Evidence that federation works across *competitive*, not just institutional, trust boundaries. [secondary]

The foundational framing is [Rieke et al., *npj Digital Medicine* 2020;3:119](https://www.nature.com/articles/s41746-020-00323-1) and [Kairouz, McMahan et al., *Advances and Open Problems in Federated Learning*](https://arxiv.org/pdf/1912.04977).

### 1.2 The counterweight — read this before committing

A 2024 review of healthcare FL through May 2024 concluded that **"the vast majority are not appropriate for clinical use,"** citing privacy, generalization, and communication deficiencies ([Li et al., arXiv:2409.09727](https://arxiv.org/abs/2409.09727)). [verified — abstract read directly]

This should temper the program's ambitions. The published successes above are the exceptions that required enormous institutional effort; they are not the median outcome. A widely-repeated figure holds that only ~5.2% of published healthcare FL studies represent real clinical deployment — **[unresolved]**, source not reachable, do not cite without verification.

### 1.3 What inverts relative to centralized ML

| Centralized assumption | Federated reality | Consequence for this document |
|---|---|---|
| One schema, one curation team | N schemas, N curation cultures | §4 (feature contracts) |
| Debug by looking at the data | You can never look; you debug through statistics and contracts | §5 (quality gates) |
| One trust boundary | Every participant is a dependency *and* a potential adversary | §6 (threat model) |
| Availability is one platform's problem | Availability is the product of all participants' | §7 (partial participation) |
| Model output is the only artifact | Gradients, metrics, and loss curves are potential disclosure channels | §6.3 (egress) |

FL reduces raw-data movement risk. It does **not** reduce — and in some respects concentrates — model-mediated disclosure risk, data-quality risk, and dual-use risk.

### 1.4 Federation is not the only option

Before committing to FL, compare against the **Five Safes** model — safe projects, safe people, safe settings, safe data, safe outputs — which underpins Trusted Research Environments and is the mature alternative for controlled analysis of sensitive data ([Ritchie, UK ONS](https://www.dundee.ac.uk/tre/safe-settings/five-safes-framework)). [secondary]

This matters beyond the build/buy decision: **"safe outputs" is statistical disclosure control, a discipline with decades of practice.** Any egress control this platform builds (§6.3) should bind to that literature rather than reinvent it. The EU's European Health Data Space is converging on the same architecture — computation inside a certified secure processing environment, with only anonymised or pseudonymised data leaving it (§8.2).

---

## 2. Core goals

Five goals with measurable objectives. Everything later traces to one.

### G1 — Scientific validity

Performance must transfer to sites that did not contribute to training. The evidence that this is the binding constraint, not a theoretical worry: [Zech et al., *PLOS Medicine* 2018;15(11):e1002683](https://journals.plos.org/plosmedicine/article?id=10.1371%2Fjournal.pmed.1002683) trained pneumonia classifiers on 158,323 radiographs from three institutions and found performance on outside-hospital images significantly lower in **3 of 5 comparisons** — because the CNNs identified *hospital system and department* with high accuracy and calibrated predictions to local disease prevalence. [secondary]

- **O1.1** Every released model reports performance on ≥2 held-out sites not used in training, with **disaggregated** breakdowns by demographic, phenotypic, and intersectional groups and by assay platform and specimen type. Disaggregated evaluation is the defining contribution of Model Cards and is not optional ([Mitchell et al., FAT\* 2019](https://dl.acm.org/doi/10.1145/3287560.3287596)). [verified]
- **O1.2** Every release is reproducible from recorded inputs: pinned code, container digests, round schedule, aggregation config, seeds, per-site snapshot hashes.
- **O1.3** Pre-registered evaluation plan before training. Post-hoc metric selection is a release blocker.
- **O1.4** An explicit leakage audit. Leakage is a *widespread* failure mode — at least **294 papers across 17 disciplines** ([Kapoor & Narayanan, *Patterns* 2023;4(9):100804](https://www.cell.com/patterns/fulltext/S2666-3899(23)00159-9)). Adopt their **model info sheet** alongside the datasheet and model card. [verified]
- **O1.5** Train/test independence enforced at the **site and patient** level, never the sample level — an explicit regulatory expectation under GMLP principle 4 (§8.1).

### G2 — Data protection and sovereignty

- **O2.1** Raw records never leave a participant's boundary. Only model deltas, encrypted aggregates, and approved summary statistics cross it.
- **O2.2** Every cross-boundary flow carries a named legal basis and data-use agreement reference in metadata.
- **O2.3** If DP is used, the **unit of privacy** (record / patient / site) is declared with every ε, and the budget is enforced by the platform rather than by convention. See §6.2 — an unstated unit makes ε meaningless.
- **O2.4** Do not assume federation removes you from data-protection scope. **Gradients, embeddings, and weights derived from personal data are frequently still personal data** under GDPR (§8.2). [secondary]

### G3 — Security and integrity

- **O3.1** No single participant can move the global model beyond configured bounds in one round.
- **O3.2** Every artifact — container, snapshot, delta, aggregate, release — is signed and verifiable to an identity (§6.4).
- **O3.3** Time to isolate a misbehaving participant measured in minutes, via a **rehearsed** procedure. An unrehearsed kill switch is a hypothesis.

### G4 — Reliability and operability

- **O4.1** Training completes at a defined quorum, not full participation.
- **O4.2** Every round resumable from checkpoint; no failure costs more than one round.
- **O4.3** SLOs per stage with error budgets that gate feature work — except integrity and privacy SLOs, which get no budget.

### G5 — Responsible use and accountability

- **O5.1** Approved intended-use statement and explicit out-of-scope list before first release.
- **O5.2** Standing review body signs off on new data domains, new model classes, and any widening of access.
- **O5.3** Tamper-evident audit trail: who trained what, on which data, under which approval, consumed by whom.
- **O5.4** Export-control and dual-use jurisdiction determination recorded at intake and re-run at every change of collaborator or data source (§8.3–8.4).

---

## 3. Reference architecture

Five planes, each with one job and a stated trust boundary.

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
│  bounds enforcement · robust aggregation · checkpointing              │
└───────────────────────────────────────────────────────────────────────┘
        ▲ signed deltas / encrypted shares          ▼ signed global model
┌───────────────────────────────────────────────────────────────────────┐
│ PARTICIPANT PLANE  (per site, inside the site's boundary)             │
│  ingest → harmonization → QC gates → snapshot → local training        │
│  local eval · egress / disclosure control · local audit log           │
└───────────────────────────────────────────────────────────────────────┘
┌───────────────────────────────────────────────────────────────────────┐
│ EVALUATION PLANE  (independent of training)                           │
│  held-out site benchmarks · drift & subgroup monitors · red team      │
└───────────────────────────────────────────────────────────────────────┘
┌───────────────────────────────────────────────────────────────────────┐
│ CONSUMPTION PLANE                                                     │
│  serving · decision support · reporting · feedback capture            │
└───────────────────────────────────────────────────────────────────────┘
```

**Architectural rules:**

1. **The coordinator is untrusted with data.** It sees only what secure aggregation and disclosure control permit. Design as if it will be breached.
2. **Egress is an enforced chokepoint** implementing statistical disclosure control (§1.4, §6.3). Default deny.
3. **Evaluation does not report to training.** Different owners, different credentials, separate held-out data. A team that can tune against the benchmark does not have a benchmark.
4. **Reproducibility is infrastructure**: content-addressed artifacts, pinned digests, a run manifest per round.
5. **Zero trust between planes.** Treat each training job, feature store, model registry, and inference endpoint as a separately authenticated subject, per [NIST SP 800-207](https://csrc.nist.gov/pubs/sp/800/207/final) and [SP 800-207A](https://csrc.nist.gov/pubs/sp/800/207/a/final). [secondary]
6. **Governance is enforced in code paths.** An approval that exists only in a wiki is not a control.

---

## 4. Data standards

The hardest part of federated work is not the learning. It is agreeing what a row means at every site.

### 4.1 Bind to existing standards

Record the exact version bound to. Versions below were verified where the domain was reachable; standards-body sites were largely blocked from this environment (§0).

| Domain | Standard | Notes on version and scope |
|---|---|---|
| Clinical / EHR exchange | **HL7 FHIR** | **R5 (v5.0.0, March 2023)** is current published; R6 is in normative ballot, **not** published. But **R4 (v4.0.1) remains the deployment reality** for US Core and EHDS work — choose deliberately. [secondary] |
| Observational analytics | **OMOP CDM (OHDSI)** | **v5.5** is current — a non-breaking superset of v5.4. **v6.0 exists but is not supported by the OHDSI tool stack**; never cite it as latest. [verified via repo] |
| Clinical trial data | **CDISC SDTM** (as-collected tabulation), **ADaM** (analysis-ready, traceable) | Version numbers **[unresolved]** — cdisc.org unreachable; verify at source before publishing. |
| Nonclinical toxicology | **CDISC SEND** | SEND is **SDTM applied to the nonclinical domain** — animal toxicology, not clinical. Include only if the program covers preclinical safety. [secondary] |
| Lab tests / observations | **LOINC** (Regenstrief) | Names *what was measured*, not the answer. Free license. |
| Clinical concepts | **SNOMED CT** (SNOMED International) | Findings, disorders, procedures. **Requires a license** — free in member countries via national release centres, paid otherwise. Do not imply it is freely usable. [secondary] |
| Units | **UCUM** (Regenstrief) | "Unified **Code** for Units of Measure" — singular. |
| Phenotype exchange | **GA4GH Phenopackets v2** | Published as **ISO 4454:2022** — the first GA4GH standard published by ISO. [secondary] |
| Data resolution | **GA4GH DRS v1.5.0** | An **ID→access-URL resolution layer**, not a transfer protocol and not a search API. [verified via repo] |
| Researcher authz | **GA4GH AAI v1.2** *and* **GA4GH Passport v1.2** | **Two distinct specifications** from two work streams (Data Security; DURI). AAI is the OIDC protocol, Passport the token format. Their own spec notes they may merge later. [verified via spec sources] |
| Imaging | **DICOM PS3.15 Annex E**, Attribute Confidentiality Profiles | E.2 Basic Profile plus named Options; Table E.1-1 is the per-attribute action table. **The standard itself states the profiles do not guarantee de-identification and do not replace a de-identification process.** Burned-in pixel data, private tags, and free-text descriptors are the residual-risk vectors. [secondary] |
| Experiment metadata | **ISA** (Investigation–Study–Assay) | Domain-neutral community framework (ISA-Tab / ISA-JSON), ELIXIR Recommended Interoperability Resource. Not a ratified SDO standard — describe it accurately. [secondary] |
| Standards discovery | **FAIRsharing** | The live successor to MIBBI (2008) → BioSharing (2011) → FAIRsharing (2017). MIBBI no longer exists as an active registry. [secondary] |
| Organism identity | **NCBI Taxonomy** | The nomenclature repository for INSDC. **Not** an authoritative phylogenetic or nomenclatural authority — NCBI says so itself. |
| Provenance | **W3C PROV** (Recommendation, 30 April 2013) | Cite **PROV-O** specifically if you mean the RDF ontology. Stable for a decade. |
| Catalog metadata | **W3C DCAT 3** (Recommendation, 22 August 2024) | Built on PROV-O terms. The health profile **HealthDCAT-AP** is separate and evolving — do not conflate. |
| Findability principles | **FAIR** ([Wilkinson et al., *Sci Data* 2016;3:160018](https://doi.org/10.1038/sdata.2016.18)) | [verified — DOI resolves] |

**Two corrections worth stating explicitly, because both are common errors:**

- **MIAPPE does not belong in a human biomedical standards list.** It is the *Minimum Information About a Plant Phenotyping Experiment* — its entities are germplasm, growth facility, field plot layout, and agronomic treatment. An earlier draft of this document cited it; that was wrong. [secondary]
- **MINSEQE is orphaned.** The FGED Society dissolved in September 2021. Cite it as a historical checklist with its Zenodo record, never as a live standard with a governing body. [secondary]

**"FAIR compliance" is not a meaningful claim.** FAIR has no conformance test, no version, and no certifying body. Say the platform "implements the FAIR principles via DCAT, PROV, and persistent identifiers." Note also that **Accessible ≠ open** — FAIR explicitly accommodates authenticated, controlled access (A1.2), which is exactly why GA4GH Passports/AAI is its right companion for human data.

**Rule:** no free-text where a code system exists; every coded field carries `(system, version, code)`. Unmapped values go to an explicit queue with the original string retained locally — never silently coerced.

### 4.2 The Feature Contract

One versioned, machine-readable contract per model family, authored centrally, enforced locally. Each field specifies:

- logical name, semantic type, unit (UCUM)
- source standard and code system with version
- permissible values / range, and the action on violation (reject row, null with flag, clamp)
- missingness policy — is missing informative, and how encoded
- normalization/derivation as executable code, not prose
- required or optional, and behavior when absent
- sensitivity class, and whether it may influence egressed statistics

Contracts are semantically versioned. A breaking change forces a new model lineage rather than silent reinterpretation of history. Every round records its contract version; a site reporting a different version is excluded from that round.

### 4.3 Harmonization at the edge

Each participant runs the same containerized harmonization job: source → contract-conformant local dataset. Mapping logic stays with the people who understand the source system; the platform sees one schema; mapping bugs surface in conformance reports rather than requiring access to raw data.

Each site maintains a reviewed, version-controlled **mapping specification**. Treat an unreviewed mapping change as a production change, because it is.

### 4.4 Metadata required with every snapshot

**Dataset-level:** persistent identifier, snapshot hash, creation time, contract version; provenance (source systems, extraction window, harmonization image digest); cohort definition as executable query; population descriptors sufficient for representativeness assessment at a granularity that does not itself identify anyone; collection context (assay platforms, instruments and versions, specimen handling, batch structure); legal basis, consent scope, DUA reference, retention and deletion date; sensitivity classification and export-control determination; known limitations written by the site.

**Record-level** (only as the contract permits): stable pseudonymous key, timestamps at defined resolution, batch identifiers, quality flags.

**Batch structure is first-class metadata.** Zech et al. (§2/G1) is the empirical basis: site, instrument, reagent lot, and collection period are the leading sources of spurious signal. If you cannot name the batch variables, you cannot claim the model learned biology rather than logistics.

Pair the prose **datasheet** ([Gebru et al., *CACM* 2021;64(12):86–92](https://dl.acm.org/doi/10.1145/3458723) — seven sections: Motivation, Composition, Collection, Preprocessing, Uses, Distribution, Maintenance) with machine-readable DCAT/PROV records. Datasheets are qualitative by design and have no conformance criteria; they complement rather than replace structured metadata. [verified]

---

## 5. Data quality: use the canonical framework

Do not invent a quality taxonomy. The harmonized framework for secondary use of EHR data is [Kahn et al., *eGEMs* 2016;4(1):1244](https://doi.org/10.13063/2327-9214.1244) [verified — DOI resolves], and it has **two axes**, the second of which is commonly dropped:

**Categories:** **Conformance** (value, relational, computational) · **Completeness** · **Plausibility** (uniqueness, atemporal, temporal).
Note the asymmetry: Conformance and Plausibility have subcategories; **Completeness does not.**

**Contexts:** **Verification** — against internal expectations and metadata · **Validation** — against an external gold standard or real-world referent.

**Implementation.** The [OHDSI Data Quality Dashboard](https://github.com/OHDSI/DataQualityDashboard) ([Blacketer et al., *JAMIA* 2021;28(10):2251–2257](https://doi.org/10.1093/jamia/ocab132)) [verified] operationalizes exactly this taxonomy against OMOP: **24 parameterized check types instantiated as >3,000 individual checks**, each tagged verification or validation. (Those two numbers describe templates and instantiations respectively — do not conflate them.) Pair with **ACHILLES** for database characterization.

**Federated quality assessment specifically.** There is **no agreed standard.** Kahn gives terminology, DQD gives an OMOP implementation, and the operational patterns come from FDA **Sentinel** (coordinating center distributes check programs, sites execute locally, results return) and **PCORnet** (which adapted Sentinel's approach for "foundational data quality" — fitness across a research portfolio rather than per-study). The standard critique is that these are **top-down**: sites must satisfy centrally-defined targets rather than assess in local context. Cross-network comparability remains open. Do not imply a settled standard exists. [secondary]

**Gates to run before a site may join a round**, expressed in Kahn's terms plus two additions this setting requires:

1. Conformance — types, units, code systems, required fields, relational integrity.
2. Completeness — missingness per field within contract thresholds.
3. Plausibility — physiological/physical implausibility; temporal consistency; within-site duplication.
4. Cross-site duplication **risk** — assessed via privacy-preserving methods, never by exchanging identifiers.
5. Distributional stability — drift vs. the site's prior snapshot, on approved statistics only.
6. Label quality — provenance, adjudication procedure, inter-rater agreement where applicable.
7. **Leakage check** — no field encoding the outcome or post-outcome information (§G1/O1.4).

Results are signed and published as a **conformance report**. Failing sites are excluded automatically with a stated reason, not quietly down-weighted.

### 5.1 Lifecycle

Intake and eligibility review → classification → harmonization → snapshot and registration → active use → drift monitoring → retention expiry → verified deletion → deletion attestation.

Two commonly skipped pieces: **deletion must propagate** to snapshots, caches, checkpoints, and derived artifacts, with a per-participant attestation; and **consent withdrawal** needs a defined procedure — decide up front whether it implies retraining, because retroactively removing one subject's influence from a trained model is expensive and sometimes infeasible.

---

## 6. Safety, security, and privacy

Write the threat model in the vocabulary of **[NIST AI 100-2e2025](https://csrc.nist.gov/pubs/ai/100/2/e2025/final)**, *Adversarial Machine Learning: A Taxonomy and Terminology of Attacks and Mitigations* (final, March 2025; supersedes the 2023 edition). It organizes attacks by lifecycle stage and by attacker goal, capability, and knowledge — evasion, poisoning, privacy, abuse — and its attack/mitigation index is the practical artifact. Map each pipeline stage to its attack classes, then map mitigations back to SP 800-53 controls. [secondary]

### 6.1 Threat model

| Threat | Vector | Controls | Evidence / caveat |
|---|---|---|---|
| Data poisoning | Corrupted or mislabeled data at a site | QC gates (§5); per-site influence monitoring; held-out evaluation | — |
| Model poisoning / backdoor | Crafted updates | Update-norm clipping; bounds enforcement; anomaly scoring; backdoor probes in eval | See §6.5 — the literature here is genuinely contested |
| Sybil / collusion | Multiple identities from one actor | Enrollment vetting; identity attestation; cohort floors; participation caps | Low participant counts in cross-silo make this easier to control than cross-device |
| Gradient inversion | Reconstructing records from updates | Secure aggregation; DP; **not** cohort size alone | [Geiping et al., NeurIPS 2020](https://arxiv.org/abs/2003.14053): "averaging gradients over several iterations or several images does not protect privacy"; input to a fully connected layer is analytically reconstructible |
| Membership inference | Querying the released model | DP; output regularization; rate limiting and query auditing at serving | — |
| Metric-channel leakage | Loss curves, per-site metrics, error analyses | Disclosure control on all egress; minimum cell sizes; suppression | §1.4 — bind to SDC practice |
| Supply-chain compromise | Malicious dependency or image | Pinned digests; SBOM; signed builds; in-toto attestations; verify at participant before execution | §6.4 |
| Coordinator compromise | Aggregator exfiltrates or manipulates | Secure aggregation; participant-side verification of global model signatures; append-only transparency log of round configs | TEEs help but are not a clean answer — §6.4 |
| Insider misuse | Legitimate access, illegitimate purpose | Least privilege; two-person control on releases and policy changes; scheduled audit-log review | — |
| Dual-use misuse of outputs | Legitimate model, harmful application | Intended-use gating; tiered access; release review; usage monitoring and revocation | §8.3 |

### 6.2 Privacy mechanisms — state precisely what each guarantees

**Secure aggregation** ([Bonawitz et al., CCS 2017](https://dl.acm.org/doi/10.1145/3133956.3133982)) gives a communication-efficient, dropout-robust protocol by which the server computes a **sum** over a cohort without learning any individual contribution. [secondary]

Its limit must be stated: it protects the *individual update*, not the *aggregate*. Anything inferable from the sum remains inferable. Secure aggregation and DP are complements, not substitutes.

**Differential privacy.** Declare the **unit of privacy** with every ε — record, patient, or site. Example-level and user/client-level DP give materially different guarantees, and in a multi-hospital setting the thing needing protection is usually the **patient**, who may contribute many records; a single-record definition degrades accordingly. An ε reported without its unit, its accounting method, and its composition assumptions is not a privacy claim. [secondary]

**End-to-end integration** is demonstrated by **PriMIA** ([Kaissis et al., *Nature Machine Intelligence* 2021;3:473–484](https://www.nature.com/articles/s42256-021-00337-8)) — differentially private, securely aggregated FL with encrypted inference on paediatric chest radiographs, achieving performance comparable to locally trained models and empirically resisting a gradient-based inversion attack. See also [Kaissis et al., *Nat Mach Intell* 2020;2:305–311](https://www.nature.com/articles/s42256-020-0186-1) for the survey framing. [secondary]

### 6.3 Egress as disclosure control

Each participant runs a default-deny filter over an allowlist of artifact types and shapes, with minimum cell sizes and suppression rules. This is the "safe outputs" leg of the Five Safes (§1.4). Treat it as statistical disclosure control with an established methodology and tooling, not as a bespoke filter.

### 6.4 Platform controls, not runbook controls

- **Identity and attestation.** Workload identity per node; short-lived credentials; mutual TLS across planes; remote attestation of the participant runtime before it receives the global model.
- **Bounds enforcement.** Per-round caps on update norm and on any cohort's contribution, so O3.1 holds by construction rather than by detection.
- **Supply chain.** **SLSA v1.2** (approved 12 November 2025) — Build Track L1 provenance exists, **L2** signed by a hosted build platform (tamper-evidence), **L3** strong run isolation plus non-falsifiable provenance; there is no L4 in the Build Track. v1.2 adds a **Source Track** reaching L4 with mandatory two-person review. **[secondary — slsa.dev was unreachable; corroborated across three sources; verify level semantics at source.]** **in-toto Attestation Framework v1.2.0** [verified via releases] provides the DSSE envelope binding a subject (artifact + digest) to a predicate (a claim about how it was built). in-toto is the *format*; SLSA levels assert *build-platform integrity*. The high-value move here is extending in-toto predicates beyond build provenance to **dataset provenance, model-weight provenance, and evaluation-result attestations**, so a model carries a verifiable chain back to its training corpus and its review.
- **Confidential computing, with eyes open.** TEEs reduce the trusted computing base but **side-channel attacks are the primary and well-documented threat**, and enclave memory limits constrain ML workloads ([SoK: Limitations of Confidential Computing via TEEs](https://www.cs.ucdavis.edu/~peisert/research/2022-SEED-TEE-SOK.pdf)). Use TEEs as defense in depth; do not let them carry the privacy argument alone. [secondary]
- **Secrets.** HSM/KMS-backed, documented rotation, no long-lived credentials in CI or images.
- **Tamper-evident audit log.** Append-only, hash-chained, covering enrollment, approvals, round configs, participation, gate outcomes, releases, and access to outputs. This also discharges the ALCOA+ *Attributable/Contemporaneous* expectations (§8.1) and EU AI Act Article 12 logging (§8.2).
- **Baseline.** [NIST SP 800-53 Rev. 5 **Release 5.2.0**](https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final) (27 August 2025) — **pin the release, not just the revision.** 5.2.0 adds controls on software resiliency by design, developer testing, secure update and patch deployment, and software integrity. Consume the OSCAL feed. [secondary]

### 6.5 Where the literature is contested — robust aggregation

An earlier draft of this document listed Krum and trimmed-mean aggregation as settled controls. That was too confident in both directions, and the honest picture is regime-dependent.

- **Attacks are real under their stated assumptions.** [Bagdasaryan et al., AISTATS 2020](https://arxiv.org/abs/1807.00459) showed model-replacement backdoors reaching 100% backdoor accuracy in a single round, with an attacker controlling <1% of participants able to prevent unlearning. [secondary]
- **Defenses have formal but bounded guarantees.** Krum (Blanchard et al. 2017) and coordinate-wise median / trimmed mean ([Yin et al. 2018](https://arxiv.org/pdf/1803.01498), order-optimal statistical error rates under subexponential gradient noise) are the canonical estimators. [secondary]
- **But the threat models were largely unrealistic.** [Shejwalkar, Houmansadr, Kairouz & Ramage, IEEE S&P 2022](https://arxiv.org/abs/2108.10241) systematized the space and concluded prior work assumed impractically high compromised-client fractions and adversary capability, finding that **"FL is highly robust in practice even when using simple, low-cost defenses."** [verified — abstract read directly]
- **And defense evaluation remains weak.** [*SoK: Benchmarking Poisoning Attacks and Defenses in FL* (arXiv:2502.03801, 2025)](https://arxiv.org/abs/2502.03801) notes defenses are typically evaluated in isolation against limited attack strategies, and benchmarks 15 attacks × 17 defenses across 2,040 settings. [secondary]

**What this means here.** Shejwalkar's reassurance is grounded in *cross-device* production FL with large participant populations. A biomedical consortium is **cross-silo**: few participants, each with large influence — where a single compromised site is a much larger fraction of the update than in the setting that study examined. The controls that transfer regardless of regime are the ones that do not depend on detecting a clever adversary: **contractual and identity vetting at enrollment, bounds enforcement, per-site influence monitoring, and independent held-out evaluation.** Treat robust aggregators as useful defense in depth with contested guarantees, not as the security argument.

---

## 7. Reliability engineering

### 7.1 Partial participation is the normal case

Sites will be down, slow, mid-migration, or failing QC. Design for it: **quorum-based rounds** at a defined participation floor, recording who participated; **asynchronous or buffered aggregation** with explicitly bounded staleness where round-time variance is high; a **deadline and straggler policy** in config rather than decided ad hoc; **checkpoint every round**, content-addressed; **idempotent, retryable stages** with backoff and deduplication by artifact hash; **cohort-weighted aggregation with caps**, so a large site cannot dominate and a small one cannot be erased.

Statistical heterogeneity is the dominant performance risk, not the availability problem: non-IID label and feature distributions across sites cause client drift and unstable convergence. Quantity imbalance compounds it. [secondary]

### 7.2 SLOs and error budgets

| Stage | Example SLI | Example SLO |
|---|---|---|
| Snapshot freshness | Age of latest conforming snapshot per site | ≥95% of sites <7 days |
| QC gate | Gate completes and reports | ≥99% within 2h of snapshot |
| Round completion | Rounds reaching quorum | ≥98% per week |
| Round latency | p95 wall-clock per round | Program target |
| Recovery | Time to resume after coordinator failure | <30 min, verified by drill |
| Release integrity | Releases with complete verifiable manifest | 100% — no budget |
| Privacy budget enforcement | Releases with declared unit and accounted ε | 100% — no budget |
| Serving | Availability, p95 latency | Consumption-plane target |

### 7.3 Testing, evaluation, drift

Contract tests for harmonization via a shared synthetic conformance suite every site passes before enrollment and after any mapping change. Synthetic end-to-end rehearsals including failure injection. **Chaos and adversarial drills** on a schedule: drop a site mid-round, corrupt a delta, replay a stale update, simulate coordinator loss. Continuous evaluation on held-out sites with subgroup and batch-confound analysis. Drift monitoring on inputs (approved statistics only), outputs, and delayed-label performance — with the action on drift (retrain, roll back, restrict use) defined *before* drift occurs. Shadow deployment before any model influences a decision. Rollback as a tested first-class path with a named owner.

### 7.4 Change management

Contracts, mapping specs, aggregation configs, privacy parameters, gate thresholds, and release criteria are all code under review. **Two-person review** for anything touching privacy parameters, egress rules, or release gates — which also aligns with SLSA Source Track L4. Participants get advance notice and a compatibility window for contract changes; the platform refuses to mix contract versions within a round.

---

## 8. Regulatory and governance landscape

> **This section has the shortest half-life in the document.** Several frameworks commonly assumed to be in force are not. Verify before relying on any of it.

### 8.1 Medical and quality regimes

- **FDA PCCP guidance is FINAL** — *Marketing Submission Recommendations for a Predetermined Change Control Plan for Artificial Intelligence-Enabled Device Software Functions*, Federal Register notice 4 December 2024, Docket FDA-2022-D-2628. **This is the single most consequential document for pipeline architecture.** A PCCP authorizes pre-specified modifications without a new marketing submission, and requires (1) a bounded Description of Modifications, (2) a Modification Protocol covering data management, re-training, performance evaluation and acceptance criteria, and (3) an Impact Assessment. **Your retraining loop, drift monitoring, dataset versioning, and acceptance thresholds must be designed to be writable into a Modification Protocol up front** — otherwise every model refresh becomes a new submission. [secondary]
- **FDA AI device lifecycle guidance is still DRAFT** (issued 7 January 2025, not finalized as of September 2026). Do not cite as binding. [secondary]
- **FDA AI-in-drug-development guidance** (draft, January 2025) introduces a 7-step risk-based **Credibility Assessment Framework** anchored on Context of Use. Design implication: emit a credibility dossier **per COU, not per model**. Whether a final version issued in 2026 is **[unresolved]** — sources conflicted.
- **FDA + EMA joint *Guiding Principles of Good AI Practice in Drug Development*, 14 January 2026** — ten joint principles, the first FDA/EMA alignment on AI for medicines. [secondary]
- **GMLP guiding principles** (FDA / Health Canada / MHRA, 10 principles, October 2021), plus PCCP principles (2023) and Transparency principles (June 2024). Principle 4 — **training and test datasets independent** — dictates splitting at site/patient level (§G1/O1.5). Principle 7 evaluates the **human-AI team**, not the model alone. [secondary]
- **IEC 62304 Ed. 1.1** (2015, corrected 2017) remains current; Edition 2 is **not** published. Its **SOUP** requirements are where your ML framework, CUDA stack, and any pretrained weights land — a foundation model in a Class C device is SOUP with an awkward anomaly history. **ISO 14971:2019** reconfirmed March 2025; its post-production feedback loop is where drift monitoring legally attaches. **ISO 13485:2016** reconfirmed 31 October 2025. **FDA QMSR** became effective 2 February 2026, replacing 21 CFR Part 820 and incorporating ISO 13485:2016 by reference. [secondary]
- **ALCOA+ / GxP data integrity** — MHRA *GXP Data Integrity Guidance* (Rev 1, March 2018); **PIC/S PI 041-1** (effective 1 July 2021), the most broadly adopted; FDA CGMP data integrity Q&A (final, December 2018). ALCOA+ maps onto technical controls: *Attributable/Contemporaneous* → signed, server-timestamped immutable audit log; *Original* → retain raw source plus verified true copy, never only derived features; *Enduring/Available* → readability over a retention period that for biologics may outlive your ML framework. [secondary]
- **⚠️ Draft EU GMP Annex 22 (Artificial Intelligence)** — published for consultation 7 July 2025 alongside a heavily revised Annex 11; consultation closed 7 October 2025; **final text targeted Q4 2026, not in force.** As drafted it would restrict AI in GMP-critical applications to **static, deterministic models**, excluding dynamic/continuously-learning models and LLMs. **If any countermeasure manufacturing decision path assumes a continuously-learning model, that architecture is at risk.** [secondary]

### 8.2 Data protection and the EU

- **⚠️ EU AI Act high-risk obligations were DEFERRED.** Regulation (EU) 2024/1689 was amended by **Regulation (EU) 2026/1744** (the "Digital Omnibus on AI"), published in the OJ 24 July 2026, in force 27 July 2026 — six days before the original deadline. **Annex III standalone high-risk moved from 2 August 2026 to 2 December 2027; Annex I (including AI in MDR/IVDR devices) moved to 2 August 2028.** Prohibited practices (since February 2025), GPAI obligations (since August 2025), and Article 50 transparency duties were **not** deferred. [verified independently]
- **Most clinical AI reaches high-risk via Annex I, not Annex III**, because an AI-enabled medical device already requires third-party conformity assessment under MDR/IVDR. Article 9 and 17 map substantially onto ISO 14971 and ISO 13485; **Article 10 (data and data governance) has no clean MDR analogue and is the genuine new work**; Article 12 automatic logging is a hard architectural requirement. [secondary]
- **GDPR.** Federation does not remove you from scope — gradients, embeddings, and weights derived from personal data are frequently still personal data. The operative provisions: Art. 5(1)(b) + Art. 89(1) research compatibility with safeguards; Art. 9(2)(j) for health and genetic data in research; Art. 35 DPIA (effectively mandatory here); Chapter V transfers, the binding constraint on cross-border consortia; and **Art. 26 joint controllership** — federated consortia usually create joint controllership, not processorship, so settle the Art. 26 arrangement before the first training round. See EDPB **Opinion 28/2024** on AI models and anonymity, and **Guidelines 01/2025 on pseudonymisation**. [secondary]
- **⚠️ The GDPR "Data Omnibus" is NOT adopted.** Only the AI Omnibus passed. The proposed GDPR amendments — including making research processing always purpose-compatible — remain in negotiation, and the **EDPB/EDPS Joint Opinion 2/2026** (11 February 2026) is strongly critical. **Do not design to the proposed text.** [secondary]
- **European Health Data Space — Regulation (EU) 2025/327**, in force 26 March 2025; general application 26 March 2027; **the secondary-use regime applies from 26 March 2029.** Access via national Health Data Access Bodies issuing data permits, inside a secure processing environment, with only anonymised or pseudonymised data leaving. This is the same architecture FL implies — building for HDAB-mediated access now is not premature. [secondary]

### 8.3 Biosecurity and dual-use — the largest set of changes

**⚠️ The 2024 DURC/PEPP policy has been rescinded and replaced.** The *US Government Policy for Oversight of Dual Use Research of Concern and Pathogens with Enhanced Pandemic Potential* (issued 6 May 2024, due to take effect 6 May 2025) was overtaken by **Executive Order 14292, *Improving the Safety and Security of Biological Research*, signed 5 May 2025** — one day before its effective date — which paused federal funding for dangerous gain-of-function research and directed OSTP to revise or replace it.

The replacement is the **US Government Policy for Stopping High-Risk Life Sciences Research**, approved 20 July 2026 and released 28 July 2026. [verified independently] It:

- **replaces** the 2024 DURC/PEPP policy and the earlier DURC and P3CO policies;
- **shifts from a list-based to a risk-based framework**, and from "oversee and mitigate" to **prohibit and stop** — establishing federal funding prohibitions, not only review requirements;
- introduces two categories: **Dangerous Gain-of-Function (DGOF)** research and **International Research of Concern (IROC)**;
- **restricts federally supported life-sciences research conducted outside the United States** — directly relevant if the program involves foreign collaborators, foreign sample sources, or offshore validation;
- imposes expanded review, certification, monitoring, reporting, and enforcement obligations on **all** institutions receiving federal life-sciences funding.

**The critical open question for this program.** Whether purely computational or in-silico biological design work falls within "life sciences research" under this policy **determines whether this pipeline is in scope at all**. That question is **[unresolved]** here — the policy text could not be retrieved in this environment. **Reading it in full is the first action item in §9.** Institutions are also acting beyond the federal floor: the University of California mandated a pause on work meeting the DGOF definition **regardless of funding source**, effective 29 May 2026. [secondary]

**Other biosecurity changes:**

- **⚠️ EO 14110 was rescinded** (20 January 2025, by EO 14148); EO 14179 (23 January 2025) and *Winning the AI Race: America's AI Action Plan* (July 2025) are operative. Note that **artifacts produced under EO 14110 — the NIST AI RMF, OMB AI memoranda, agency AI policies — survive its rescission.** [secondary]
- **⚠️ Nucleic acid synthesis screening is in limbo.** The OSTP *Framework for Nucleic Acid Synthesis Screening* (April 2024) made provider attestation a condition of federal research funding, but EO 14292 directed its revision or replacement within 90 days — a deadline that passed in August 2025 with **no replacement published**. The Sequences of Concern definition is expected to widen. **If any part of the pipeline emits sequences for synthesis, build provider-attestation verification and an order-level screening audit trail now, and make the SOC definition a config value rather than a design assumption.** [secondary]
- **AI biological-capability evaluation is by agreement, not regulation.** NIST **CAISI** (renamed from the US AI Safety Institute, June 2025) runs pre-deployment evaluations including biosecurity; the UK AI Security Institute does likewise. **There is no US statutory requirement to evaluate a model for biological uplift before release.** The binding obligations are the EU AI Act GPAI systemic-risk regime (Art. 55, in force since 2 August 2025) and provider-internal frameworks. If this pipeline consumes a frontier model, the enforceable safeguards sit with the provider under EU law and with you contractually. [secondary]
- **NASEM, *The Age of AI in the Life Sciences: Benefits and Biosecurity Considerations*** (2025, [NAP catalog 28868](https://nap.nationalacademies.org/catalog/28868)) assesses AI-enabled biological design tools and finds AI is also a biosecurity *strengthener*. Its most useful contribution for architecture: **risk concentrates at the AI-to-wet-lab interface — synthesis and physical execution — rather than in the model itself**, which argues for putting the hardest controls at the design-to-order boundary. [secondary]
- **⚠️ The NIH Guidelines are being replaced.** The Draft *NIH Biosafety Policy for Research Involving Biohazards* was released 19 August 2026 (Guide Notice **NOT-OD-26-112**); **the comment period closes 19 October 2026 — open now.** Scope widens from recombinant/synthetic nucleic acids to biohazards generally, with tiered risk-based oversight. The NIH Guidelines remain operative until finalized. Separately, **IBC minutes from meetings on or after 1 June 2025 must be posted publicly** — assume deliberations about this program are public. [secondary]

**Institutional gate stacking.** A countermeasure program typically passes four to five independent gates with different owners: **IBC** (containment, any wet-lab validation) → **high-risk / dual-use review** (the IRE-equivalent under the new policy) → **IRB** (human data; sIRB mandatory for NIH-funded multi-site human-subjects research, plus consent or HIPAA waiver under 45 CFR 164.512(i)) → **Select Agent registration** if applicable → **export control review**. Build these hand-offs into project intake, because the dual-use gate is the one most often discovered late.

### 8.4 Export control

- **⚠️ Biotech equipment controls tightened.** BIS Interim Final Rule, 15 January 2025, created **ECCN 3A069** (equipment) and **3E069** (technology) covering high-parameter flow cytometers and mass spectrometers designed for top-down proteomics — license required for all destinations except Country Group A:1. The rule's rationale explicitly cites the combination of this equipment with AI and biological design tools. [secondary]
- **⚠️ There is currently no general US export control on AI model weights.** The *Framework for Artificial Intelligence Diffusion* (January 2025) was **rescinded by BIS on 12 May 2025**. Compute and IC controls remain and have tightened; model-weight controls were withdrawn. If a risk register assumes weights are export-controlled, it is wrong. [secondary]
- **Deemed export is the trap.** Releasing controlled technology — including source code and technical data — to a foreign national **inside the United States** is a licensable event. Granting a foreign-national collaborator access to a repository, model weights, or controlled sequence data can require a license. **Build nationality-aware access control into the platform, not only into HR process.**
- The **fundamental research exclusion** (EAR §734.8) is lost the moment you accept publication-approval or foreign-national-access restrictions — which DoD-funded countermeasure work frequently imposes. **ITAR** (USML Category XIV) has no de minimis and no fundamental-research equivalent.

### 8.5 Management-system and AI-risk frameworks

- **ISO/IEC 42001:2023** (AI management systems) is a *management-system* standard, not a technical AI-safety standard. It gets you a defensible governance wrapper; it will not satisfy EU AI Act Annex III requirements, FDA credibility expectations, or any biosecurity control on its own. Treat it as the shell you hang substantive controls on. [secondary]
- **⚠️ ISO/IEC 27701:2025** (14 October 2025) is now a **standalone PIMS standard**, separately certifiable without ISO 27001, with scope expanded to AI-related processing and health data. [secondary]
- **ISO/IEC 27001:2022** (+ Amd 1:2024) for the ISMS. **NIST CSF 2.0** (February 2024) adds the **GOVERN** function and strengthened supply-chain outcomes — GV.SC is what auditors will use to ask who owns model-risk decisions and how third-party models and datasets are vetted. [secondary]
- **⚠️ NIST AI RMF (AI 100-1) is under active revision** — confirmed directly on the NIST AI Resource Center, which carries the notice that a revised version is in progress. No 1.1/2.0 published as of September 2026. **Pin the version and date in any compliance artifact.** The **Generative AI Profile (NIST AI 600-1**, July 2024) identifies CBRN information/capability among 12 GenAI risk areas. [verified for the revision notice; secondary otherwise]

---

## 9. Reporting and documentation

**There is no EQUATOR-registered consensus reporting guideline specific to federated learning.** Do not imply otherwise. Report against the stage-appropriate general guideline:

| Lifecycle stage | Guideline |
|---|---|
| Prediction model development/validation | **TRIPOD+AI** ([Collins et al., *BMJ* 2024;385:e078378](https://doi.org/10.1136/bmj-2023-078378)) — unifies regression and ML; adds fairness and open-science items |
| Early-stage live clinical evaluation | **DECIDE-AI** ([Vasey et al., *Nat Med* 2022;28:924–933](https://doi.org/10.1038/s41591-022-01772-9)) — human factors and clinician–AI interaction, not accuracy |
| Trial protocol | **SPIRIT-AI** ([Cruz Rivera et al., *Nat Med* 2020;26:1351–1363](https://doi.org/10.1038/s41591-020-1037-7)) |
| Trial report | **CONSORT-AI** ([Liu et al., *Nat Med* 2020;26:1364–1374](https://doi.org/10.1038/s41591-020-1034-x)) |
| Medical imaging AI | **CLAIM 2024 Update** ([Tejani et al., *Radiol Artif Intell* 2024;6(4):e240300](https://doi.org/10.1148/ryai.240300)) — cite the 2024 update, not the 2020 original |
| Baseline minimum information | **MINIMAR** ([Hernandez-Boussard et al., *JAMIA* 2020;27(12):2011–2015](https://doi.org/10.1093/jamia/ocaa088)) — a proposal, not a Delphi-consensus guideline |

All DOIs above [verified — resolve to correct publisher paths]. SPIRIT-AI is the **protocol**, CONSORT-AI the **report** — confusing the two is the most common error with that pair.

Supplement with **FUTURE-AI** ([Lekadir et al., arXiv:2309.12325](https://arxiv.org/abs/2309.12325)) — 118 experts across 51 countries, two-year modified-Delphi process, six principles (Fairness, Universality, Traceability, Usability, Robustness, Explainability) expanded into 28 best practices. It is a trustworthiness and deployment framework, not a reporting checklist. [verified]

**FL-specific reporting this platform should emit regardless:** site count, per-site sample sizes and distributions, aggregation strategy, communication rounds, privacy mechanism with unit of privacy and ε accounting, and **per-site rather than pooled performance**.

**Per artifact:** every dataset snapshot carries a **datasheet** (§4.4); every model carries a **model card** with disaggregated evaluation (§G1/O1.1); every model carries a **model info sheet** for leakage (§G1/O1.4). No release without all three.

---

## 10. Roles

| Role | Owns |
|---|---|
| Program steering | Core goals, participation terms, escalation |
| Data stewards (per site) | Local data, mapping spec, legal basis, QC outcomes |
| Contract owner | Feature contracts and versioning |
| Platform / SRE | Orchestration, SLOs, reliability, incident response |
| Security | Threat model, controls, attestation, IR |
| Privacy / legal | Legal bases, agreements, DP parameters, retention, joint-controllership arrangements |
| Regulatory | PCCP and Modification Protocol, credibility dossiers, QMS |
| Independent evaluation | Held-out benchmarks, subgroup analysis, red teaming |
| Dual-use / biosecurity review | DGOF and IROC assessment, synthesis screening, release decisions |
| Export control | Jurisdiction determination, nationality-aware access |
| Model owner | Lifecycle, documentation, retirement |

---

## 11. Implementation sequence

1. **Resolve scope exposure first (weeks 0–2).** Read the July 2026 *Stopping High-Risk Life Sciences Research* policy in full and determine whether in-silico design work is in scope, and whether IROC restrictions apply to any planned collaborator. Nothing else is worth sequencing until this is answered. Record the determination.
2. **Weeks 0–6 — contract and governance skeleton.** One narrow use case, one feature contract, the intended-use statement, the review body, the participation agreement, the Art. 26 joint-controllership arrangement. Write the threat model in NIST AI 100-2 vocabulary. If a regulated device is plausible, draft the PCCP Modification Protocol *now* — it constrains everything downstream.
3. **Weeks 4–10 — edge harmonization and QC on synthetic data.** Containerized harmonizer plus Kahn-framework gate suite; every candidate site passes the conformance suite before touching real records.
4. **Weeks 8–16 — orchestration with security from the start.** Identity, attestation, signed artifacts, secure aggregation, disclosure control, audit log, checkpointing. Retrofitting secure aggregation means re-approving every agreement.
5. **Weeks 12–20 — two- or three-site pilot on real data.** A boring model and a real held-out evaluation. The goal is a trustworthy end-to-end result, not a good metric.
6. **Weeks 18–28 — reliability and adversarial hardening.** SLOs, drills, DP budget ledger, rollback rehearsal, per-site influence monitoring.
7. **Weeks 24+ — scale participants, then model ambition.** In that order. Each new site is a conformance run, an agreement, and an export-control determination — not a config line.

**Signs the program is off track:** raw data requested "just to debug"; per-site metrics circulating informally; mapping changes without review; a benchmark the training team can see; a kill switch never exercised; a model card written after the model shipped; privacy parameters chosen to hit an accuracy target; a dual-use determination deferred to "later."

---

## 12. Summary

- Federated learning solves data movement. It does not solve semantics, quality, security, or dual-use — and the published record shows most healthcare FL work is still not fit for clinical use. Budget accordingly.
- **Feature contracts** and **edge harmonization** are the backbone of data standards; the **Kahn conformance/completeness/plausibility** framework, with its verification-vs-validation axis, is the quality vocabulary. Do not invent parallel schemes.
- Treat the **coordinator as untrusted**, make **egress statistical disclosure control**, and enforce **bounds** so no participant can move the model far. State precisely what secure aggregation and DP each guarantee — and always declare the **unit of privacy**.
- Robust aggregation is **defense in depth with contested guarantees**, not a security argument. The controls that survive any threat model are enrollment vetting, bounds, influence monitoring, and independent evaluation.
- **Partial participation is normal**: quorum rounds, checkpoints, idempotent stages, SLOs with error budgets, rehearsed drills. Statistical heterogeneity, not availability, is the dominant performance risk.
- **Governance sits in the code path**: intended-use gating, release review, datasheets, model cards, leakage info sheets, tamper-evident logs, rehearsed isolation.
- **The regulatory ground moved in 2025–2026.** The 2024 DURC/PEPP policy is replaced; EO 14110 is rescinded; EU AI Act high-risk obligations are deferred to 2027/2028; the AI Diffusion Rule is withdrawn; nucleic acid synthesis screening awaits replacement; NIH Guidelines are being rewritten. Pin versions, cite dates, re-verify.
- Integrity and privacy objectives get **no error budget**. Everything else is negotiable against delivery speed.

---

## Appendix — verification status and open items

**Environment limitation.** This review was conducted from a container whose egress proxy blocked most standards-body, regulator, and publisher domains — including nature.com, PMC, ScienceDirect, hl7.org, cdisc.org, dicom.nema.org, w3.org, nist.gov, fda.gov, whitehouse.gov, eur-lex.europa.eu, and iso.org. Reachable: arxiv.org, doi.org (resolution), raw.githubusercontent.com, and search indexes. Consequently many citations are **[secondary]** — confirmed via DOI resolution and official-page indexing with independent corroboration, but not read in full. **Widening the network allowlist would materially raise confidence in §4.1 and §8.**

**Open items, in priority order:**

1. **Read the July 2026 *Stopping High-Risk Life Sciences Research* policy in full** — specifically the DGOF and IROC definitions and whether in-silico design is in scope. Highest-stakes unresolved item in this document.
2. **Verify CDISC version numbers** directly at cdisc.org (§4.1) — currently low confidence.
3. **Confirm whether FDA's January 2025 AI-in-drug-development draft was finalized** — sources conflicted.
4. **Verify SLSA v1.2 level semantics** first-hand at slsa.dev.
5. **Track the replacement nucleic acid synthesis screening framework**; keep the SOC definition configurable.
6. **NIH Draft Biosafety Policy comment window closes 19 October 2026.**
7. **Verify the "5.2% of healthcare FL studies are real deployments" figure** before using it anywhere.
