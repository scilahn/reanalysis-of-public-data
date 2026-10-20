# Reanalysis of public data

This is a repo of re-analyses of published sc/snRNA-seq datasets by indication.

## `CAR-T/`

scRNA-seq + CITE-seq + TCR-seq of CD19 CAR T cells, tracking clones from the infusion product
into blood across three post-infusion phases.

The cohort is seven complete responders, each with the residual infusion product plus three
sorted blood draws — expansion (d8–12), persistence 1 (d21) and persistence 2 (d28). Because
every patient responded, the question this supports is clone *fate*, not response.

The headline finding is negative, and it is a negative about method as much as biology. The
reference argues that the clones dominating the expansion peak and those dominating
persistence derive from different infusion-product precursors. Mapping fate-classified clones
back to product transcriptional clusters appears to confirm it emphatically — CD8 effector
precursors enriched in persistence-biased clones, Tc2-like precursors depleted, at cell-level
FDRs down to 1e-18. None of it survives. Cells within a clone are clonal descendants rather
than independent observations, and fate class is unbalanced across patients; testing one
observation per clone with patient adjustment leaves nothing significant across clusters,
effector/memory marker genes, or the CITE-seq panel. The same pattern recurs elsewhere in the
analysis: a product-phenotype association that looks strong pooled (Spearman 0.37) collapses
within patients (0.04), which is Simpson's paradox rather than signal.

One limit is structural rather than statistical. Clones large enough to track reliably are by
construction the clonally expanded ones, and those sit in effector and proliferating clusters —
the memory clusters contribute one and three cells. The memory-precursor hypothesis therefore
cannot be tested in this design at any sample size; it needs clone tracking that does not
condition on product abundance, such as bulk TCR-seq. A suggestive association does survive —
clones with lower CAR transgene expression in the product persist better, with a clean
dose-response — but it weakens under stricter measurement filters and is offered as a
hypothesis, not a result.

The reanalysis also documents several properties of the deposited data that constrain what it
can answer: the CAR and CD3 CITE-seq channels are effectively absent after infusion (the
anti-CD3 antibody was deliberately omitted, and the panel's "CAR" feature is not among the 31
documented reagents), monocyte and NK contamination survives the CD3+ sort in some libraries,
and CAR status after infusion rests on the sort design rather than any measured signal — though
the CAR transgene is recoverable from the RNA.

- **Deliverable:** [`reanalysis-cao-2025.qmd`](CAR-T/reanalysis-cao-2025.qmd) and its rendered, self-contained [`reanalysis-cao-2025.html`](CAR-T/reanalysis-cao-2025.html). GitHub will not render the HTML inline — download it and open it in a browser, or view it through a raw-HTML viewer.
- **Data:** GEO `GSE207192`, processed matrices only, from Cao et al., *Nature Communications* 2025.
- **Stack:** R / Seurat + Harmony + scDblFinder, authored in Quarto.

## `endometriosis/`

scRNA-seq of the Human Endometrial Cell Atlas (HECA), endometriosis cases versus controls.

The headline finding is negative. There is no endometriosis-specific cell type, which the
reference reports too, and the reanalysis goes further: the case/control expression signal
the reference reads off this atlas does not survive batch control.

The reference points to decidualized stromal cells and macrophages, and to IGF1 as the node
they share. That result does not reproduce here. The atlas pools seven source datasets that
are partly separated by condition, several of them entirely cases or entirely controls, so a
contrast pooled across them reports dataset-of-origin as disease. Fit with one pseudosample
per donor, with dataset in the model and restricted to datasets containing both arms, IGF1 is
not significant in uM2 macrophages, uM1 macrophages, or decidualized stroma, and the
macrophage compartment returns no genes at FDR < 0.05. It is not a difference of method: the notebook
reproduces the reference's own limma-voom, with their three-metacells-per-donor design, and
gets the same null. The divergence is their design and replication choices rather than the
differential expression test.

What the atlas does support is offered as leads rather than targets: CRIP1 in secretory and
luminal epithelium, the one externally-defined gene that holds its direction across both
independent datasets, and a faint TGF-beta and angiogenesis program in the functionalis
epithelium. The more transferable output is the study design the reanalysis implies for
nominating a target credibly from data like this.

The scope of the negative claim is narrow on purpose. It concerns the reference's scRNA-seq
differential expression, not its differential abundance, which was run on single-nuclei data,
and not its functional GWAS.

- **Deliverable:** [`endometriosis_analysis_richard_ahn.ipynb`](endometriosis/endometriosis_analysis_richard_ahn.ipynb) — runs top to bottom, config-driven, with the reasoning carried in markdown alongside the code.
- **Data:** ArrayExpress `E-MTAB-14039`, from Mareckova et al., *Nature Genetics* 2024 (`s41588-024-01873-w`).
- **Stack:** Python / scanpy.

## `mash/`

snRNA-seq of human liver across the MASLD spectrum — which cell types are most perturbed in
MASH versus healthy controls, by composition and by within-cell-type expression.

The source study is primarily a single-cell eQTL paper; the cell-type perturbation question
is described there but under-developed, which is the opening this analysis works in. The
cohort is 249,233 nuclei from 48 donors staged across four disease levels, from no-MASLD
controls through advanced MASH.

- **Deliverable:** [`ahn_mash_analysis.qmd`](mash/ahn_mash_analysis.qmd) and its rendered, self-contained [`ahn_mash_analysis.html`](mash/ahn_mash_analysis.html). GitHub will not render the HTML inline — download it and open it in a browser, or view it through a raw-HTML viewer.
- **Data:** GEO `GSE289173` (BioProject `PRJNA1221860`), distributed as a processed Seurat object via Zenodo (`10.5281/zenodo.14586466`), from Hong et al., *Nature Genetics* 2025 (`s41588-025-02237-8`).
- **Stack:** R / Seurat, authored in Quarto.

## Reproducing

None of these folders ship their input data — the datasets are large and publicly available
at the accessions above. Download the data for the analysis you want to run, point the config at
it, and execute the notebook or render the `.qmd`.

The `CLAUDE.md` in each folder documents that project's methods, design decisions, and the
reasoning behind them in more depth than this index does.
