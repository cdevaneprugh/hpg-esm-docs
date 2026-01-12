# Troubleshooting
## Types of Failures
### Build Failures
### Runtime Failures

## Common SLURM Job Failure Messages

When running jobs on an HPC cluster using SLURM, users may encounter failure messages indicating why a job did not complete successfully. Below are common SLURM error messages and their typical causes and solutions.

1. NODE_FAIL
- **Message**: `Job <job_id> failed, node failure`
- **Cause**: The compute node running the job encountered a hardware or system failure.
- **Solution**:
  - Check system logs (`/var/log/messages`) or consult the cluster administrators.
  - Resubmit the job.
  - Use `sinfo` to check the node status and avoid requesting faulty nodes.

2. TIMEOUT
- **Message**: `Job <job_id> cancelled at <timestamp> because it expired`
- **Cause**: The job exceeded its allocated wall time (`--time` limit).
- **Solution**:
  - Request more time with `--time=HH:MM:SS`.
  - Optimize the job to complete within the time limit.
  - Use checkpointing to save progress and restart the job if needed.

3. OUT_OF_MEMORY
- **Message**: `Job <job_id> killed due to out-of-memory (OOM) condition`
- **Cause**: The job used more memory than allocated (`--mem` or `--mem-per-cpu`).
- **Solution**:
  - Increase memory allocation using `--mem=XXGB` or `--mem-per-cpu=XXMB`.
  - Profile memory usage using `sstat -j <job_id> --format=MaxRSS`.
  - Optimize the program to use memory more efficiently.

4. FAILED
- **Message**: `Job <job_id> failed with exit code <X>`
- **Cause**: The program crashed due to an error such as segmentation fault or missing dependencies.
- **Solution**:
  - Check the job output file (`slurm-<job_id>.out`) for error messages.
  - Debug using tools like `gdb`, `strace`, or `valgrind`.
  - Ensure all required modules and libraries are loaded before job execution.

5. DEPENDENCY NEVER SATISFIED
- **Message**: `Job <job_id> dependency never satisfied`
- **Cause**: A dependent job (`--dependency=afterok:<job_id>`) failed or was never completed.
- **Solution**:
  - Check the status of the parent job using `sacct -j <job_id>`.
  - Resolve any failures in the dependency chain before resubmitting.
  - Consider removing dependencies if they are not necessary.

## Log Files
```bash
# cd to case directory
cd /case/directory/log/files

# list logs by time modified
ls -t

# inspect the most recently modified log file (most likely where failure is indicated)
tail logfile
```

Another way is to grep for errors recursively through your case files.
```bash
# -i flag to ignore case
# -r flag to search recursively
grep -ir error
```

## The CaseStatus File


## Did My Case Fail, or Time Out?<a name="fail_vs_timeout"></a>
There may be a situation that arises where it is difficult to tell if a case has failed, or just timed out. Here's one way to check if it is a time out issue.

```bash
# cd to your case's run directory
cd /blue/GROUP/USER/earth_model_output/cime_output_root/CASE/run

# save the names of the run time log files to a `bash` variable
LOGS=$(ls | grep .log)

# use the stat command to see when they all were last modified
stat $LOGS | grep Modify
```

If all the times printed to the terminal are within a few seconds to a few minutes of each other, your case likely timed out. You can try rebuilding the case after increasing the `JOB_WALLCLOCK_TIME` variable.

# 11.4. Troubleshooting runtime problems ([FROM CIME DOCS](https://esmci.github.io/cime/versions/master/html/users_guide/troubleshooting.html))
To see if a run completed successfully, check the last several lines of the cpl.log file for a string like SUCCESSFUL TERMINATION. A successful job also usually copies the log files to the $CASEROOT/logs directory.

Check these things first when a job fails:

Did the model time out?

Was a disk quota limit hit?

Did a machine go down?

Did a file system become full?

If any of those things happened, take appropriate corrective action (see suggestions below) and resubmit the job.

If it is not clear that any of the above caused a case to fail, there are several places to look for error messages.

