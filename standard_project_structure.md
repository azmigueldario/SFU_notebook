# How to organize bioinformatic projects

Following the recommendations of several authors to improve reproducibility, 
I will document the common standards to use in any kind of data science project, 
so I can quickly reference them in the future and facilitate the understanding 
of my peers and collaborators. 

## Sources

- A quick guide to organizing computational biology projects by William Stafford Noble, 2009 [link](https://journals.plos.org/ploscompbiol/article?id=10.1371/journal.pcbi.1000424)
- Jhon Hopkins Intro to reproducibility in Cancer genomics. October 2022 [Link](https://jhudatascience.org/Reproducibility_in_Cancer_Informatics/organizing-your-project.html)
- Introduction to Project Management by Andrew Severin, 2023/ [link](https://bioinformaticsworkbook.org/projectManagement/Intro_projectManagement.html#gsc.tab=0)
- Project organization section of the Introduction to command line for genomics by the Carpentries [link] (https://bioinformatics-core-shared-training.github.io/shell-genomics/07-organization/index.html)

# Project structure

## Every project is a GitHub repository

The **raw_data** will probably be held in a separate subfolder to facilitate data sharing and to avoid synchronization issues with Git. 

## Use four different sub-folders for organization

1. `project/notebook` contains documentation about the project
2. `project/scripts` will contain all necessary steps to reproduce the analysis, scripts and custom workflows
3. `project/processed_data` will contain intermediate results or cleaned datasets, **not the raw data**
4. `project/output` contains processed results, can be organized by subtype or date

```sh
    project
    ├── notebook
    │   └── notebook.md
    ├── output
    │   ├── cgMLST.csv
    │   ├── run-final.log
    │   └── snp-distances.csv
    ├── processed_data
    │   ├── metadata_samples.csv
    │   └── temporary_results.csv
    └── scripts
        ├── SNV_call.sh
        ├── assembly.sh
        ├── assembly_qc.sh
        ├── fastqc_raw.sh
        └── phylogenetic_tree.sh
```

## Document project advances and work in a markdown notebook

The notebook(s) will be located in a subfolder called notebook and will be created in markdown format. 

The structure of notes must include:
- Dates where every work was performed
- Updated on advances
- Notes about versions and troubleshooting for future references

## Naming convention 

When feasible, name files using an easy to understand name with underscores. Dates can be added if necessary using the ISO format (YYYYMMDD):

    20230909_introduction_machine_learning.pdf
    20210908_metadata_klebsiella.csv
    fetch_ngs.sh
    main.nf
    fastqc_raw_reads.sh