# CIDGOH genomic tutorials

## Objective

- Introduce new team members with limited experience in microbial genomics to the common approach to analyze bacterial genomes using an executable jupyter notebook tool
- Create a reference repository for microbial genomics training that can be updated with advanced pipelines and novel tools

## Datasets

Downloaded data from NCBI for project. Only 10 samples in paired end Illumina reads. Primary study of an outbreak of *P. aeruginosa* isolates in Switzerland [PMID:34412676](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC8376114/). 

## Environment and tools

The structure of the directories is below. For analysis, reference genomes and datasets are included in `tools/reference_data` and analysis tools in container `tools/sing_imgs`

    /home/jupyter-mdprieto
    |-- scripts
    |-- tools
    |   |-- reference_data
    |   `-- sing_imgs
    `-- tutorials
        |-- annotation
        |-- assembly_checkm
        |-- assembly_quast
        |-- contigs
        |-- raw_reads
        |-- results_qc
        `-- trimmed_reads

A virtual environment in python allows execution of the notebook in a bash kernel. Can be setup with the following commands, it creates a python and bash kernel. 

```sh
# Create a virtual environment to run the tutorials
cd
python3 -m venv venv_tutorials
source venv_tutorials/bin/activate
conda deactivate
pip3 install ipython ipykernel bash_kernel
python3 -m ipykernel install --user --name=tutorials_python
python -m bash_kernel.install
deactivate
```

Server has module manager similar to cedar, load it beforehand to use
> source /cvmfs/soft.computecanada.ca/config/profile/bash.sh

Annotation with Bakta requires reference database, for simplicity I use a reduced version

    singularity exec -B /scratch,/project,/etc "$BAKTA_IMG" bakta_db download \
        --output /project/60005/cidgoh_share/database \
        --type light

# Notebook

## 20230206 - QC and Assembly tutorials

- Annotated all pipeline for QC of reads
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

```sh
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

```sh
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

## 20230307 - Finished assemblyQC tutorial

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

## 20230308 - Started annotation tutorial

- I try to setup the bakta database for later analysis in eagle. 
    - The db in object_storage is not writable and cannot be unpacked. Also, it seems to be a previous outdated version.  
    - Bakta checks for certificates in the /etc directory, I have to mount it to execute successfully through singularity
    - Downloaded a new light version database. No need to be comprehensive for the tutorials.
    - Jobs cannot access object_storage, object_database and such. Have to use different directories
    - In seagull, I bring the downloaded database for Bakta and try to run the program to annotate a few genomes in the tutorial. 

### Annotation with bakta in eagle

```sh

 singularity exec -B /scratch,/project,/etc "$BAKTA_IMG" bakta_db download \
    --output /project/60005/cidgoh_share/database \
    --type light

#!/bin/bash                                 
#SBATCH --mem-per-cpu=6G                   
#SBATCH --time=02:30:00                     
#SBATCH --cpus-per-task=8                
#SBATCH --job-name="bakta_annotation_tutorials"     
#SBATCH --chdir=/scratch/mdprieto/         
#SBATCH --output=bakta_tutorials.out 

###################################################
module load singularity
BAKTA_IMG="/project/60005/cidgoh_share/singularity_imgs/bakta_1.7.sif"
BAKTA_DB="/project/60005/cidgoh_share/database/db-light"

# run bakta
for i in $(ls /project/60005/mdprieto/tutorials/*contigs.fa)
    do
    singularity exec -B /scratch,/project,/etc "$BAKTA_IMG" bakta \
    --db "$BAKTA_DB" \
    --output /scratch/mdprieto/tutorials_2023/results_bakta \
    $i \
    --genus pseudomonas
    done

```

### Transfer data to seagull

```sh
cp -r object_storage_eagle/mdprieto/tutorials/db-light/ /home/jupyter-mdprieto/tools

# commands to run in seagull
singularity exec -B /etc "$BAKTA_IMG" bakta \
    --db "$BAKTA_DB" \
    /home/jupyter-mdprieto/tutorials/contigs/ERR10479518_contigs.fa \
    --output /home/jupyter-mdprieto/tutorials/annotation/ \
    --genus Pseudomonas \
    --prefix $(echo "ERR10479518_contigs.fa" | grep -Eo "ERR[0-9]+")
```

## 20230320 - Finished bakta annotation 

- Configured VScode to run jupyter notebooks in seagull remotely
    + Re-install bash kernel for python
    + Working for python interpreter with virtual environment. Could not make it work with conda
- To produce a tree of the directory organization for the project use:
> tree -dL 2 -I 'CIDG*|venv*|mnt*|globus*' -P 'tutorials' $HOME

```sh
# create virtual environment at $HOME
cd
python3 -m venv venv_tutorials
source venv_tutorials/bin/activate
conda deactivate
pip3 install ipython ipykernel
python3 -m ipykernel install --user --name=tutorials_python
pip install bash_kernel 
python -m bash_kernel.install
deactivate

# conda 
mamba create -n tutorials_env
mamba activate tutorials_env
mamba install ipykernel jupyter
python3 -m ipykernel install --user --name=tutorials_env

```

## 20230321 - Finished all project

- A core genome MLST seems to be more complicated to setup. So I will use a core genome SNP approach with **snippy to produce a phylogenetic tree
- Snippy-multi requires a tab separated file with filenames and paths. Can be contigs or reads. 
    + Can be created easily by saving every input type (filenames, read_1, read_2, contigs) to a file
    + Then, files can be joined horizontally using join
- Easier to run a forloop as snippy-multi does not produce ready to use commands when executing the tool from a container

```sh
# produce snippy input
cd /PATH/CONTIGS_DIR
readlink -f *contigs.fa > path.txt
ls * contigs.fa > filenames.txt
paste filenames.txt path.txt > snippy-input.tab
rm filenames.txt path.txt



for contig in $(ls $HOME/tutorials/contigs/*contigs.fa | head -n 2)
do
    # define isolate name
    meta=$(basename "$contig" '_contigs.fa')

    # run snippy for each isolate
    singularity exec "$SNIPPY_IMG" snippy \
    --outdir "$HOME/tutorials/results_snippy/$meta"  `# save in a subdirectory`\
    --ctgs "$contig" \
    --ref $REF_GENOME \
    --cpus 8 \
    --force \
    --cleanup \
    --quiet

    # produce core genome analysis in results_snippy folder
    cd $HOME/tutorials/results_snippy/
    singularity exec "$SNIPPY_IMG" snippy-core --ref $REF_GENOME \
    /home/jupyter-mdprieto/tutorials/results_snippy/ERR*
done

```
## 20230416 - Cleanup of previous sections

- For `raw_reads_processing`
    - Made sure that the blurbs had a nice flow and content 
    - Organized the structure of the results and verified that commands are running with kernel created in virtual environment
- For `genome_assembly`
    - Updated text in the blurbs
    - Added additional explanations to terms and references to external URLs
    - Printed tree structure of directories after every major step
    - Verified how the commands were running

## 20230502 - Final clean up of tutorials

- Finished review of all tutorials, including updated to blurbs and additional references for further reading. 
- Uploaded everything to `shared_notebooks` (`/share/jupyterNote`)  directory in **Seagull** 
- Once everything is deployed, I will request feedback
    