# Metagenomics preliminary analysis for seed grant

## Objective

- To define if predominance of _Pseudomonas_ spp. affects the alpha and beta diversity of the lung microbiome in bronchiectasis

## Datasets

The primary dataset is a publicly available collection of lung 16S amplicon sequencing available at NCBI/ENA. The identifier of the project is **PRJNA605315** and the primary URL for NCBI is https://www.ncbi.nlm.nih.gov/bioproject/PRJNA605315/. An associated paper reporting the findings of this project is located at 

The dataset of the project https://www.ncbi.nlm.nih.gov/bioproject/?term=PRJNA590225 seems to contain actual metagenomic information and not just amplicon. The primary article can be found here: [Mac Aogáin et al. 2021](https://www.nature.com/articles/s41591-021-01289-7).

No other papers in the ENA or NCBI seem to contain metagenomic data for bronchiectasis by March, 2023.

Suggestion by Amy: https://www.bv-brc.org/app/MetagenomicReadMapping. Helps to maps virulence fo

## Environment in Eagle

Project directory 
> /project/60005/mdprieto/cf_seed_2023_

Downloaded sra-tools singularity image to obtain SRA sequences for lung metagenomes of bronchiectasis patients from the link 'https://depot.galaxyproject.org/singularity/sra-tools%3A3.0.0--pl5321hd0d85c6_1'.
> /project/60005/cidgoh_share/singularity_imgs/sra-tools_3.0.0.sif

# Notebook of advances

## 20230309

- Downloaded singularity image for sra toolkit
- Configured tool using `vdb-config -i` and setting a temporary directory to prefetch data in the `/scratch` directory
- 


```sh
SRA_IMG="/project/60005/cidgoh_share/singularity_imgs/sra-tools_3.0.0.sif"
```