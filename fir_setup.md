# Setting up virtual environment and running jupyter notebooks on fir

These notes were put together by Anna based on https://docs.alliancecan.ca/wiki/JupyterNotebook.

## Set up virtual environment

These steps only need to be done once each time you want to create a new virtual environment.

1. Navigate to your user directory:

    `cd /home/<username>`

2. Load `python` and `scipy` modules:

    `module load python/3.14`

    `module load scipy-stack`

3. Create a virtual environment. Note that Digital Alliance instructions recommend creating a new environment *for each job* , but I haven't tested this yet. The following assumes you will use this same environment each time you run a job for a particular project: 

    `virtualenv --no-download --clear path/to/venv/<venv_name>`

    where `path/to/venv/` is where you want to store your virtual environment. You can make environments specific to each project if they have different sets of requirements (`python` packages).

4. Activate the environment:

    `source path/to/venv/<venv_name>`

    The name of the environment should now appear at the start of your command prompt.

5. Upgrade `pip` and install jupyter inside your new environment:

    `pip install --no-index --upgrade pip`

    `pip install --no-index jupyter jupyterlab`

6. Create a wrapper script called `notebook.sh` inside the new environment that will allow you to launch notebooks:

    `echo -e '#!/bin/bash\nexport JUPYTER_RUNTIME_DIR=$SLURM_TMPDIR/jupyter\njupyter notebook --ip $(hostname -f) --no-browser' > $VIRTUAL_ENV/bin/notebook.sh`

7. Make the script executable:

    `chmod u+x $VIRTUAL_ENV/bin/notebook.sh`

8. If you want to exit the virtual environment (for example if you want to use a different one or set up a new one), run:

    `deactivate`

## Launch jupyter notebook in interactive job

Unlike on `elephant`, where all resources were available to the whole group at any time, on `fir` you first need to request a 'job', which will allocate a compute node that you can use to run your notebook.

1. When you log into `fir`, navigate to the directory where you want to do your work, load the modules and activate the environment you want to use:

    `cd /home/<username>`

    `module load python/3.14`

    `module load scipy-stack`

    `source path/to/venv/<venv_name>`

2. Request the job allocation, specifying:
    - The time you need it for, noting that jobs of 3 hours or less will be granted almost immediately, and you will have to wait longer for longer-running jobs (possibly several hours).
    - The amount of memory required (how much are we allowed to request?)
    - The 'job' you want to run, which in this case is the notebook script.

    `salloc --time=2:0:0 --mem-per-cpu=3G --ntasks=1 --account=def-ashill srun $VIRTUAL_ENV/bin/notebook.sh`

3. In a separate terminal, `ssh` into `fir` using:

    `ssh -L 8888:<hostname:port> <username>@fir.alliancecan.ca`

    where `<hostname:port>` is indicated in the first URL provided when the job is allocated and looks something like: `fc30559.int.fir.alliancecan.ca:8888`

4. In a web browser, navigate to:

    `http://localhost:8888/?token=<token>`

    where `<token>` is the token string provided in the URL. Note that just clicking on the URL does not work (since the hostname needs to be replaced by `localhost`)

When you're done you work, to close everything just disconnect from `fir` in the second terminal, and press `ctrl+C` twice in the first terminal to shut down the job.
