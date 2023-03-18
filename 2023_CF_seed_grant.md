# Metagenomics preliminary analysis for seed grant

## Objective

- To define if predominance of _Pseudomonas_ spp. affects the alpha and beta diversity of the lung microbiome in bronchiectasis

### NCFB data

The dataset of the project [PRJNA590225](https://www.ncbi.nlm.nih.gov/bioproject/?term=PRJNA590225) seems to contain actual metagenomic information and not just amplicon. The primary article can be found here: [Mac Aogáin et al. 2021](https://www.nature.com/articles/s41591-021-01289-7).  

- Metagenomic data was processed from sputum  
- Sample size of 166 participants in a single time-point - includes a few blank samples (total n = 176) 
- Recruited in Singapore, Malaysia, UK and Italy  

No other papers in the ENA or NCBI seem to contain metagenomic data for bronchiectasis by March, 2023.

### Cystic fibrosis (CF) data

Similar to NCFB datasets, I checked the **BioProject** and **SRA** databases from **NCBI** in March 17, 2023 for studies reporting metagenomic data from shotgun sequencing in sputum (to make it consistent with NCFB dataset). 

Using the following search terms we retrieved 341 sample runs and manually verified which were amenable for analysis. 

> ((cystic fibrosis lung) AND "metagenomic"[Source] ) NOT ("amplicon"[Strategy] OR "rna seq"[Strategy] OR "abi solid"[Platform] OR "bgiseq"[Platform] OR "capillary"[Platform] OR "helicos"[Platform] OR "ls454"[Platform] OR "pcr"[Selection] ) 

A subset of samples (n = 77 sputum specimens) from the [PRJEB52317](https://www.ncbi.nlm.nih.gov/bioproject/PRJEB52317) project were selected for analysis. Further data is available in the primary article by [Pienkowska et al. 2023](https://journals.asm.org/doi/10.1128/spectrum.03633-22). 
- Samples processed from induced sputum  
- Recruited in Hannover, Germany  
- Sample size of 77, more samples from oral and nasal cavity available

## Environment in Eagle

Project directory and github repository (data files are not tracked)
> /project/60005/mdprieto/cf_seed_2023

Downloaded sra-tools singularity image to obtain SRA sequences for lung metagenomes of bronchiectasis patients from the link 'https://depot.galaxyproject.org/singularity/sra-tools%3A3.0.0--pl5321hd0d85c6_1'.
> /project/60005/cidgoh_share/singularity_imgs/sra-tools_3.0.0.sif

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