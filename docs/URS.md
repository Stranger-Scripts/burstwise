# Burstwise — User Requirements Specification (URS)

| Field | Value |
|---|---|
| Document ID | BW-URS-001 |
| Version | 0.1 (draft) |
| Date | 2026-11-02 |
| Owner | Gabriel |
| Status | Draft — not reviewed, not approved |
| System | Burstwise — phage amplification planning, run documentation and analysis |
| Regulatory frame | R&D under a quality system (not GMP release testing) |
| Software category | Custom application (GAMP 5 Category 5) |

---

## 1. Purpose and scope

Burstwise replaces two Excel tools:

- **Phage Amplification Planner** — MOI/OD600/yield calculator, MOI scan, OD600 calibration (k) and growth-curve (t_d) fits.
- **Phage Amplification Analysis** — run log with derived values, correlations, binned means, per-stock summaries and top-N ranking.

Burstwise combines planning and analysis in one application, documents the full amplification workflow from the overnight culture to the titered pool, and makes results comparable across projects that share the same phage and host.

**Main deficit of the Excel tools addressed here:** they cannot distinguish between phage starting material from different supplier lots, so lot effects are mixed into parameter effects.

**In scope (v1):** see requirements with priority **M** and **S**.
**Out of scope (v1):** see section 22.

## 2. Requirement conventions

| Column | Meaning |
|---|---|
| ID | `URS-<AREA>-<nnn>`; IDs are never reused or renumbered |
| Prio | **M** = must, v1 · **S** = should, v1 · **L** = later version |
| GxP | **Y** = data-integrity or quality relevant; must be covered by risk assessment and verification |

Requirements state *what* is needed, not *how*. Design decisions referenced in notes are informative.

## 3. Definitions

| Term | Definition |
|---|---|
| Organism | A phage or a host strain, identified by a culture-collection accession |
| Accession | Collection + number (e.g. DSM …, ATCC …) |
| Supplier lot | One delivery of phage starting material from a supplier/collection |
| Project | One amplification campaign; starts from exactly one supplier lot |
| Overnight culture | Host pre-culture prepared on day 1 |
| Cultivation | The day-2 session in which batches are inoculated from one overnight culture |
| Batch | One broth culture that is infected (spiked) with phage; has an internal number |
| Infection | The spiking of a batch with phage at a measured OD600 |
| Harvest | Processing and cleanup of a batch after incubation |
| Pool | Combination of batches with identical conditions (per condition key) |
| Condition key | The set of factors, chosen per project, that defines which batches are pooled together |
| Stock | Any titered phage material that can be used as a source (supplier lot, batch, pool) |
| Titer measurement | A titer value with target (lot/batch/pool/stock) and stage |
| Stage | Point in the process at which a titer was measured: `receipt`, `post_cleanup`, `pre_freeze`, `post_thaw` |
| Root lot | The supplier lot from which a stock descends through lineage |
| Passage depth | Number of amplification generations between a stock and its root lot |
| Parameter set | Versioned set of host–phage parameters (k, B, safety factor, t_d, L, adsorption allowance, OD_max) |
| Event | Immutable, attributable record of a data change; source of truth and audit trail |
| Client | One Burstwise installation, identified by a client ID |

## 4. Process overview

| Day | Step | Burstwise record |
|---|---|---|
| 1 | Overnight culture of the host | Overnight culture |
| 2.1 | X mL overnight culture inoculates Y batches of V mL broth | Cultivation, batches |
| 2.2 | Each batch is spiked with phage at a predetermined OD600 — **planner used per batch** | Infection |
| 3 | Batches are processed/cleaned up; batches with identical conditions are pooled (one or more pools) | Harvest, pool |
| 3 (+) | Pool is prepared for freezing; after freezing, an aliquot is thawed and titered | Freeze, titer measurement |

Titers are occasionally also measured per batch (always **after** cleanup) and per pool before freezing.

---

## 5. General and platform (GEN)

