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