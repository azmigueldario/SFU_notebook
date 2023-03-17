## Useful resources

https://medium.com/23andme-engineering/introduction-to-nextflow-4d0e3b6768d1#5b41

## 20230301

- When you need to run a process after all previous ones have been completed use `.collect()
- For local testing, I will use the **nextflow_test** conda environment

## 20230314

- The input of a process is the expected input when running the script, not precisely the channel that is going to be used but the format of it
    + val (meta) means that it expects a value
    + path (reads) needs file(s) from a particular PATH
- Pipeline parameters are specified as `params.parameter_name` and can be modified from the default in the pipeline by specifying it in the call as `--parameter_name`