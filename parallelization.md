<!-- .slide: id="parallelization" -->

## Parallelization

Sometimes in order processing takes too much time

Running tasks in parallel can reduce the execution time

Example: Retrieve the OS version from 100 hosts

--

<!-- .slide: id="jobs" -->

## Jobs

Jobs allow for parallel execution of commands

```powershell
# start a new job
Start-Job { hostname }

# wait for jobs to finish
Get-Job | Wait-Job

# receive output of job
Get-Job | Receive-Job
```

Note that jobs are not lightweight and starting may take some timeouts

See [runspaces](#/runspaces) as an alternative

--

<!-- .slide: id="jobs_progress" -->

## Jobs and progress

First read [progress](#/progress)

Progress in a job is published to caller through a stream

```powershell
$Job = Get-Job
$Job.ChildJobs | ForEach-Object { $_.Progress }
```

Progress information can be [parsed and displayed by caller](https://gist.github.com/nicholasdille/13b21c1c91e8c4e756ef)

--

<!-- .slide: id="remote_jobs" -->

## Remote jobs

See [parallelization](#/invoke_command)

--

<!-- .slide: id="runspaces" -->

## Runspaces

Runspaces provide better performance than jobs

Boe Prox has created a module called [PoshRSJob](https://github.com/proxb/PoshRSJob) for this

The interface is very similar to `Get-Command *-Job`

--

<!-- .slide: id="invoke-parallel" -->

## Invoke-Queue

Invoke the same code to a list of items

See [`Invoke-Queue`](http://dille.name/blog/2015/09/08/processing-a-queue-using-parallel-powershell-jobs-with-throttling/)