## 20230725 - Testing datasets

```sh
# illumina
/mnt/cidgoh-object-storage/hackathon/seqqc/isolate_wgs/illumina/test/SRR*

# nanopore
/mnt/cidgoh-object-storage/hackathon/seqqc/isolate_wgs/nanopore/sra_split/**

# interactive node with decent cpus
salloc --time=1:30:00 --ntasks=1 --cpus-per-task=8  --mem-per-cpu=6G  
```

# Trying BUSCO with nanopore data 

## set up samplesheet

```sh
# add headers
echo "sample,fastq_dir" > input_samplesheet_hackathon.csv

for fastq_dir in $(realpath /mnt/cidgoh-object-storage/hackathon/seqqc/isolate_wgs/nanopore/sra/*);
    do 
    sample_id=$(echo $fastq_dir | xargs -n 1 basename)
    echo $sample_id,$fastq_dir/ >> input_samplesheet_hackathon.csv
    done

```

## run with trial data

Busco issues:
    - `auto-lineage` options cycles through different domains downloading each lineage dataset until it fails
    - specifying a specific `lineage` uses only that one and not every taxa downstream from it
    - We are trying to download the databases to minimize run time and avoid failures due to downloading inside the pipeline

```sh
nextflow run nf-seqqc/main.nf \
    --input input_samplesheet_hackathon.csv \
    -profile singularity \
    --outdir /scratch/mdprieto/hackathon_results \
    -resume \
    --max_memory 40.GB \
    --max_cpus 8 \
    --skip_taxonomy_qc \
    --skip_subsampling
```