# CIDGOH genomic tutorials

## 20230206 - QC and Assembly tutorials

- Annotated all pipeline for QC of reads
- Downloaded data from NCBI for project. Only 10 samples in paired end Illumina reads. Primary study of *P. aeruginosa* isolates in Switzerland [PMID:34412676](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC8376114/). 
- Jun installed `modules` functionality in test server, yet I cannot run it or execute the installed commands from jupyter notebook (like seqkit after `module load StdEnv/2020 seqkit/2.3.1`)
- Will use `BBtools` script to conduct trimming of these reads



```
for R1 in $(ls $INPUT_DIR/*_R1*)
	do
	R1=$(basename $i)
	R2=$(echo $R1 | sed 's/_R1/_R2/')
	echo $R1 $R2
	echo bbduk.sh /
		in1=$R1 in2=$R2 
	done
		out1=clean_$input_.fq out2=clean.fq \
		ref=adapters.fa \
		ktrim=r \
		k=23 \
		mink=11 \
		hdist=1 tpe tbo
	done
```
