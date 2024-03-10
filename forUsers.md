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
- Displays documentation when run with `--help` flag
     

# Current dependencies
- nvidia-smi
- mpstat (from the sysstat package)
- bash

# Job Queuing
- Jobs can be submitted to the queue by any user using `submitjob` command
- Jobs can be canceled by the user using `canceljob` command ONLY IF it was added to the queue by the same user
- (any user attempts to cancel another user's jobs are logged. Do not do this)
- When a job is assigned hardware or canceled, it is removed from the queue
- Running jobs are not in queue, but are logged separately
- Using PIDs, running jobs can be canceled, use `htop -u <your_username>` to see your running processes

