# Reanalysis of public data

This is a repo of re-analyses of published sc/snRNA-seq datasets by indication.

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

Neither folder ships its input data — both datasets are large and publicly available at the
accessions above. Download the data for the analysis you want to run, point the config at
it, and execute the notebook or render the `.qmd`.

The `CLAUDE.md` in each folder documents that project's methods, design decisions, and the
reasoning behind them in more depth than this index does.
