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

# MASHUP workflow step-by-step
The aim of this tool is to ascertain whether MASH distances, in combination with community partitioning (e.g. Louvain or Leiden), could be used to rapidly determine strains, species and genera of bacteriophages.

A systematic evaluation of varying distance, hashes and p-value thresholds is currently underway, so the defaults are the best guess from assessment of correlation between these parameters and ANI or VIRIDIC measures of similarity.

## 1. Download dataset
The Inphared refseq genomes dataset is a subset of genome files and is much smaller than the _excluding_refseq file that I normally use:

```bash
wget https://millardlab-inphared.s3.climb.ac.uk/2Jul2023_refseq_genomes.fa.gz    
gunzip https://millardlab-inphared.s3.climb.ac.uk/2Jul2023_refseq_genomes.fa.gz
```

## 2. Data processing with [Mash](https://github.com/marbl/Mash)  
First, create sketches for records in inphared dataset, then create a file of calculated distances 

Variables include  
-k \<int>: kmer-size   
-s \<int>: sketch size - number of non-redundant min-hashes  
-i: Sketch individual sequences, rather than whole files, e.g. for multi-fastas of single-chromosome genomes or pair-wise gene comparisons  
-p \<int>: number of threads to use for processing  
-d \<num>: maximum distance to report  
```bash
mash sketch -i -k 15 -s 25000 -p 24 2Jul2023_refseq_genomes.fa
mash dist -i -d 0.3 -p 24 2Jul2023_refseq_genomes.fa.msh 2Jul2023_refseq_genomes.fa.msh > 2Jul2023.refseq.d0.3.k15.s25000.tsv
```

## 3. MASHUP
See accompanying code and requirements in

Explanation of process:
> 1. Input mash tsv file is parsed, removing self-matches where the distance will be 0, i.e. binA=binB
> 2. Matching hashes converted to a ratio
> 3. Dataframe used to create a networkx object with edge attributes of distance, matching hashes and p-value
> 4. Edges dropped from network according to user defined thresholds of distance, matching hashes and p-value
> 5. Filtered networkx object passed to Louvain paritioning
> 6. Structured outputs produced

**NOTE** There are options to produce a python rendered network and dendrogram. These should be disabled as they take a massive amount of time to produce. The network output can be visualised in cytoscape.

### Input
MASH distance file: 2Jul2023.refseq.d0.3.k15.s25000.tsv  

**User parameters:**  
Input file path: location and name of input mash distance file   
Sample size: 0 for everything, else limit to (n) sequence records  
Distance threshold: maximum distance to use for network edges    
P-value threshold: maximum p-value to use for network edges  
Hashes threshold: minimum hashes ratio to use for network edges  


### Output
Output folders are created automatically, named by date and time of run


passed_to_networkx.tsv: Checkpoint dataframe   
community_counts_table.csv: Number of member genomes for each community  
community_members_table.csv: Comma-separated list of accession numbers for each community  
network.xml: graphxml file of networkx object following filtering - can be visualised in cytoscape   
full_louvain_partition.json: json output of louvain paritioning

## 4. Downstream analysis (not coded)
> 1. Extract accession numbers for a random community
> 2. Download records from GenBank as multi-fasta file for those accessions (E-utilities API)  
> 3. Process fasta record names so only accession numbers remain: \.[0-9].*  
> 4. Use [VIRIDIC](https://rhea.icbm.uni-oldenburg.de/viridic/) to assess whether the communities defined fall within or outside of current genus demarcation criteria.