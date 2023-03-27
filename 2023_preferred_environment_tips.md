## Share between eagle and seagull

Files in the `object_storage` that will be read by the other server must have `r`. Use the following permissions to set read-write access for everyone. 
> chmod 666 filenames/pattern

## Submitting jobs in eagle

As the server is dedicated for **CIDGOH**, there is no need to input the account. 

```
#!/bin/bash                                 
#SBATCH --mem-per-cpu=10G                   # GB of memory per cpu core - max 120GBper node
#SBATCH --time=00:30:00                     # walltime
#SBATCH --cpus-per-task=12                  # CPU cores per task (multithread) - Max 8 
#SBATCH --job-name="assembly_qc_checkm"     # job_name (optional)
#SBATCH --chdir=/scratch/mdprieto/          # change directory before executing optional)
#SBATCH --output=checkm_tutorials.out       # output_file specification (optional)
```

## Set interactive section

To run an interactive allocation for testing, we can use the following command. No need to specify allocation

```
salloc --time=1:30:0 --ntasks=1 --cpus-per-task=8  --mem-per-cpu=8G 
```

## One liners for command line shell

```sh
# remove all but directory matching pattern
find . -mindepth 1 -name 'FOLDER_NAME' -prune -o -exec rm -rf {} \;

# paste files horizontally, useful to create csv
paste file1 file2 --delimiters=,
```