| ID | Requirement | Prio | GxP |
|---|---|---|---|
| URS-GEN-001 | The system shall run as a desktop application on Windows and Linux. | M | N |
| URS-GEN-002 | The system shall be fully usable without a network connection (offline-first). Synchronisation shall happen when a connection to the shared location is available. | M | N |
| URS-GEN-003 | One database shall hold many projects together with shared master data (organisms, parameter sets, additives, media). | M | N |
| URS-GEN-004 | All quantities shall be displayed and entered with their units. | M | Y |
| URS-GEN-005 | Numeric input shall accept scientific notation (e.g. `5e9`) and reject invalid or out-of-range values with a visible message. | M | Y |
| URS-GEN-006 | The system shall display its version and the active calculation-model version. | M | Y |

## 6. Organism identity (ORG)

| ID | Requirement | Prio | GxP |
|---|---|---|---|
| URS-ORG-001 | Phages and host strains shall be identified by culture-collection accession (collection + number). | M | Y |
| URS-ORG-002 | Each accession shall create its own organism record by default. | M | Y |
| URS-ORG-003 | Two or more accessions of the same organism (cross-deposits) shall only be treated as one organism after an explicit, audited link with a stated reason. | M | Y |
| URS-ORG-004 | A link between accessions shall be reversible by an audited unlink with a stated reason. | S | Y |
| URS-ORG-005 | The comparison key for cross-project analysis shall be (phage organism, host organism). | M | Y |

## 7. Master data and parameters (MD)

| ID | Requirement | Prio | GxP |
|---|---|---|---|
| URS-MD-001 | The system shall store host–phage parameter sets: k (CFU/mL per OD600), B (burst size), safety factor, t_d, latent period L, adsorption allowance, OD_max. | M | Y |
| URS-MD-002 | Parameter sets shall be versioned. Existing versions shall never be modified; a change creates a new version with a reason. | M | Y |
| URS-MD-003 | Each parameter value shall carry its provenance: manual entry, OD calibration dataset, growth dataset, or derived from runs (with reference). | M | Y |
| URS-MD-004 | The system shall store OD600 calibration datasets (OD600 vs CFU/mL) per host strain and photometer. | M | Y |
| URS-MD-005 | The system shall store growth datasets (time vs OD600) per host strain and medium. | M | Y |
| URS-MD-006 | The system shall maintain a vocabulary of additives (name, default unit) and media. | M | N |
| URS-MD-007 | Master data shall be reusable across projects without re-entry. | M | N |

## 8. Projects and supplier lots (PRJ)

| ID | Requirement | Prio | GxP |
|---|---|---|---|
| URS-PRJ-001 | Each project shall reference exactly one root supplier lot. | M | Y |
| URS-PRJ-002 | A supplier lot shall by default be usable as root of only one project. The design shall allow relaxing this rule without restructuring data. | M | Y |
| URS-PRJ-003 | A supplier lot shall record: phage organism, supplier/collection, catalog/accession, supplier lot or order number, received date, certificate titer (if given). | M | Y |
| URS-PRJ-004 | The titer measured on receipt shall be recordable as a titer measurement with stage `receipt`. | M | Y |
| URS-PRJ-005 | Each project shall have an owner (user and client). Ownership transfer shall be an explicit, audited action. | M | Y |
| URS-PRJ-006 | Each project shall define its condition key (pooling factors) from a fixed vocabulary (e.g. additives with concentration, planned MOI, OD target, medium, incubation time). | M | Y |
| URS-PRJ-007 | Changing the condition key after pools exist shall require a reason and shall be audited. | M | Y |

## 9. Internal numbering (NUM)

| ID | Requirement | Prio | GxP |
|---|---|---|---|
| URS-NUM-001 | Batches, pools and other derived stocks shall receive a machine-generated, human-readable internal number. | M | Y |
| URS-NUM-002 | The numbering format shall be configurable via a template (e.g. `{project}-{kind}{seq:02}`). | M | N |
| URS-NUM-003 | Internal numbers shall be unique across all clients, including when clients work offline. | M | Y |
| URS-NUM-004 | A change of the numbering template shall apply only to new numbers; existing numbers shall never be changed. | M | Y |

