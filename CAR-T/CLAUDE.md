# CLAUDE.md

Project instructions for Claude Code. Read this fully before proposing changes.

## What this is

A reanalysis of CD19 CAR T-cell single-cell data (scRNA-seq + CITE-seq + TCR-seq) from
Cao et al., *Nature Communications* 2025, GEO `GSE207192`.

The question being answered:
> Track infusion-product clones into blood after infusion, and test the paper's claim that
> expansion-phase and persistence-phase clones derive from different product precursors.

Deliverable: a Quarto document (`reanalysis-cao-2025.qmd`) rendered to a single
self-contained HTML report.

## Study design (from the paper's schematic — do not re-derive)

Seven patients, **all complete responders**. There is no responder / non-responder contrast
in this dataset; the question it supports is clone *fate*, not response.

| Phase | Days | Notes |
|---|---|---|
| IP | infusion product | unsorted; contains untransduced T cells |
| T_exp | d8–12 | expansion peak |
| T_per1 | d21 | present for all 7 patients |
| T_per2 | d28 | absent for P03 |

Post-infusion cells were sorted **CD3+ CD19-tetramer+**. Schedule deviations in the deposited
data: P04's T_per2 is d30 (keep it); P06's T_exp is d14 (moot, P06 has 7 product cells and is
excluded); P02 d90 and P02/P03 d05 are outside the design (exclude).

**Never use a two-phase peak-vs-persistence framing.** It discards d21 entirely — 20,593 cells,
and the deepest timepoint for three patients. Always pick the *deepest* library within each
window, not the earliest or latest: an earliest-first rule selects near-empty d05 libraries
(P49 peak = 1 cell), and a latest-first rule picked P02 d90 (631 cells) over d28 (5,609).

## Join keys and identifiers — the two traps

1. **Patient IDs differ by stage.** Pre-infusion uses `031`, post-infusion uses `P031`. Join on
   `patientShort` (P01–P07), which is present in both and verified consistent. Joining on
   `patient` silently returns zero overlap.
2. **TCR barcodes are Cell Ranger `aggr` output.** The sample prefix is dropped and a library
   suffix appended (`-1`, `-2`, …). 10x reuses the same barcode list in every library, so
   joining on the 16-nt barcode alone produces ~900 false matches. Join on barcode **plus**
   suffix, after mapping suffix → library from the contig table's own `donor`/`origin`/`time`
   columns. Suffixes 1–23 are post-infusion, 24–30 are the products.

## Method decisions that must hold

- **Normalization:** `LogNormalize` for RNA, `CLR` (margin 2) for ADT. SCTransform was ruled
  out to keep peak memory manageable on a 16 GB M1. BPCells considered and deferred.
- **Integration:** Harmony on 30 PCs, correcting for **patient**, not sample. Patient-level
  correction removes between-patient differences while keeping day-to-day change, which is the
  biology of interest.
- **Variable genes exclude** TCR V/J/C segments (they track clonotype, not state), `MTRNR2L`
  pseudogenes and hemoglobin genes. `YESCAR` is kept — CAR expression level is of interest.
- **Clone definition:** patient + top-UMI TRB CDR3 nucleotide sequence. Cell Ranger's
  `raw_clonotype_id` was built from unfiltered chains; prefer a key from the kept chains.
- **Clone frequency testing:** use the **exact conditional (Fisher/hypergeometric)** test, never
  a binomial with the product frequency treated as known. Product frequencies come from ~2–4k
  sampled cells; the naive binomial returned 64% more hits (148 vs 90).
- **Call contraction only when expected ≥ 3**, so P(obs = 0 | stable) ≤ 0.05.
- **Clone-fate questions are tested one observation per clone, with patient adjustment.** Never
  cell-level. Cells within a clone are clonal descendants; cell-level Fisher tests on this data
  give FDRs down to 1e-18 that vanish entirely (p = 0.52) under clone-level testing.
- **Always check pooled against within-patient** correlations. Simpson's paradox bites here
  repeatedly (pooled rho 0.37 → within-patient 0.04).

## Data properties that constrain the analysis

- **The "CAR" ADT feature is undocumented.** The Methods list 31 TotalSeq-C reagents (28
  antibodies + 3 isotypes); the matrix has 32 features. Its signal appears only in the
  P-prefixed pre-infusion batch. Treat as unusable. Use `YESCAR` RNA for CAR status.
- **CD3E ADT is absent after infusion by design** — the CITE-seq anti-CD3ε (UCHT1) was omitted
  from the post-infusion panel because those cells were stained with BV421 anti-CD3ε (SK7) for
  sorting. CAR and CD3E were dropped from the post ADT assay and CLR re-run.
- **Selection confounds contraction.** The product is unsorted; post-infusion is CAR+ sorted.
  An untransduced clone cannot appear post-infusion and would be misread as contracting. CD8
  effector cells — the expanded compartment — show about half the CAR detection at matched
  sequencing depth.
- **Contamination survives the sort** in some libraries: monocytes (up to 53% of cells in
  116-d21) and an NK cluster post-infusion. Both are removed; see the QC section.
- **Two pre-infusion batches.** The P-prefixed samples (P033/P034/P049) differ from the rest in
  naming, mitochondrial fraction and CAR ADT presence.

## Known fragility

- **scDblFinder is not reproducible run-to-run** despite `set.seed()`, which shifts downstream
  cell counts by ~0.1% and perturbs clustering. Cluster-number → label annotation is therefore
  brittle: changing upstream randomness can silently mislabel clusters. If doublet calling is
  changed, **re-derive the annotations** rather than assuming the old map holds.
- Numbers quoted in the report prose are inline R expressions for this reason. Do not hardcode
  computed values into the narrative.

## Guardrails

- Run R in the user's live Positron session, chunk by chunk, not via `Rscript`.
- `rm()` + `gc()` intermediates after each step; this runs on a 16 GB machine.
- Do not ship input data. `data/` is gitignored; everything is regenerable from GEO.
