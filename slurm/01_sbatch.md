# Submitting jobs

Typically you will create a text file (refered to here as a job script) to specify the details of your job.  A job script is set of Linux commands paired with a set of resource requirements that can be passed to the Slurm job scheduler. Slurm will then generate a job according to the parameters set in the job script. Any commands that are included with the job script will be run within the job.  A job script will usually consist of four components:

- The first line must be `#!/bin/bash`
- Job resources and directives
- Load/initialize software
- Run the program(s)

Once you have created your script, you can submit your job using `sbatch`.  For example if your job script file is called *neuron.job* then you would submit your job as:

```bash
$ sbatch neuron.job
```
## Slurm batch directives

These directives specify resource requirements and other job information (e.g., job name). These directives must come after `#!/bin/bash` and before any commands are issued in the job script. Each directive contains a flag that requests a resource the job would need to complete execution. An sbatch directive always starts with `#SBATCH` and takes the general form:

```bash
#SBATCH --<flag>=<value>
```

For example, if you wanted to use only 2 nodes to run your job, you would include the following directive:

```bash
#SBATCH --nodes=2
```

| Type               | Description                                         | Flag                       | Example                    |
| :----------------- | :-------------------------------------------------- | :------------------------- | :------------------------- |
| Job Name           | Name your job so you can identify it in the queue   | --job-name=jobname         | --job-name=neuron-job      |
| Output file        | Specify a file that                                 | --output=file              | --output=myjob.out         |
| Sending email      | Receive email at beginning or end of job completion | --mail-type=type           | --mail-type=END, FAIL    |
| Email address      | Email address to receive the email                  | --mail-user=user           | --mail-user=auser@fadm.edu |
| Number of nodes    | The number of nodes needed to run the job           | --nodes=nodes              | --nodes=4                  |
| Number of tasks    | The ***total*** number of processes needed to run the job | --ntasks=processes   | --ntasks=96                |
| Tasks per node     | The number of processes you wish to assign to each node | --ntasks-per-node=processes | --ntasks-per-node=24   |
| Partition          | Specify a partition. Currently only needed if using a GPU | --partition=partition | --partition=gpus         |
| Generic resource   | Specify a GRES. Currently only needed if using a GPU | --gres=gres:#              | --gres=gpu:1     |
| Wall time          | The max amount of time your job will run for        | --time=wall time           |


**A few comments about these directives.**

- The directives for emailing are not currently available so do not include them (yet).  We will let users know when these are available.
- Not all job scripts need all of these directives.  In most cases you will specify:
  - job name
  - output file (See below for more details on output file)
  - email directives (when it becomes available).
  - wall time

- For jobs that can be split up to run in paralell (e.g. using MPI) you should specify:
  - number of tasks
  - tasks per node **Note:** This value should not exceed the number of CPUs for a resource otherwise the job may not run.  On the cluster this value is 40 CPUs per node. We also recommend you try to balance the number of tasks across nodes.  For example, if your job requires 96 total tasks, then you might set the tasks per node to 24 which will likely use 4 nodes.

