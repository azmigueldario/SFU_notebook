# CIDGOH genomic tutorials

## 20230206 - QC and Assembly tutorials

- Annotated all pipeline for QC of reads
- Downloaded data from NCBI for project. Only 10 samples in paired end Illumina reads. Primary study of *P. aeruginosa* isolates in Switzerland [PMID:34412676](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC8376114/). 
- Jun installed `modules` functionality in test server, yet I cannot run it or execute the installed commands from jupyter notebook (like seqkit after `module load StdEnv/2020 seqkit/2.3.1`)
- Will use `BBtools` script to conduct trimming of these reads 

## 20230208 - Updated modules functionality

- According to Jun suggestion, I source the PATH to the modules command and it is now functional in Jupyter notebooks `source /cvmfs/soft.computecanada.ca/config/profile/bash.sh`
-  **fastqc** does not need to be looped, better to run with all files using a wildcard `/PATH/*fastq.gz` and specify several threads in parallel `-t 8`
-  **bbmap** can do trimming of adaptors and quality trimming in same run. 
   -  options for command must not have spaces while writing (`option=value`)


## 20230209 - Finished sequencing reads tutorials

- Organized results by project (project_name/results_subdirectories)
- Ran and completed trimming of reads in server. 
	- Output creates new directory, must specify only filename and not complete PATH (`clean/filename`)
- Summarized with multiQC
- Shared tutorial with CIDGOH group for hackaton

```
# env variables
adapters_file='/cvmfs/soft.computecanada.ca/easybuild/software/2020/Core/bbmap/38.86/resources/adapters.fa'
OUTPUT_TRIM="/home/jupyter-mdprieto/tutorials/trimmed_reads"
INPUT_DIR="/home/jupyter-mdprieto/tutorials/raw_reads"

# for loop BBtools
for i in $(ls $INPUT_DIR/*_R1*)
do
R1=$(basename $i)
R2=$(echo $R1 | sed 's/_R1/_R2/')
bbduk.sh \
    in1=$INPUT_DIR/$R1 in2=$INPUT_DIR/$R2 \
    out1=$OUTPUT_TRIM/$R1 out2=$OUTPUT_TRIM/$R2 \
    ref=$adapters_file \
    k=23 \
    trimq=6 \
    tpe \
    tbo \
    threads=9
done
```

## 20230228 - Finish assembly tutorial 

Only 2 samples will be processed in the tutorial to minimize the run time. The remaining ones will be already processed. I add the `--force` option to **Shovill** so it will override any existent output.

- All assemblies are already available in seagull after running the following command.
    - For future uses, it took a long time in Seagull. I can run a job in eagle, which is faster, and send the resulting data later. 

```
for read1 in $(ls "$TRIMMED_READS"/*R1.fastq.gz)
do

    # substitute R1 for R2 in variable
    read2=${read1/_R1/_R2}

    # extract isolate id
    prefix_isolate=$(basename $read1 _R1.fastq.gz)
    echo "Started processing $prefix_isolate"
    
    # execute shovill for each sample
    singularity exec $HOME/tools/shovill_1.1.sif shovill --R1 $read1 --R2 $read2 \
    --outdir $OUTPUT_DIR_TUTORIAL \
    --gsize 7M \
    --force \
    --ram 140 
    
    # name resulting file with isolate id
    mv $OUTPUT_DIR_TUTORIAL/contigs.fa $OUTPUT_DIR_TUTORIAL/$prefix_isolate\_contigs.fa
    
    echo "Finished assembly of sample $prefix_isolate"
    
done
```

- I will use the P. aeruginosa PAO1 assembly as reference for all quality control (GCF_000006765.1_ASM676v1) together with its corresponding annotation file (GCF_000006765.1_ASM676v1_genomic.gff.gz)
    - These files are stores in the tools directory of my seagull account 
    - Will create a parser for the output of Quast if I have time
- CheckM is not available as a module. So, instead of creating a virtual environment, I will download a singularity container.\
> singularity pull checkm_1.2.2.sif https://depot.galaxyproject.org/singularity/checkm-genome%3A1.2.2--pyhdfd78af_1

## 20230302 - Finishing CheckM section

- Run the complete pipeline in the eagle cluster and then move results to optimize my time

## 20230307 - Finished assembly + QC tutorial

- It was not necessary to run **CheckM** in another cluster. For 10 samples it took around 15 min to analyze the presence of marker sets and less than 1 minute to produce the output
- Tested the code in seagull and updated git repo accordingly

```sh
# define checkm PATH
CHECKM_IMG="/project/60005/cidgoh_share/singularity_imgs/checkm_1.2.2.sif"

# create reference markers for pseudomonas in scratch directory
singularity exec -B /scratch,/project "$CHECKM_IMG" checkm \
    taxon_set species 'Pseudomonas aeruginosa' /scratch/mdprieto/tutorials_2023/pseudomonas.ms

# analyze presence of markers 
singularity exec "$IMG_CHECKM" checkm analyze \
    /home/jupyter-mdprieto/tools/pseudomonas.ms         `#file with checkm marker set for assemblies` \
    "$CONTIGS_DIR"                                      `#dir with assemblies in fasta format` \
    "$ASSEMBLY_CHECKM"                                  `#output directory` \
    -x fa                                               `#extension of assemblies` \
    -t 8   

# produce table of contaminations
singularity exec "$IMG_CHECKM" checkm qa \
        /home/jupyter-mdprieto/tools/pseudomonas.ms     `#file with checkm marker set for assemblies` \
        "$ASSEMBLY_CHECKM"                              `#output directory` \
        --file "$ASSEMBLY_CHECKM/checkm_output.tsv" \
        --tab_table                                     `# print tabular output` \
        --threads 8                                     `# number of simultaneous threads for process` \
        --out_format 1                                  `# format of output 1 = summary, 2 = extended`                        
```

## 20230308 - Finished assembly + QC tutorial

- I try to setup the bakta database for later analysis in eagle. 
    - Downloaded a new one, light version as it is just for the tutorials. I do not need it to be comprehensive
    
- The db in object_storage is not writable and cannot be unpacked in there. Also, it seems to be a previous outdated version. 

```sh
# eagle

BAKTA_IMG="/project/cidgoh-object-storage/images/bakta_1.7.sif"
BAKTA_DB="/project/cidgoh-object-storage/database/bakta/db.tar.gz"

# mount etc so the API can access certificates
singularity exec -B /scratch,/project,/etc "$BAKTA_IMG" bakta_db list

# download new database - run in job to optimize time
singularity exec -B /scratch,/project,/etc "$BAKTA_IMG" bakta_db download \
    --output /project/cidgoh-object-storage/eagle/mdprieto/tutorials \
    --type light

# update database
singularity exec -B /scratch,/project,/etc "$BAKTA_IMG" bakta_db update \
    --db "$BAKTA_DB" \
    --tmp-dir /scratch/mdprieto/tmp_bakta

# run bakta
singularity exec -B /scratch,/project,/etc "$BAKTA_IMG" bakta \
    --db "$BAKTA_DB" \
    ERR10479510_contigs.fa
```

In seagull

```sh
# seagull
BAKTA_IMG="/mnt/cidgoh-object-storage/images/bakta_1.7.sif"
export BAKTA_DB="/mnt/cidgoh-object-storage/database/bakta/"

singularity exec -B /etc "$BAKTA_IMG" bakta_db list

singularity exec -B /etc "$BAKTA_IMG" bakta 

```