## 10. Workflow records (WF)

| ID | Requirement | Prio | GxP |
|---|---|---|---|
| URS-WF-001 | Overnight culture: host organism, medium, date/time, operator. | M | Y |
| URS-WF-002 | Cultivation: project, overnight culture, inoculum volume, number of batches, date. | M | Y |
| URS-WF-003 | Batch: broth volume, medium, additives (name + concentration + unit), internal number. | M | Y |
| URS-WF-004 | Infection (per batch): measured OD600, time of infection, time from inoculation to infection, source stock, planned MOI, planner snapshot (URS-PLN-008), actual spike volume. | M | Y |
| URS-WF-005 | The actual MOI shall be calculated from spike volume, source titer, OD600, k and broth volume; it shall not be entered manually. | M | Y |
| URS-WF-006 | Harvest (per batch): time, incubation time after infection, cleanup parameters (e.g. centrifuge speed and time, filtration), volume after cleanup. | M | Y |
| URS-WF-007 | The system shall propose pools by grouping batches with identical values of the project's condition key. | S | Y |
| URS-WF-008 | Pool: input batches with contributed volume each, condition-key values, final pool volume. A deviation from the proposed grouping shall require a reason. | M | Y |
| URS-WF-009 | Freeze: pool, date, aliquot information. | M | Y |
| URS-WF-010 | Shaker speed and incubation temperature shall be recordable per cultivation or batch. | M | Y |

## 11. Titer measurements (TIT)

| ID | Requirement | Prio | GxP |
|---|---|---|---|
| URS-TIT-001 | A titer measurement shall record: target (supplier lot, batch, pool, stock), stage, value (PFU/mL), method, operator, date, reference to the original record (lab notebook or raw-data location). | M | Y |
| URS-TIT-002 | Stages shall at least include `receipt`, `post_cleanup`, `pre_freeze`, `post_thaw`; the list shall be extensible. | M | Y |
| URS-TIT-003 | Multiple measurements per target and stage shall be possible (repeat titrations). | M | Y |
| URS-TIT-004 | Each measurement shall record its source: `manual_entry` (v1) or `computed_from_raw` (reserved). | M | Y |
| URS-TIT-005 | Raw titration data (plaque counts, dilutions, plated volume, replicates) shall be capturable and the titer computed from it. | L | Y |

## 12. Lineage (LIN)

| ID | Requirement | Prio | GxP |
|---|---|---|---|
| URS-LIN-001 | Every stock shall be traceable to its root supplier lot. | M | Y |
| URS-LIN-002 | Lineage shall support multiple parents (pools are formed from several batches). | M | Y |
| URS-LIN-003 | All inputs to a pool shall share the same root lot; the system shall reject violations. | M | Y |
| URS-LIN-004 | Passage depth shall be derived automatically from lineage. | M | Y |
| URS-LIN-005 | A batch or pool shall be usable as source stock for further amplification within the project. | M | Y |
| URS-LIN-006 | The lineage of a stock shall be viewable as a graph or tree. | S | N |

## 13. Planner (PLN)

All formulas shall reproduce the Excel planner (see Appendix A).

