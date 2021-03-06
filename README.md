# Dask cluster on Snellius

The following instructions allow one to setup a Dask cluster on the [Snellius](https://servicedesk.surfsara.nl/wiki/display/WIKI/Snellius) supercomputer at [SURF](https://www.surf.nl) using [Dask-Jobqueue](https://jobqueue.dask.org/en/latest/index.html).

**Note**: Access to Snellius from non-white-listed IP addresses is only possible via [a separate login server](https://servicedesk.surfsara.nl/wiki/pages/viewpage.action?pageId=30660265#FAQSnellius/Lisa-HowtoconnecttotheSnelliussystemfromabroad) (or via VPN) 

## Install Miniconda

Download and install Miniconda:
```shell
wget https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh
chmod +x Miniconda3-latest-Linux-x86_64.sh
./Miniconda3-latest-Linux-x86_64.sh
```

Type `yes` to accept the license term, select the install location (press ENTER to accept the default ${HOME}/miniconda3), then type `yes` to initialize Miniconda3. Logout/login to activate conda. 

## Install Dask

Create an environment with Dask and dask-jobque:
```shell
conda create -n dark_generators -c conda-forge dask dask-jobqueue ipython
```

## Start a Dask cluster

Activate the new environment:
```shell
conda activate dark_generators
```

The following snippet configure a Dask cluster with 2 workers per node (`processes=2`) and one thread per worker (`cores=2`, per node). The same python executable used to run the script  will be used to start the workers. Calling `cluster.scale` submits the jobs and actually starts the cluster. 
**Note**: this snippet needs to run on a compute node (not on the login node), e.g. via an interactive or batch SLURM jobs (see [here](https://userinfo.surfsara.nl/systems/cartesius/usage/batch-usage)).

```python
from dask.distributed import Client

from dask_jobqueue import SLURMCluster

# no cluster is
cluster = SLURMCluster(
    queue="normal",  # "normal" or "gpu" (max 5d),
                     # "short" or "gpu_short" (max 1h)
    memory="60GB",  # memory per node
    processes=2,  # num workers per node
    cores=2,  # num threads per node
    job_mem="60GB",  # per job
    job_cpu=24,  # per job
    local_directory="$TMPDIR",
    walltime="01:00:00"
)

print(cluster.job_script())
# #!/usr/bin/env bash
#
# #SBATCH -J dask-worker
# #SBATCH -p short
# #SBATCH -n 1
# #SBATCH --cpus-per-task=24
# #SBATCH --mem=60GB
# #SBATCH -t 01:00:00
#
# /home/nattino/miniconda3/envs/dark_generators/bin/python -m distributed.cli.dask_worker tcp://145.100.203.172:40190 --nthreads 1 --nprocs 2 --memory-limit 27.94GiB --name dummy-name --nanny --death-timeout 60 --local-directory $TMPDIR --protocol tcp://

cluster.scale(jobs=3)

client = Client(cluster)
```

## Examples

See [this example](examples/hello_world) for test scripts employed to run an executable through the Dask cluster and for instructions on how to access the Dask dashboard from your local computer.

## Other resources

* [Pangeo on HPC guide](https://pangeo.io/setup_guides/hpc.html)
