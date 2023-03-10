# Metagenomics preliminary analysis for seed grant

## Objective

- To define if predominance of _Pseudomonas_ spp. affects the alpha and beta diversity of the lung microbiome in bronchiectasis


## Environment in Eagle

Project directory 
> /project/60005/mdprieto/cf_seed_2023_

Downloaded sra-tools singularity image to obtain SRA sequences for lung metagenomes of bronchiectasis patients from [https://depot.galaxyproject.org/singularity/sra-tools%3A3.0.0--pl5321hd0d85c6_1](https://depot.galaxyproject.org/singularity/sra-tools%3A3.0.0--pl5321hd0d85c6_1)
> /project/60005/cidgoh_share/singularity_imgs/sra-tools_3.0.0.sif

# Notebook of advances

## 20230309

- Downloaded singularity image for sra toolkit
- Configured tool using `vdb-config -i` and setting a temporary directory to prefetch data in the `/scratch` directory



```sh
SRA_IMG="/project/60005/cidgoh_share/singularity_imgs/sra-tools_3.0.0.sif"
```