Check component log files in your run directory ($RUNDIR). This directory is set in the env_run.xml file. Each component writes its own log file, and there should be log files for every component in this format: cpl.log.yymmdd-hhmmss. Check each log file for an error message, especially at or near the end.

Check for a standard out and/or standard error file in $CASEROOT. The standard out/err file often captures a significant amount of extra model output and also often contains significant system output when a job terminates. Useful error messages sometimes are found well above the bottom of a large standard out/err file. Backtrack from the bottom in search of an error message.

Check for core files in your run directory and review them using an appropriate tool.

Check any automated email from the job about why a job failed. Some sites’ batch schedulers send these.

Check the archive directory: $DOUT_S_ROOT/$CASE. If a case failed, the log files or data may still have been archived.

Common errors

One common error is for a job to time out, which often produces minimal error messages. Review the daily model date stamps in the cpl.log file and the timestamps of files in your run directory to deduce the start and stop time of a run. If the model was running fine, but the wallclock limit was reached, either reduce the run length or increase the wallclock setting.

If the model hangs and then times out, that usually indicates an MPI or file system problem or possibly a model problem. If you suspect an intermittent system problem, try resubmitting the job. Also send a help request to local site consultants to provide them with feedback about system problems and to get help.

Another error that can cause a timeout is a slow or intermittently slow node. The cpl.log file normally outputs the time used for every model simulation day. To review that data, grep the cpl.log file for the string tStamp as shown here:

> grep tStamp cpl.log.* | more
The output looks like this:

tStamp_write: model date = 10120 0 wall clock = 2009-09-28 09:10:46 avg dt = 58.58 dt = 58.18
tStamp_write: model date = 10121 0 wall clock = 2009-09-28 09:12:32 avg dt = 60.10 dt = 105.90
Review the run times at the end of each line for each model day. The “avg dt =” is the average time to simulate a model day and “dt = “ is the time needed to simulate the latest model day.

The model date is printed in YYYYMMDD format and the wallclock is the local date and time. In the example, 10120 is Jan 20, 0001, and the model took 58 seconds to run that day. The next day, Jan 21, took 105.90 seconds.

A wide variation in the simulation time for typical mid-month model days suggests a system problem. However, there are variations in the cost of the model over time. For instance, on the last day of every simulated month, the model typically writes netcdf files, which can be a significant intermittent cost. Also, some model configurations read data mid-month or run physics intermittently at a timestep longer than one day. In those cases, some variability is expected. The time variation typically is quite erratic and unpredictable if the problem is system performance variability.

Sometimes when a job times out or overflows disk space, the restart files will get mangled. With the exception of the CAM and CLM history files, all the restart files have consistent sizes.

Compare the restart files against the sizes of a previous restart. If they don’t match, remove them and move the previous restart into place before resubmitting the job. See Restarting a run.

It is not uncommon for nodes to fail on HPC systems or for access to large file systems to hang. Before you file a bug report, make sure a case fails consistently in the same place.

Rerunning with additional debugging information

There are a few changes you can make to your case to get additional information that aids in debugging:

Increase the value of the run-time xml variable INFO_DBUG: ./xmlchange INFO_DBUG=2. This adds more information to the cpl.log file that can be useful if you can’t tell what component is aborting the run, or where bad coupling fields are originating. (This does NOT require rebuilding.)

Try rebuilding and rerunning with the build-time xml variable DEBUG set to TRUE: ./xmlchange DEBUG=TRUE.

This adds various runtime checks that trap conditions such as out-of-bounds array indexing, divide by 0, and other floating point exceptions (the exact conditions checked depend on flags set in macros defined in the cmake_macros subdirectory of the caseroot).

The best way to do this is often to create a new case and run ./xmlchange DEBUG=TRUE before running ./case.build. However, if it is hard for you to recreate your case, then you can run that xmlchange command from your existing case; then you must run ./case.build --clean-all before rerunning ./case.build.

Note that the model will run significantly slower in this mode, so this may not be feasible if the model has to run a long time before producing the error. (Sometimes it works well to run the model until shortly before the error in non-debug mode, have it write restart files, then restart after rebuilding in debug mode.) Also note that answers will change slightly, so if the error arises from a rare condition, then it may not show up in this mode.
