# ShirLURM
Shirley's Linux Utility for Resource Management and job queue handling for compute clusters. Implemented as bash scripts and a cronjob. Minimal overhead and dependencies.

# Usage
- `sudo submitjob numcpus numgpus full-path-to-script` (will display generated jobID which is a 10-digit number)
- `sudo canceljob jobID`
- `resourcetracker`
- `jobqueue`
- Note: all users have permissions to run these scripts as root

# Command Documentation For Users

## submitjob
- expects 3 arguments:
  1. numcpus: number of CPU cores to be requested
  2. numgpus: number of GPUs to be requested
  3. Complete path to the script which will handle running the batch job
- will return errors if numcpus or numgpus is outside of limits
- will return error if script does not exist at given path
- Returns jobID if job is successfully added to the queue
- Example: `sudo submitjob 5 1 /home/testuser/codefolder/jobscript.sh`

## canceljob
- expects argument `jobID` as a 10-digit number
- will return an error if the jobID is for a different user than the one requesting cancellation
- Example: `sudo canceljob 0768110458`

## resourcetracker
- returns two numbers: avail_CPUS avail_GPUS

## jobqueue
- will display all jobs in queue, with username and requested resources
- Example: `jobqueue`
     

# Current dependencies
- nvidia-smi
- mpstat (from the sysstat package)
- bash

# Architecture

## About
- the aim of this project is to implement all the basic useful functionality of SLURM with simple linux utilities, and using the operating system's built-in monitoring of resources as much as possible
- due to all code running as bash scripts, it should be portable to any linux distro without any changes
- any compute node does not need to be limited to only running programs sent through approved sessions, as CPU and GPU utilization is measured independent of the resources requested by the currently running jobs
- No overhead of resource-utilization tracking, as the built-in tracking by the OS is used as much as possible
- Scalable to large clusters
- Easy restore from a system crash as the queue is stored in a file on disk
- NFS not required
- Head node not required (but recommended)
- Sophisticated scheduling across the compute cluster has NOT been implemented yet

## Resource Usage Measurement
- CPU utilization is measured as used%=100-idle% for each core(including virtual cores through hyperthreading, easily modifiable to only include physical cores, however)
- CPU core is considered not-in-use if used% is below threshold cpu_t, set to 10 by default
- GPU is considered not-in-use if memory-in-use is below threshold mem_t(1000MiB by default) AND utilization% is below threshold gutil_t(15% by default)

## Job Queuing
- Jobs can be submitted to the queue by any user using `submitjob` command
- Jobs can be canceled by the user using `canceljob` command IF it was added to the queue by the same user
- When a job is assigned hardware or canceled, it is removed from the queue
- Running jobs are not in queue, but can be logged including start times
- Using PIDs, running times for all jobs can be tracked and logged

# How to deploy
- files in command folder need to be in /usr/local/bin with execute permissions, and need to be given sudo permissions (as they need to modify the queue file)
- Add to `/etc/sudoers` the following lines:
  ```
  # Allow all users to run submitjob, canceljob
  ALL ALL=NOPASSWD: /usr/local/bin/submitjob
  ALL ALL=NOPASSWD: /usr/local/bin/canceljob
  ```
- Users should not be able to modify queue file (stored in /usr/local/bin)
- Modify the variables for max_gpus, max_cpus, and not-in-use thresholds
- max_gpus and max_cpus can also be dynamically set on a per-system basis using `nproc` and `nvidia-smi -L | wc -l`, however it is better to set global limits for the cluster
- Install script can be written which automates this process. Useful for large clusters

# TODO
- add the scripts for cluster load management and tracking from the head node
- add the scripts for transferring the required files and scripts to assigned compute node
- Implement sophisticated scheduling functionality