- For software that runs on the GPU you will need to specify
  - partition (A **partition** is just a group of related resources like GPUs)
  - generic resource (which will usually be in the form: `gres=gpu:1`  The number after the colon is the number of GPUs needed.  In almost all
    cases the value should be 1

**Output files**

Some software packages output messages and errors directly to the terminal as they run.  The `output` directive specifies a file name where such messages will be written to instead of being logged to the terminal.  It **does not** specify the name of output file(s) which your particular program may use to capture output for its calculations.  Those still need to be specified as you normally would when running the code.

For this directive we recommend using one of the following two forms depending on your specific curcumstances:
- `output=myfilename_%j.out`  Here the `%j` will be replaced by the job-id Slurm generates automatically.  So if the job id is 439 then the output would go to `myfilename_439.job`.  This is useful to differentiate output files if you run the same job script over and over again.
- `output=myfilename_%A_%a.out`  Sometimes a job will have sub-jobs that get run (for example, running the same simulation multiples times but each run uses different parameters).  In this case, `%A` refers to the job-id and `%a` refers to the sub-job-id.  For example if you ran 3 simulations, their sub-job-ids might be 1, 2, and 3 producing output files, `myfilename_439_1.out`, `myfilename_439_2.out`, and `myfilename_439_2.out`

A full list of commands [can be found in Slurm's documentation for sbatch.](https://slurm.schedmd.com/sbatch.html)

## Resource limits

Currently there are no limits in place when it comes to running jobs on the cluster.  However, if an individual user begins to "hog" the cluster,
then we may enforce limits for that indivdual user.  Further, general limits may be introdcuced in the future as needs and usage change.

## Example submission scripts

One thing to note is that these examples are relatively simple.  Often times your submission will include many jobs to be run.  In addition, some pieces of software (notably Miniconda and Gaussian) require some
additional steps in order to use them.  These steps and more general information about the software loaded on the cluster can be found in the [software section](../software/README.md) of the documentation.

### Single job, Single CPU
```bash
#!/bin/bash

# Set Slurm job directives
#SBATCH --job-name=neuron-test-job
#SBATCH --output=neuron_%j.out

# Turn on mail notification. There are five possible values:
# NONE, BEGIN, END, FAIL, ALL (including all aforementioned)
# For more values, check "man sbatch"
#SBATCH --mail-type=END,FAIL
#SBATCH --mail-user=auser@fadm.edu

# Load the necessary software
module purge
module load miniconda

eval "$(conda shell.bash hook)"
conda activate neuron
cd neuron

# Run the code.  Start by printing the date (to know when the job started)
date

nrnivmodl ./
nrniv beginSimulation.hoc

# Print the date again to know when it ended
date
```

This script will submit a single job that runs on a single CPU.  For such a simple case there really aren't many requirements when it comes to the Slurm directives.  It will email the user when the job completes (or if the job failed).  

### Single parallel job using MPI

```bash
#!/bin/bash

# Set Slurm job directives
#SBATCH --job-name=neuron-mpi-test-job
#SBATCH --ntasks=96
#SBATCH --tasks-per-node=24
#SBATCH --output=neuron_mpi_%j.out
#SBATCH --mail-type=END,FAIL
#SBATCH --mail-user=auser@fadm.edu

# Load the necessary software
module purge
module load miniconda
module load openmpi

eval "$(conda shell.bash hook)"
conda activate neuron
cd neuron_mpi

# Run the code
date

nrnivmodl ./
srun mpiexec -n 96 nrniv -mpi beginSimulation.hoc

date
```

This script is similar to the one above except the code will be run using multiple CPUs (96 in total) in parallel.  Not all software can be run in parallel like this.  It will depend on the software.  Because this example runs a single job, the number of CPUs MPI uses (mpiexec **-n 96**) cannot exceed the number of tasks (**#SBATCH --ntasks=96**)

### Multiple single CPU jobs

```bash
#!/bin/bash

#SBATCH --job-name=neuron-test-job

module purge
module load miniconda

eval "$(conda shell.bash hook)"
conda activate neuron
cd neuron

# Run the jobs
date

# Submit/run 1st job
nrnivmodl ./
srun nrniv beginSimulation.hoc > sim.out 2>&1 &

# Submit/run 2nd job
srun python eval_cands.py candidates.csv > eval.out 2>&1 &

# Submit/run 3rd job
srun ./generateCands -o randomData.txt > gen.out 2>&1 &

wait

date
```

The real power of Slurm is the ability to submit and run multiple jobs at once assuming the necessary resources (e.g., number of CPUs) are available.  In this example, we are submitting three different single CPU jobs to run to run at the same time.

The first thing to notice is the use of *srun* before the program calls.  This is a SLURM command to submit a single run.

The second thing to notice is the use of *>* followed by a filename (e.g. *> sim.out*) when running our programs.  Notice too we did not include the *#SBATCH --output=*.  What is happening here is that each program is redirecting output to it's own file.  This makes it much easier to distinguish the output from the different programs.

Finally each run uses *2>&1 &*.  This does two things.  First, any errors will print to the same file used for output.  Second, the job will run as a background process.  This is important because otherwise the script would wait for the job to finish before submitting the next program to run.  The use of the *wait* command tells the system to wait for all the programs to finish running before finishing the job.

### Multiple similar jobs

```bash
#!/bin/bash

#SBATCH --job-name=neuron-mpi-multi-job
#SBATCH --nodes=1
#SBATCH --ntasks=24
#SBATCH --output=neuron_mpi_%A_%a.out
#SBATCH --array=1-4

module purge
module load miniconda
module load openmpi

eval "$(conda shell.bash hook)"
conda activate neuron

date

cd sim${SLURM_ARRAY_TASK_ID}
nrnivmodl ./
srun mpiexec -n 24 nrniv -mpi beginSimulation.hoc
cd ..

date                          
```

This example uses a powerful feature of Slurm called job arrays.  Some explanation here is needed

1. `--array=1-4` specifies the fact that we will be using a job array, specifically one that will include four jobs whose sub-job-id will be the values 1-4.
2. `${SLURM_ARRAY_TASK_ID}` is a Slurm specific variable that will take on the values specified by the `--array` directive.  Here we are using it to refer to 4 different directories, `sim1`, `sim2`, `sim3`, `sim4`
3. The values for `--nodes` and `--ntasks` (and other directives that might apply to physical resources) apply to each individual sub-job.  This is important for example when doing parallel code runs using MPI.  Here each individual simulation is requesting 24 CPUs on a single node.

In this simulation we have decided to differentiate the runs by placing the appropriate pieces in different directories but theare are many other ways you could setup your job.  More details and examples can be found in [Slurm's job array documentation](https://slurm.schedmd.com/job_array.html)

### GPU jobs

```bash
#!/bin/bash                                                                                                                              
#SBATCH --job-name=heimdall-test-job                                                                                                     
#SBATCH --partition=gpus                                                                                                                 
#SBATCH --gres=gpu:1                                                                                                          
#SBATCH --output=heimdall_test_%j.out                                                                                                    
                                                                                                                                     
module purge                                                                                                                             
module load heimdall                                                                                                                                                                                                                                                            
date
                                                                                                                                     
srun heimdall -f test.fil                                                                                                                     
cat *.cand > beam01.cand                                                                                                                 
                                                                                                                                        
date         
```

In this example, the `heimdall` program runs on a GPU.  We have specified `#SBATCH --partition=gpus` to tell Slurm we need to use the set of nodes which have GPUs and we also specified `#SBATCH --gres=gpu:tesla_v100` to indicate the specific GPU to use.
