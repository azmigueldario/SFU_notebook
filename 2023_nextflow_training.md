# Introduction

This is not a project per se but more of a documentation of my experiences while learning **Nextflow**. In mid 2022 when I tried to dip my toes in **Nextflow** I had a hard time understanding the syntax and only after getting more familiar with scripting in **Python** I feel like I am up to the challenge. 

Thus, I will try to list resources to review when I need to re-visit any topic and also provide quick snippets and tips for my future scripting. Finally, as in all my other notebooks, I will register my advances with dates. 

## Useful resources

- Yearly aggregated list of resources to learn Nextflow - [2023 link](https://www.nextflow.io/blog/2023/learn-nextflow-in-2023.html)
- Introduction to Nextflow form Seqera Labs, not updated to use DSL 2 (https://github.com/seqeralabs/nextflow-tutorial)


# Notebook of activities

## 20230301

- When you need to run a process after all previous ones have been completed use `.collect()
- For local testing, I will use the **nextflow_test** conda environment

## 20230314

- The input of a process is the expected input when running the script, not precisely the channel that is going to be used but the format of it
    + val (meta) means that it expects a value
    + path (reads) needs file(s) from a particular PATH
- Pipeline parameters are specified as `params.parameter_name` and can be modified from the default in the pipeline by specifying it in the call as `--parameter_name`
