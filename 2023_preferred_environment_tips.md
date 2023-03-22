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
