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

#!/bin/bash                                 
#SBATCH --mem-per-cpu=10G                   
#SBATCH --time=00:30:00                     
#SBATCH --cpus-per-task=12                  
#SBATCH --job-name="assembly_qc_checkm"     
#SBATCH --chdir=/scratch/mdprieto/          
#SBATCH --output=checkm_tutorials.out       
```

To use more resources, we would have to distribute tasks among the available nodes. 
The cluster has 10 compute nodes, each with 120 GB of available memory and 16 cores

```sh
#!/bin/bash                                 
#SBATCH --mem-per-cpu=11G                           # GB of memory per cpu core - max 120GBper node
#SBATCH --ntasks=6                                  # number of tasks per node
#SBATCH --nodes=2                                   # requested number of nodes
#SBATCH --time=20:00:00                             # walltime
#SBATCH --cpus-per-task=3                           # CPU cores per task (multithread) - Max 8 
#SBATCH --job-name="taxprof_pipeline_cf"            # job_name (optional)
#SBATCH --chdir=/scratch/mdprieto/                  # change directory before executing optional)
#SBATCH --output=./jobs_output/cfseed_taxprof_cf.out      

######################################################################################################
```


## Set interactive section

To run an interactive allocation for testing, we can use the following command. No need to specify allocation

```
# cedar must specify allocation
salloc --time=1:30:0 --ntasks=1 --cpus-per-task=4  --mem-per-cpu=8G --account=def-whsiao-ab


# eagle
salloc --time=1:30:00 --ntasks=1 --cpus-per-task=6  --mem-per-cpu=4G 
    # max in a node
salloc --time=1:30:00 --ntasks=1 --cpus-per-task=8  --mem-per-cpu=11G 
```

## Data transfer between cluster

Install and prepare `globus personal connect`

    wget https://downloads.globus.org/globus-connect-personal/linux/stable/globusconnectpersonal-latest.tgz
    tar xzf globusconnectpersonal-latest.tgz


Run without GUI, allows recognition by web server

    # eagle
    $HOME/.globusconnectpersonal-3.2.0/globusconnectpersonal -start &

Inside the same environment, an effective way to transfer big files with the capacity to restart is **rsync**. A large memory allocation to a single core speeds up the process as it is not inherently parallel. 

    rsync -avi --partial --progress  PATH/SOURCE_FILE PATH/DESTINATION_FILE



## One liners for command line shell

```sh
# remove all but directory matching pattern
find . -mindepth 1 -name 'FOLDER_NAME' -prune -o -exec rm -rf {} \;

# paste files horizontally, useful to create csv
paste file1 file2 --delimiters=,

# branch a repo and set upstream tracking
git checkout --track origin/BRANCH_NAME
```

## Using pushd and popd

- `pushd` adds a directory to the stack 
    - `push -n` adds a directory as second in the stack
    - `push +2` moves to the second directory in the stack 
- `dirs` lets you see the stack
- `popd` moves to the last added directory in the stack and removes it from the list

## Singularity was changed to apptainer in 2023

Load in eagle with  

> module load  StdEnv/2020 apptainer/1.1.8