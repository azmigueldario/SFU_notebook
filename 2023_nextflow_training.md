# Introduction

This is not a project per se but more of a documentation of my experiences while learning **Nextflow**. In mid 2022 when I tried to dip my toes in **Nextflow** I had a hard time understanding the syntax and only after getting more familiar with scripting in **Python** I feel like I am up to the challenge. 

Thus, I will try to list resources to review when I need to re-visit any topic and also provide quick snippets and tips for my future scripting. Finally, as in all my other notebooks, I will register my advances with dates. 

## Useful resources

- Yearly aggregated list of resources to learn Nextflow - [2023 link](https://www.nextflow.io/blog/2023/learn-nextflow-in-2023.html)
- Introduction to Nextflow form Seqera Labs, not updated to use DSL 2 (https://github.com/seqeralabs/nextflow-tutorial)
- The [Nextflow documentation](https://www.nextflow.io/docs/latest/index.html) is a comprehensive guide to concepts and in-depth dynamics

# Important definitions

### Input and output 

Structure to define **_input_** and **_output_**
```
input:
    <input qualifier> <input name>

output:
    <output qualifier> <output name> [, <option>: <option value>]
```

To show the standard output of the process in the terminal use the **Directive:**
    debug true

### Multi line comments

Multi-line comments are specified inside `/*  <COMMENTS>  */`

```groovy
/* This would be a
multi-line comment written out
of the script */
```

### Parameters

Parameters are defined inside the pipeline as `params.variable` and can be called in execution using:

    nextflow run script.nf --variable user_defined_value

### Channel factories

The factory `channel.of` creates a queue channel with values as arguments

    ch1 = channel.of (1, 3, 5, 8)
    ch.view{ "value: $it" }

### Order of assignment output

```groovy
//--snippet--
    output:
    val x 
    val 'str'
    val "${infile.name}".out

workflow{
    (ch_var, ch_str, ch_exp) = foo(ch_dummy)
    /* assignment of foo output is done as follows:
        ch_var = x
        ch_str = 'str'
        ch_exp = "${infile.name}".out
        */
}
```


# Notebook of activities

## 20230301

- When you need to run a process after all previous ones have been completed use `.collect()
- For local testing, I will use the **nextflow_test** conda environment

## 20230314

- The input of a process is the expected input when running the script, not precisely the channel that is going to be used but the format of it
    + val (meta) means that it expects a value
    + path (reads) needs file(s) from a particular PATH

- Pipeline parameters are specified as `params.parameter_name` and can be modified from the default in the pipeline by specifying it in the call as `--parameter_name`

## 20230503 - Working through the Seqera Labs tutorial I

- parameter for execution are defined as `params.variable` and can be specified in call to nextflow as `--variable XXXXX` 

- **_output_** in a process is similar to the return function in python
    - It specifies what is actually produced outside of the process 

## 20230528 - Finished working through the Seqera Labs tutorial II

- Eagle cluster requires you to use apptainer now instead of singularity, but there is an issue with username id that needs to be solved from the developer end
- Working through extended explanations from Seqera, adding notes to repository as it goes
- Learnt more about definition of operators to modify channels as necessary
- In a process defined in nextflow, the order of `output:` defines the assignment

## 202300601 - Reading through additional information from [Seqera](https://training.seqera.io/) and the [nextflow documentation](https://www.nextflow.io/docs/latest/index.html)

- Nextflow stages every process with the staged files in a different directory, guaranteeing that even if two files have the same name, they won't be overwritten
- Expansion of variables sometimes requires the use of single quotes 
- 

