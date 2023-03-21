# Metagenomics preliminary analysis for seed grant

## Objective

- To define if predominance of _Pseudomonas_ spp. affects the alpha and beta diversity of the lung microbiome in bronchiectasis

### Notes preparatory meeting with Amy

Applying to a cystic fibrosis(CF) seed grant with the same proposal for the CIHR to get preliminary data and working funds. 
- CIHR wants to make a comparison with CF. Christina will get 20 CF sputum and 20 non-CF bronchiectasis (NCFB) sputum (each group contains 10 with _Pseudomonas_ spp. and 10 without it)  
- What happens in the microbiome. There is a decrease in diversity as CF individuals age that is faster when _Pseudomonas_ spp. is present.  
- **Questions:** When you have _Pseudomonas_ infection you have decreased diversity. Is this consistent between NCFB and CF? Is this decrease correlated with virulence potential, like from PAGs?

## Datasets

### NCFB data

The dataset of the project [PRJNA590225](https://www.ncbi.nlm.nih.gov/bioproject/?term=PRJNA590225) seems to contain actual metagenomic information and not just amplicon. The primary article can be found here: [Mac Aogáin et al. 2021](https://www.nature.com/articles/s41591-021-01289-7).  

- Metagenomic data was processed from sputum  
- Sample size of 166 participants in a single time-point - includes a few blank samples (total n = 176) 
- Recruited in Singapore, Malaysia, UK and Italy  

No other papers in the ENA or NCBI seem to contain metagenomic data for bronchiectasis by March, 2023.

### CF data

Similar to NCFB datasets, I checked the **BioProject** and **SRA** databases from **NCBI** in March 17, 2023 for studies reporting metagenomic data from shotgun sequencing in sputum (to make it consistent with NCFB dataset). 

Using the following search terms we retrieved 341 sample runs and manually verified which were amenable for analysis. 

> ((cystic fibrosis lung) AND "metagenomic"[Source] ) NOT ("amplicon"[Strategy] OR "rna seq"[Strategy] OR "abi solid"[Platform] OR "bgiseq"[Platform] OR "capillary"[Platform] OR "helicos"[Platform] OR "ls454"[Platform] OR "pcr"[Selection] ) 

The project [PRJNA516870](https://www.ncbi.nlm.nih.gov/bioproject/PRJNA516870) was selected for analysis. Further data is available in the primary article by [Bacci et al. 2020](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC7409339/). 
- Samples processed from spontaneous sputum  
- Recruited in three Italian CF centers
- Sample size of 78, represents repetitive data from 26 patients

**Note:** The study PRJEB52317 has a larger sample of CF patients, but the available data seems to lack one of the paired-end reads. 

## Environment in Eagle

```sh
# project directory and github repository (data files are not tracked)
/project/60005/mdprieto/cf_seed_2023

# SRAtools singularity from the link https://depot.galaxyproject.org/singularity/sra-tools%3A3.0.0--pl5321hd0d85c6_1
SRA_IMG="/project/60005/cidgoh_share/singularity_imgs/sra-tools_3.0.0.sif"

# singularity cache dir, set up in `.bashrc` options
NXF_SINGULARITY_CACHEDIR="/project/60005/cidgoh_share/singularity_imgs/"

# directory with scripts for project
SCRIPTS="/project/60005/mdprieto/cf_seed_2023/scripts" 
```
# Notebook of advances

## 20230309

- Downloaded singularity image for sra toolkit
- Configured tool using `vdb-config -i` and setting a temporary directory to prefetch data in the `/scratch` directory
- Suggestion by Amy: https://www.bv-brc.org/app/MetagenomicReadMapping. Helps to map virulence in metagenomes

```sh
SRA_IMG="/project/60005/cidgoh_share/singularity_imgs/sra-tools_3.0.0.sif"
```

## 20230317

- Searched and selected a proper CF dataset for lung shotgun metagenomes
- Cleaned design of laboratory notebook
- Prepared accession list and metadata files in Github repo
- Created, troubleshooted and ran script to download all raw data files
    + fasterq-dump requires accession as parameter
    + to run multiples accessions by fasterq-sump use `cat accessions | xargs fasterq-dump`

## 20230320
- **SRA tools** is having issues while downloading and it produces a single fastq file instead of two (R1 and R2)
- Unfortunately, it seems like the CF project PRJEB52317 has only a read uploaded in the repositories and failed to make **reads_2** available

#### Downloading data effectively


- Tried using **nf-core** modules to retrieve data
    + Installed using (`pip install nf-core`)
    + Prepared containers and workflow by calling `nf-core download <pipeline_name>` which interactively downloads necessary dependencies and pipeline
    + To keep a centralized container repository and not download in pipeline folder I add the flag `--singularity-cache-only`
    + Accession list must be in `.csv` format  
    + Eliminate empty spaces at the end of accession list, may produce error as input of pipeline


- Download of 10 samples took less than 10 minutes, so I will ask for 3 hours of wall time for the complete dataset (n= 200)
