# Job management and monitoring

Slurm provides a variety of tools that allow a user to manage and
monitor their jobs. This tutorial will briefly introduce these tools and their usage.

## Queued jobs

The `squeue` command is a tool that can be used to pull up information about the
jobs in queue. By default, the squeue command will print out the
*__job ID__*, *__partition__*, *__username__*, *__job status__*,
*__number of nodes__*, and *__name of nodes__* for all jobs queued or
running within Slurm. Usually you don't want information for all
jobs that were queued in the system, so you can specify jobs that only
you are running with the `--user` flag:

```bash
$ squeue --user=username
```

You can output additional information with the `--long` flag. This
flag will print out the non-abbreviated default information with the
addition of a *__timelimit__* field:

```bash
$ squeue --user=username --long
```

The squeue command also provides users with a means to calculate a
job's estimated start time by adding the `--start` flag to your
command. This will append Slurm's estimated start time for each job in
the output information. 
> Note: The start time provided by this command
can be inaccurate. This is because the time calculated is based on
jobs queued or running in the system. If a job with a higher priority
is queued after the command is run, your job may be delayed.

```bash
$ squeue --user=username --start
```

When checking the status of a job, you may want to repeatedly call the
squeue command to check for updates. You can accomplish this by adding
the `--iterate` flag to our squeue command. This will run squeue every
`n` seconds, allowing for a frequent, continuous update of queue
information without needing to repeatedly call squeue:

```bash
$ squeue --user=username --start --iterate=n_seconds
```

Press `ctrl`-`c` to stop the command from looping and bring you back
to the terminal.

