# Metagenomics preliminary analysis for Cystic Fibrosis Canada seed grant

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
# Analytical workflow

## Summary of analysis pipeline for diversity measures

1. Download and prepare datasets of lung shotgun metagenomics for NCFB and CF
2. Perform basic QC of raw reads for all datasets (nf-mag)
3. Calculate taxonomic abundances in all samples using kraken2 and bracken
4. Using kraken tools calculate alpha and betadiversity, after classifying samples with predominance of _Pseudomonas_ spp. 


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

## 20230322 - Exploring nfseqqc pipeline

- Decided to download data for CF and NCFB into separate directories for downstream analysis later on
- Evaluating CIDGOH pipeline for reads qc
- Working, still have to review pertinence of report
- How to calculate diversity from reads, what is the necessary input?
- **Pipeline does not perform dehosting before calculation of **


```sh
OUTPUT_QC="/project/60005/mdprieto/cf_seed_2023/results/ngs_qc"
SEQQC_NF="/scratch/mdprieto/nf-seqqc/main.nf"
TRIAL_SHEET="/project/60005/mdprieto/cf_seed_2023/raw_data/ncfb_data/samplesheet/trial_ncfb_samplesheet.csv"

NFCORE_IMG="/project/cidgoh-object-storage/images/nf-core_2.7.2.sif"

export NXF_SINGULARITY_CACHEDIR="/project/cidgoh-object-storage/images"

nextflow run $SEQQC_NF \
    -profile singularity \
    -resume \
    -c eagle \
    --outdir $OUTPUT_QC \
    --input $TRIAL_SHEET \
    --skip_assembly \
    --skip_assembly_qc \
    --skip_confindr \
    --skip_subsampling \
    --max_memory 50GB \
    --max_cpus 8 \
    --fasta /project/cidgoh-object-storage/database/test_fasta/GCF_009858895.2_ASM985889v3_genomic.200409.fna.gz 
```
## 20230326 - Running metagenome analysis on Cedar or Eagle

### Defined pipeline for analysis


### nf-core mag pipeline

- Input needs to be on quotes when specifying path 'PATH/FASTQ_FILES'
- Minimum cpu requires for bowtie while dehosting is 10 by default


```sh
# install dependencies for mag pipeline
singularity exec -B /scratch,/etc $NFCORE_IMG nf-core download mag -r 2.3.0 --singularity-cache-only --container singularity --force

NCFB_FASTQ="/project/60005/mdprieto/cf_seed_2023/raw_data/ncfb_data/fastq"
KRAKEN2_DB="/home/mdprieto/object_database/kraken2/k2_standard_20221209.tar.gz"

# trial with 10 samples
nextflow run nf-core/mag -r 2.3.0 \
    -profile singularity \
    -resume \
    --input "$NCFB_FASTQ/SRX900293*_{1,2}.fastq.gz" \
    --outdir /scratch/mdprieto/cf_seed_results/nf-mag \
    --host_genome GRCh38 \
    --kraken2_db $KRAKEN2_DB \
    --skip_prodigal \
    --skip_prokka \
    --skip_krona \
    --skip_spades \
    --skip_megahit \
    --skip_binning \
    --max_memory 50GB \
    --max_cpus 8

# all ncfb data
nextflow run nf-core/mag -r 2.3.0 \
    -profile singularity \
    -resume \
    --input "$NCFB_FASTQ/SRX*_{1,2}.fastq.gz" \
    --outdir /scratch/mdprieto/cf_seed_results/nf-mag \
    --host_genome GRCh38 \
    --kraken2_db $KRAKEN2_DB \
    --skip_prodigal \
    --skip_prokka \
    --skip_krona \
    --skip_spades \
    --skip_megahit \
    --skip_binning \
    --max_memory 77GB \
    --max_cpus 8
    
```

## 20230327 - nf-core tax profiler

- Requires a pretty specific samplesheet with the following columns: 
`sample,run_name_accession,platform(ILLUMINA),fastq_1,fastq_2,fasta`
- I create it for now manually, and if necessary will build a python script to make it automatically ([Useful base script](https://github.com/nf-core/rnaseq/blob/master/bin/fastq_dir_to_samplesheet.py))

```sh
KRAKEN2_DB="/home/mdprieto/object_database/kraken2/k2_standard_20221209.tar.gz"

# clean and obtain path files
readlink -f raw_data/cf_data/fastq/* | \
    grep "_1.fastq.gz" | \
    sort

# get sample_run accession cleaned
readlink -f raw_data/cf_data/fastq/* | \
    grep "_1.fastq.gz" `# sort here to maintain order with paths` | \
    sort | \
    grep -Eo "SRR[0-9]*" 

```

- To run, most options are opt-in. For now, I will explore only a few samples and try to produce results exclusively with Kraken2

```sh
KRAKEN2_DB="/home/mdprieto/object_database/kraken2/k2_standard_20221209.tar.gz"
SAMPLE_SHEET="/project/60005/mdprieto/cf_seed_2023/scripts/samplesheet_taxprof_ncfb.csv"
DB_CSV="/project/60005/mdprieto/cf_seed_2023/scripts/databases_taxprof.csv"

# test run    
nextflow run nf-core/taxprofiler -r 1.0.0 \
    -profile singularity \
    -resume \
    --input /project/60005/mdprieto/cf_seed_2023/scripts/trial_samplesheet.csv \
    --databases $DB_CSV \
    --outdir /scratch/mdprieto/cf_seed_results/taxprof \
    --perform_shortread_qc \
    --perform_shortread_hostremoval \
    --hostremoval_reference /project/cidgoh-object-storage/database/reference_genomes/human/GRCh38.p14/GCF_000001405.40/GCF_000001405.40_GRCh38.p14_genomic.fna \
    --shortread_hostremoval_index /project/cidgoh-object-storage/database/bowtie_GRCh38 \
    --run_bracken \
    --run_kraken2 \
    --max_cpus 8 \
    --max_memory 60GB

    
```

- Multiple errors while working with bracken and kraken databases, they seem to be the same name and path, so I will duplicat them. It is a pretty resource intensive process as the file is more than 48 GB. 

