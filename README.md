# BayesPrismExt

A modified version of BayesPrism package (https://github.com/Danko-Lab/BayesPrism.git)

## New run options

Two optional arguments were added to the main run functions (`run.prism` and `run.prism.st`) to control chain saving and Z coefficient-of-variation computation:

- `save.chain` (character): controls what (if anything) is written to an HDF5 chain file. Options:
	- `"none"` (default): no chains are saved (fastest).
	- `"theta"`: save posterior `theta` chains (cell-type fractions) only.
	- `"all"`: save both `theta` and `Z` chains (cell-type / cell-state expression matrices).

- `h5.file` (character): path to the HDF5 file used when `save.chain` is not `"none"`. If `NULL` and `save.chain != "none"` the default `gibbs_chain.h5` in the working directory is used.

- `compute.z.cv` (logical): whether to compute coefficient-of-variation for `Z` (`FALSE` by default). `theta.cv` is still always computed; computing `Z.cv` adds extra work and memory, so set `compute.z.cv=TRUE` only when you need per-gene Z CVs.

When `save.chain` is set to `"theta"` or `"all"`, chains are stored in HDF5 format with the following structure:

### HDF5 Chain File Structure

**Metadata:**
- `metadata`: A dataframe containing:
  - `n_samples`: Number of bulk samples
  - `n_genes`: Number of genes
  - `n_celltypes`: Number of cell types
  - `n_iterations`: Number of saved MCMC iterations
  - `save_chain`: Which parameters were saved ("theta" or "all")

**Dimension Labels:**
- `sample_names`: Character vector of bulk sample names
- `celltype_names`: Character vector of cell type names
- `gene_names`: Character vector of gene names (present when `save.chain == "all"`)

**Chain Data:**
- `theta`: Dimensions `(n_samples, n_iterations, n_celltypes)` - posterior samples of cell-type fractions for each bulk sample
- `Z`: Dimensions `(n_samples, n_iterations, n_genes, n_celltypes)` (only when `save.chain == "all"`) - posterior samples of gene-by-cell-type expression matrices

**Reading chains in R:**
Use the `read.h5.chain()` helper function with either a numeric sample index or sample name:
```r
# Read theta chain by sample index
theta_chain <- read.h5.chain("gibbs_chain.h5", sample.idx=1, param="theta")

# Or read by sample name
theta_chain <- read.h5.chain("gibbs_chain.h5", sample.idx="sample_A", param="theta")

# Read Z chain similarly
z_chain <- read.h5.chain("gibbs_chain.h5", sample.idx="sample_A", param="Z")
```

The HDF5 implementation uses chunking and compression; adjust `h5.file` or the `init.h5.gibbs` helper in `R/run_gibbs.R` if you need different compression levels or chunk sizes.