| ID | Requirement | Prio | GxP |
|---|---|---|---|
| URS-PLN-001 | Yield side: B_design, N_required, OD_needed, P_max, headroom, yield verdict, total phage produced. | M | Y |
| URS-PLN-002 | Cost side: cells/mL at infection, total cells, P_0, total phage needed, stock volume (mL and µL), carryover (v/v), carryover verdict. | M | Y |
| URS-PLN-003 | Kinetics: fold amplification F, rounds r, time to target, phage and host exponential rates, race ratio, kinetics verdict, single-cycle vs multi-cycle note. | M | Y |
| URS-PLN-004 | MOI scan over a configurable list of MOI values with stock volume, carryover, F, rounds (floored at 1), time and verdict per row. | M | Y |
| URS-PLN-005 | Verdict thresholds (carryover 5 % / 10 %, race ratio 1 / 2, MOI 10, time 8 h, yield headroom) shall be named constants, displayed with the result and versioned with the calculation model. | M | Y |
| URS-PLN-006 | The planner shall be usable per batch at the batch's measured OD600. | M | Y |
| URS-PLN-007 | Parameters shall be taken from a selected parameter-set version; manual override for a single plan shall be possible and marked. | M | Y |
| URS-PLN-008 | When an infection is recorded, the planner inputs, outputs and calculation-model version shall be stored as a snapshot. | M | Y |
| URS-PLN-009 | The model assumptions (complete adsorption, uniform burst size, no lysis-from-without, no phage decay, no resistance) shall be shown with the results. | M | Y |

## 14. Fits (FIT)

| ID | Requirement | Prio | GxP |
|---|---|---|---|
| URS-FIT-001 | k shall be calculated from a calibration dataset as fit through the origin (preferred) and OLS slope with intercept and R² (for comparison). | M | Y |
| URS-FIT-002 | t_d shall be calculated from a growth dataset by log-linear regression (slope, R², t_d in min and h). | M | Y |
| URS-FIT-003 | Fits shall require at least 3 points and shall warn when R² < 0.99 (growth) or when through-origin and OLS k differ notably (calibration). | M | Y |
| URS-FIT-004 | A fit result shall be adoptable into a new parameter-set version with provenance (URS-MD-003). | M | Y |
| URS-FIT-005 | Fits shall be shown as plots with data points and fit line. | S | N |

## 15. Analysis (ANA)

| ID | Requirement | Prio | GxP |
|---|---|---|---|
| URS-ANA-001 | Every analysis shall state which titer stage is used as outcome. Stages shall never be mixed within one analysis. Default: `post_thaw` pool titer. | M | Y |
| URS-ANA-002 | Within-project view: comparison of the pools of one project by their condition-key values. | M | Y |
| URS-ANA-003 | Cross-project view: all projects with the same (phage, host) key, stratified by project (= root lot). | M | Y |
| URS-ANA-004 | Cross-project correlations shall be computed on values centred on their project mean (within-project correlation). Pooled correlations without stratification may be shown only labelled as such. | M | Y |
| URS-ANA-005 | Every statistic shall report n (data points) and the number of projects contributing; results based on fewer than 8 data points shall be flagged. | M | Y |
| URS-ANA-006 | Derived metrics: amplification factor (output PFU / input PFU), log10 amplification, log10 titer, apparent burst (output PFU / cells at infection), log10 amplification per hour. | M | Y |
| URS-ANA-007 | Pool-level metrics shall aggregate the input batches (Σ input PFU, Σ cells at infection) using the final pool volume for output PFU. | M | Y |
| URS-ANA-008 | Freeze recovery (log10 post_thaw / pre_freeze) shall be calculated for pools with both measurements. | M | Y |
| URS-ANA-009 | Pooling mass balance shall be calculated for pools with post-cleanup batch titers: Σ(batch titer × contributed volume) vs pre-freeze pool titer × final pool volume. | S | Y |
| URS-ANA-010 | Predicted vs observed: titer, amplification and time from the planner snapshot vs measured outcome, as log10 residual. | M | Y |
| URS-ANA-011 | Effective burst size and effective safety factor per (phage, host) shall be derivable from runs and adoptable into a new parameter-set version (URS-MD-003). | S | Y |
| URS-ANA-012 | Pearson correlations (on log10 outcome) for MOI, OD at infection, cells/mL, source titer, volume, incubation times, shaker speed, temperature, centrifuge settings, additive concentration. | M | Y |
| URS-ANA-013 | Binned means with user-editable bin edges for MOI, OD at infection and incubation after infection. | M | N |
| URS-ANA-014 | Additive comparison: with vs without (and by concentration), preferably within project. | M | Y |
| URS-ANA-015 | Per-lot (= per-project) summary: receipt QC (certificate vs measured titer), mean log10 amplification, apparent burst. | M | Y |
| URS-ANA-016 | Top-N ranking by log10 amplification and by log10 titer. | S | N |
| URS-ANA-017 | Passage depth shall be available as an analysis factor. | S | Y |
| URS-ANA-018 | Spearman correlation, confounding views, mixed-effects models. | L | Y |

