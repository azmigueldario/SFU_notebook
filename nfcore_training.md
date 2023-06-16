# nf-core asynchronous training for 2023

## Objectives per session

1. Introduction to nextflow developing pipeline
2. Introduction to nf-core for users and developers
3. In depth learning of processes, operators, groovy and modularization
4. Configuration, deployment, cache/resume, and nextflow tutorial

# Session I

- Nextflow has implicit parallelism, allows simultaneous execution of processes that are usually staged in a linear manner (like fastq)
- Reproducibility is guaranteed because the pipeline is scalable to use different container/software management tools and executors in cloud or HPC environment
- nf-core is an initiative to curate and collect pipelines in nextflow language, mostly focused on genomics for now
    - modules or processes are made readily available in nf-core (more than 700 available)
    - based on community development, a common template, and an online collaborative environment


## Hands-on session 

Many name operations only work when called inside the workflow declaration or after using `file()` to load an object
```groovy
workflow {
    my_file = file(object_file)
    my_file.getName()
}

//----- process //------
script:
"""
command -reads $reads /
        -genome_dir ${reads.getParent()}
"""
```

