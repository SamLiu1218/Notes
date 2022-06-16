This note covers how to set up remote jupyter that runs on HPC but is accessible in local browser.

1. If jupyterlab is not installed, install it with 
 
```mamba install -c conda-forge jupyterlab```

(```mamba``` is a conda substitute but much faster. Use conda if you still like it, or install mamba with ```conda install mamba -c conda-forge```.)

2. Create jupyter config with

```jupyter notebook --generate-config```

3. Setup password with

```jupyter notebook password```

4. Create a script (e.g., jupyter.sbatch) (put the script somewhere like ```~/sbatch-scr/```) that initiates the jupyter:

```shell
#!/bin/bash
#SBATCH --qos=batch
#SBATCH --time=24:00:00
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --mem=64000

mkdir -p ~/sbatch-out/jupyter-logs
localcores=${SLURM_TASKS_PER_NODE}
export PATH=#HOME/miniconda3/bin:$PATH ## change it to your anaconda installation if needed

PORT=12304    ## Modify the port on your need
HOST=$(hostname -A)
jupyter lab --ip=$HOST --port=$PORT --no-browser --notebook-dir=$HOME   ## change --notebook-dir on your need

```


5. Create a bash alias for submitting the job: ```nano ~/.bash_aliases``` and add the function below to the end of the file:
```shell
launch_jupyter() {
  jobname=jupyterlab-$USER
  outputfile=~/sbatch-out/jupyter-logs/$(date '+%s').log
  launch_file=~/sbatch-scr/jupyter.sbatch   ## change it to where you put the jupyter.sbatch
  jobid=$(sbatch --job-name=$jobname --output=$outputfile $launch_file)
  
  echo "Waiting for jupyter to start up..."
  sleep 30
  if [ -f ${outputfile} ]; then
        echo "Try accessing the following jupyter-lab server:"
        grep 'sumner.jax.org' $outputfile | sed 's/ :/:/'
  else
        echo "Looks like your job '$jobid' is queued. Monitor when it starts with"
        echo -e "\tsqueue -u $USER"
        echo "To find the server IP and port, grep this file:"
        echo -e "\t${outputfile}"
        echo "e.g."
        echo -e "grep 'sumner.jax.org' $outputfile"
  fi
}
```

6. type ```source ~/.bash_aliases``` to execute it. Also ```nano ~/.bashrc``` and add ```source ~/.bash_aliases``` to the end of it.

7. Now you can launch remote jupyte by simply typing ```launch_jupyter```! If successful, it will promp something like this:
```
Waiting for jupyter to start up...
Try accessing the following jupyter-lab server:
[I 2022-05-02 15:05:40.528 ServerApp] http://sumner015.sumner.jax.org:11218/lab
```
Copy the link to your browser and have fun.


8. For jupyter lab to detect your conda environments as kernels:

First activate the enviroment

```conda activate YOUR_ENV```

then ```mamba install ipykernel```

then ```ipython kernel install --user --name=YOUR_KERNEL```

replace YOUR_ENV and YOUR_KERNEL to your env and kernel name.

(Don't forget to deactivate the env.)