## 16. Audit trail and data integrity (AUD)

| ID | Requirement | Prio | GxP |
|---|---|---|---|
| URS-AUD-001 | Every creation, change and retirement of a record shall be stored as an immutable event: who, what, when, old value, new value, reason. | M | Y |
| URS-AUD-002 | A change to an existing record shall require a reason. | M | Y |
| URS-AUD-003 | Records shall not be physically deleted; retirement is an audited event with a reason. | M | Y |
| URS-AUD-004 | The audit trail shall be generated by the system and cannot be disabled or edited by users. | M | Y |
| URS-AUD-005 | Events shall be hash-chained per client so that alteration or removal is detectable. | M | Y |
| URS-AUD-006 | The system shall provide an integrity check of the event chain and report the result. | M | Y |
| URS-AUD-007 | The audit trail shall be viewable and exportable per record, project and time range. | M | Y |
| URS-AUD-008 | Timestamps shall be recorded in UTC with local offset; events shall also carry a logical clock for ordering across clients. | M | Y |

## 17. Users, signatures and review (SEC)

| ID | Requirement | Prio | GxP |
|---|---|---|---|
| URS-SEC-001 | User attribution shall use the operating-system account (stable identifier: Windows SID, Linux uid) plus display name. | M | Y |
| URS-SEC-002 | Each user shall enrol an application PIN/password; it shall be stored only as a salted slow hash. | M | Y |
| URS-SEC-003 | Roles: analyst, reviewer, administrator. Permissions shall be role-based. | M | Y |
| URS-SEC-004 | Signatures shall require re-entry of the PIN and record name, date/time and meaning (e.g. "reviewed", "approved"). | M | Y |
| URS-SEC-005 | Records shall have a status (draft → submitted → reviewed/approved). Approved records can only be changed through a new version with reason. | M | Y |
| URS-SEC-006 | A user shall not review or approve their own record. | S | Y |
| URS-SEC-007 | When a result is signed, the calculated values and the calculation-model version shall be stored with it. | M | Y |

## 18. Calculation integrity (CALC)

| ID | Requirement | Prio | GxP |
|---|---|---|---|
| URS-CALC-001 | Every calculation shall be associated with a calculation-model version. | M | Y |
| URS-CALC-002 | Previous calculation-model versions shall remain available to reproduce historic results. | M | Y |
| URS-CALC-003 | The system shall be able to recompute a historic result with its original model version and compare it to the stored value. | S | Y |
| URS-CALC-004 | The calculation core shall be verified against the Excel tools with defined test cases (parity tests). | M | Y |

## 19. Synchronisation, storage and backup (SYN)

| ID | Requirement | Prio | GxP |
|---|---|---|---|
| URS-SYN-001 | Data shall be stored locally on each client and synchronised through a shared folder (SMB) in v1. | M | Y |
| URS-SYN-002 | No database file shall be opened for writing on the shared folder. Each client shall write only its own files there. | M | Y |
| URS-SYN-003 | Files on the shared folder shall be written atomically; partially written files shall not be read. | M | Y |
| URS-SYN-004 | Concurrent conflicting changes shall be shown to a user for resolution with a reason; they shall not be resolved silently. | M | Y |
| URS-SYN-005 | The local database shall be fully rebuildable from the synchronised event data. | M | Y |
| URS-SYN-006 | Sync status (last sync, pending events, errors) shall be visible. | M | N |
| URS-SYN-007 | Backup of the shared folder shall be sufficient to restore all data; restore shall be tested and documented. | M | Y |
| URS-SYN-008 | An HTTP sync server as alternative transport. | L | Y |