For more information on squeue, [visit the Slurm page on
squeue](https://slurm.schedmd.com/squeue.html)

## Stopping jobs

Sometimes you may need to stop a job entirely while it’s running. The
best way to accomplish this is with the `scancel` command. The scancel
command allows you to cancel jobs you are running using the job’s ID. 
The command looks like this:

```bash
$ scancel job-id
```

To cancel multiple jobs, you can use a comma-separated list of job IDs:

```bash
$ scancel job-id1, job-id2, jobid3
```

If your job has sub-jobs, it will be displayed as two numbers separated by an underscore.
The first number is the job-id and the second is the sub-job-id.  You can use scancel
to either cancel all the sub-jobs or just individual ones.  For example, if your
job-id is 439 and it has four sub-jobs 439_1, 439_2, 439_3, 439_4 then 

```bash
$ scancel 439      # Cancels all sub-jobs
$ scancel 439_3    # Only cancels sub-job 3
```

For more information, [visit the Slurm manual on scancel](https://slurm.schedmd.com/scancel.html)

## Currently running jobs

The `sstat` command allows users to easily pull up status information
about their currently running jobs. This includes information about *__CPU usage__*,
*__task information__*, *__node information__*, *__resident set size
(RSS)__*, and *__virtual memory (VM)__*. You can invoke the sstat
command as such:

```bash
$ sstat --jobs=job-id
```

By default, sstat will pull up significantly more information than
what would be needed in the commands default output. To remedy this,
you can use the `--format` flag to choose what you want in our
output. The format flag takes a list of comma separated variables
which specify output data:

```bash
$ sstat --jobs=job-id --format=var_1,var_2, ... , var_N
```

A chart of some these variables and their description is provided here:

Variable    | Description
------------|------------
avecpu      | Average CPU time of all tasks in job.
jobid       | The id of the Job.
ntasks      | Number of tasks in a job.

For an example, to print out a job's job id, average cpu time and number of tasks you would do:

```bash
sstat --jobs=job-id --format=jobid,avecpu,ntasks
```

A full list of variables that specify data handled by sstat can be
found with the `--helpformat` flag or by [visiting the slurm page on
sstat](https://slurm.schedmd.com/sstat.html).

## Examining past jobs

### sacct

The `sacct` command allows users to pull up status information about
past jobs. This command is very similar to sstat, but is used on jobs
that have been previously run on the system instead of currently
running jobs. You can use a job's id...

```bash
$ sacct --jobs=job-id
```

...or your username...

```bash
$ sacct --user=username
```

...to pull up accounting information on jobs run at an earlier time.

By default, sacct will only pull up jobs that were run on the current
day. You can use the `--starttime` flag to tell the command to look
beyond its short-term cache of jobs.

```bash
$ sacct –-jobs=your_job-id –-starttime=YYYY-MM-DD
```

To see a non-abbreviated version of sacct output, use the `--long`
flag:

```bash
$ sacct –--user=username –-starttime=YYYY-MM-DD --long
```

Like `sstat`, the standard output of sacct may not provide the
information we want. To remedy this, you can use the `--format` flag to
choose what you want in your output. Similarly, the format flag is
handled by a list of comma separated variables which specify output
data:

```bash
$ sacct --user=username --format=var_1,var_2, ... ,var_N
```

A chart of some variables is provided below:

Variable    | Description
------------|------------
account     | Account the job ran under.
averss      | Average resident set size of all tasks in the job.
cputime     | Formatted (Elapsed time * CPU) count used by a job or step.
elapsed     | Jobs elapsed time formated as DD-HH:MM:SS.
exitcode    | The exit code returned by the job script or salloc.
jobid       | The id of the Job.
jobname     | The name of the Job.
maxdiskread | Maximum number of bytes read by all tasks in the job.
maxdiskwrite| Maximum number of bytes written by all tasks in the job.
ncpus       | Amount of allocated CPUs.
nnodes      | The number of nodes used in a job.
ntasks      | Number of tasks in a job.
priority    | Slurm priority.
reqcpu      | Required number of CPUs
reqmem      | Required amount of memory for a job.
user        | Username of the person who ran the job.

As an example, suppose you want to find information about jobs that
were run on March 12, 2018. You want to show information regarding the
job name, the number of nodes used in the job, the number of cpus, and the elapsed time. Your command would look like this:

```bash
$ sacct --user=username --starttime=2018-03-12 --format=jobname,nnodes,ncpus,elapsed
```

As another example, suppose you would like to pull up information on
jobs that were run on February 21, 2018. You would like information on
job ID, job name, Number of Nodes used, Number of CPUs used, CPU time, Average CPU time, and elapsed time. Your
command would look like this:

```bash
$ sacct --user=username –-starttime=2018-02-21 --format=jobid,jobname,nnodes,ncpu,cputime,avecpu,elapsed
```

A full list of variables that specify data handled by sacct can be
found with the `--helpformat` flag or by [visiting the slurm page on
sacct](https://slurm.schedmd.com/sacct.html).

### seff

The `seff` command provides somne additional information on completed jobs specifically related to job effeciency (especially CPU and memory utilization).  It requires the job-id.  Here is an example output for a job whose id was 57957:

```bash
[auser@rcs-scsn neuron_mpi]$ seff 57957
Job ID: 57957
Cluster: rcs-sc
User/Group: auser/auser
State: COMPLETED (exit code 0)
Nodes: 4
Cores per node: 24
CPU Utilized: 2-13:11:03
CPU Efficiency: 99.24% of 2-13:39:12 core-walltime
Job Wall-clock time: 00:38:32
Memory Utilized: 2.32 GB
Memory Efficiency: 0.91% of 256.00 GB
```

Based on this output, we can see that it used 96 CPUs (Nodes * Cores per node) and that the CPU utilization  was very good, 99.24%  This basically means that pretty much all the CPUs were in use for the entire time the job ran.  Memory utilization was no so good though.  The job requested 256GB of memory but only used 2.32GB (.91% utilization).

## Controlling queued and running jobs

The `scontrol` command provides extended control of your jobs
run through Slurm. This includes actions like suspending a job,
holding a job from running.

To suspend a job that is currently running on the system, you can use
scontrol with the `suspend` command. This will stop a running job on
its current step that can be resumed at a later time. We can suspend a
job by typing the command:

```
$ scontrol suspend job_id
```

To resume a paused job, we use scontrol with the `resume` command:

```bash
$ scontrol resume job_id
```

Slurm also provides a utility to hold jobs that are queued in the
system. Holding a job will place the job in the lowest priority,
effectively "holding" the job from being run. A job can only be held
if it's waiting on the system to be run. To use the `hold` command to
place a job into a held state:

```bash
$ scontrol hold job_id
```

You can then release a held job using the `release` command:

```bash
$ scontrol release job_id
```

For more information on scontrol, [visit the Slurm page on
scontrol](https://slurm.schedmd.com/scontrol.html)