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
- CheckM is not available as a module. So, instead of creating a virtual environment, I will download a singularity container. 

```py
import  numpy as np
import argparse
import pandas as pd

def parse_quast():
    parser = argparse.ArgumentParser(description='Summarize Quast results')
    parser.add_argument('-i', '--input', type=str, default=None,
                        help='report.tsv file from quast')
    parser.add_argument('-o', '--output', type=str, default=None,
                        help='output summary file also in tsv')
    return parser.parse_args()

if __name__ == '__main__':
    args = parse_args()
    quast_df = pd.read_csv(args.input, sep='\t')


if __name__ == '__main__':
    args = parse_args()
    df = pd.read_csv(args.input, sep='\t')
    df[['isolate', 'gene']] = df['Contig'].str.split('-', expand=True)
    res = df.pivot(index='isolate', columns='Locus', values='Allele')
    res.to_csv(args.output+".txt", sep='\t', header=False)
```