## 20. Import and export (IO)

| ID | Requirement | Prio | GxP |
|---|---|---|---|
| URS-IO-001 | Historic runs from the Excel analysis tool shall be importable. | S | Y |
| URS-IO-002 | Imported records shall be marked as legacy, with the source file and row, and shall be distinguishable in analyses. | S | Y |
| URS-IO-003 | Data shall be exportable as CSV and xlsx, with units in the headers. | M | N |
| URS-IO-004 | Exports shall record who exported what and when. | S | Y |

## 21. User interface (UI)

| ID | Requirement | Prio | GxP |
|---|---|---|---|
| URS-UI-001 | Verdicts (OK / borderline / fail) shall be visually distinct and never rely on colour alone. | M | N |
| URS-UI-002 | Plots: growth fit, calibration fit, MOI scan, analysis scatter and binned means. | S | N |
| URS-UI-003 | Calculated fields shall be visually distinct from input fields and not editable. | M | Y |

---

## 22. Out of scope for v1

- Raw titration data capture (URS-TIT-005)
- HTTP sync server (URS-SYN-008)
- Spearman, confounding views, mixed-effects models (URS-ANA-018)
- Web / mobile client
- Instrument interfaces (photometer, plate reader)
- Phage decay and late host resistance in the planner model

## 23. Open points

| # | Point | Affects |
|---|---|---|
| OP-01 | Units for additive concentration: fixed per additive or free per entry? | URS-MD-006, WF-003 |
| OP-02 | Historic Excel runs have no supplier-lot attribution: assign a placeholder lot per stock name, or import as unlinked legacy data? | URS-IO-001/002 |
| OP-03 | Who holds the reviewer role, and is review mandatory for every project or only for selected records? | URS-SEC-003/005 |
| OP-04 | Data retention period and archiving of closed projects. | URS-SYN-007 |
| OP-05 | Is the batch volume after cleanup always equal to the volume contributed to the pool? | URS-WF-006, WF-008, ANA-009 |
| OP-06 | Which photometer(s) are used — does k need to be tracked per instrument? | URS-MD-004 |
| OP-07 | PIN complexity, lockout after failed attempts, and session timeout rules. | URS-SEC-002/004 |

---

## Appendix A — Calculation reference (from the Excel planner, model v1)

```
B_design   = B / safety_factor
N_required = P_target / B_design                 cells/mL to be lysed
OD_needed  = N_required / k
P_max      = B_design · k · OD_max               ceiling for host + medium
headroom   = P_max / P_target                    < 1 → not reachable
N_inf      = OD_inf · k                          cells/mL at infection
P_0        = MOI · N_inf                         input titer needed
stock_vol  = P_0 · V / P_stock                   mL
carryover  = stock_vol / V                       v/v
F          = P_target / P_0                      fold amplification
r          = ln(F) / ln(B_design)                rounds (MOI scan: max(1, r))
time       = r · (L + adsorption) / 60           h, excludes pre-infection growth
race ratio = [ln(B_design) / L] / [ln(2) / t_d]  must exceed 1
```

Analysis (per run in Excel; per batch / pool in Burstwise):

```
cells/mL at infection = OD600 · k
total cells           = cells/mL · broth volume
input PFU             = MOI · total cells          (Burstwise: spike volume · source titer)
output PFU            = titer · volume
amplification factor  = output PFU / input PFU
apparent burst        = output PFU / total cells
log10 amp per hour    = log10(amplification factor) / incubation after infection
```

Fits:

```
k (origin)  = Σ(OD·CFU) / Σ(OD²)
k (OLS)     = slope of CFU vs OD, plus intercept and R²
t_d         = ln(2) / slope of ln(OD600) vs time
```

## Appendix B — Traceability

Every requirement ID shall be referenced in the functional specification and in at least one test case. The parity tests (URS-CALC-004) serve as OQ evidence for the calculation core.
