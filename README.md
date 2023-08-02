# Snakemake workflow: `mashup-phage`

[![Snakemake](https://img.shields.io/badge/snakemake-≥6.3.0-brightgreen.svg)](https://snakemake.github.io)
[![GitHub actions status](https://github.com/ICTV-VBEG/mashup-phage/workflows/Tests/badge.svg?branch=main)](https://github.com/ICTV-VBEG/mashup-phage/actions?query=branch%3Amain+workflow%3ATests)


A Snakemake workflow for `louvain community partitioning of mash distances `


## Usage

The usage of this workflow is described in the [Snakemake Workflow Catalog](https://snakemake.github.io/snakemake-workflow-catalog/?usage=ICTV-VBEG%2Fmashup-phage).

If you use this workflow in a paper, don't forget to give credits to the authors by citing the URL of this (original) mashup-phagesitory and its DOI (see above).

# TODO for workflow implementation

Our general aims are to:

1. Follow the best practices regarding snakemake workflow folder structures: https://snakemake.readthedocs.io/en/latest/snakefiles/deployment.html#distribution-and-reproducibility
2. Use conda environments for automated software installation: https://snakemake.readthedocs.io/en/latest/snakefiles/deployment.html#integrated-package-management
3. Use snakemake wrappers for any steps where they are available: https://snakemake-wrappers.readthedocs.io/
4. Enable automatic inclusion of the workflow in the snakemake workflow catalog (this should work out-of-the-box, standardized usage can be enabled later): https://snakemake.github.io/snakemake-workflow-catalog/?rules=true
5. Use the example data provided for automated testing (continuous integration) of workflow functionality in a `.test/` folder and using GitHub Actions.

