# Setup

This page describes how to set up the XChem-FFF pipeline tools.

## Prerequisites

Version 1.0 of the XChem-FFF design pipeline is intended for use on the IRIS compute cluster at DLS and on Squonk.

Before continuing, make sure that you followed the [Onboarding](onboarding.md) instructions and have:

- an active DLS FedID;
- configured SSH access to IRIS;
- set up your IRIS profile; and
- got access to Squonk

## Step 1 - Installing and setting up conda

### 1a. Install Miniconda

Set the conda installation prefix to your working directory to avoid filling your home quota:

```bash
cd $HOME2

export CONDA_PREFIX=$HOME2/<WORKDIR>/conda

curl -L -O https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Linux-x86_64.sh
bash Miniforge3-Linux-x86_64.sh -p $CONDA_PREFIX -b
source $CONDA_PREFIX/etc/profile.d/conda.sh

conda info   # confirm base environment is active
```

### 1b. Create the xchem-fff environment

Create a conda environment (execute each line at a time)

```{note}
When running the `pip cache dir` command, please make sure that the output look like `/opt/xchem-fragalysis-2/<WORKDIR>/.cache/pip`. If it doesn't, you might run into an error when running `pip install --no-deps -r $XCHEM_FFF/xchem-fff_requirements.txt`. If that is the case, please contact your FFF coordinator.
```

```bash
conda create -f $XCHEM_FFF/xchem-fff_environment.yml
conda activate xchem-fff
pip cache dir
pip install --no-deps -r $XCHEM_FFF/xchem-fff_requirements.txt
python -c 'import pyrosetta_installer; pyrosetta_installer.install_pyrosetta()'
```

```{note}
The complete installation of the `xchem-fff` with all its packages will take some time, please do not worry if this feels long, this is expected.
```

```{important}
You only need to do this step once. If the `xchem-fff` environment already exists (you can check by running `conda env list`) you don't need to repeat this step, you can activate the environment by running `conda activate xchem-fff`.
```

### 1c. Update your login profile

If you followed the onboarding instructions, your `.bashrc_local` file should contain the following lines:

```bash
if [ $(hostname) == 'cs05r-sc-cloud-30.diamond.ac.uk' ] ; then
    export DATA=/opt/xchem-fragalysis-2
    export HOME2=$DATA/<WORKDIR>
    export XCHEM_FFF=$DATA/XChem-FFF
    export LOGS=$HOME2/logs
    export PIP_CACHE_DIR=$HOME2/.cache/pip
fi
```

Edit `.bashrc_local` using `nano ~/.bashrc_local` to match the following:

```bash
if [ $(hostname) == 'cs05r-sc-cloud-30.diamond.ac.uk' ] ; then
    export DATA=/opt/xchem-fragalysis-2
    export HOME2=$DATA/<WORKDIR>
    export XCHEM_FFF=$DATA/XChem-FFF
    export LOGS=$HOME2/logs
    export PIP_CACHE_DIR=$HOME2/.cache/pip
    export BULK=$HOME2/src/xchem-bulkdock
    export AIZYNTH_CONFIG_FILE=$XCHEM_FFF/syndirella-setup/aizynth_config.yml

    # Activate conda
    export CONDA_PREFIX=$HOME2/conda
    source $CONDA_PREFIX/etc/profile.d/conda.sh
    conda activate xchem-fff
fi
```

```{important}
As explained in the Onboarding instructions: this edit must be done on a **Diamond machine** (e.g. a DLS Linux desktop or login node), not on IRIS. `nano` is not installed on IRIS. Because your home directory is shared across Diamond systems, the change will take effect when you next log in to IRIS.
```

```{note}
Now the `xchem-fff` environment will automatically be activated everytime you ssh into IRIS (you should see something like `(xchem-fff)[<yourfedid>@cs05r-sc-cloud-30 ~]` on the terminal after login into IRIS).
```

Load the changes in your current session:

```bash
source ~/.bashrc_local
```

(setup-step2)=

## Step 2 — Start a Jupyter notebook job on IRIS

The XChem-FFF workflow can be run from a Jupyter notebook on an IRIS compute node. The following steps configure and submit a Jupyter notebook job using SLURM.

### 2a. Set up a `slurm` directory

A few slurm scripts are necessary to run the full pipeline. Some of them can directly be used from the `$XCHEM_FFF` shared directory, but some of them need to be copy and edited in your `$HOME2` directory. Follow the commands below to create a `slurm` directory and set up your slurm scripts:

```bash
cd $HOME2
mkdir slurm
cp $XCHEM_FFF/scripts/notebook.sh $HOME2/slurm/
cp $XCHEM_FFF/scripts/run_python.sh $HOME2/slurm/
sed -i 's/__YOUR_WORKDIR__/<WORKDIR>/g' $HOME2/slurm/run_python.sh # Change <WORKDIR> to your correct working directory (e.g. jsmith)
cp $XCHEM_FFF/scripts/run_bash.sh $HOME2/slurm/
sed -i 's/__YOUR_WORKDIR__/<WORKDIR>/g' $HOME2/slurm/run_bash.sh # Change <WORKDIR> to your correct working directory here as well
```

```{note}
Do not touch the `__YOUR_WORKDIR__` string, this is what sed is looking for to change into you `<WORKDIR>`.
```

### 2b. Configure Jupyter

To configure your Jupyter session, run the following commands on your IRIS termnal. Replace `<WORKDIR>` with the name of your working directory.

```bash
bash "$HOME2/slurm/notebook.sh" -p 9500 -d <WORKDIR> -cd conda -jc "$HOME2/jupyter_slurm" -ce xchem-fff -ajc -js
```
```{note}
This should ask you to set up a password for your notebook (this is optional, if you don't want a password, press RETURN twice)
```

### 2b. Submit the notebook job

Submit the Jupyter notebook job to SLURM:

```bash
sbatch --job-name notebook --exclusive --partition main "$HOME2/slurm/notebook.sh" -p 9500 -d <WORKDIR> -cd conda -jc "$HOME2/jupyter_slurm" -ce xchem-fff -ajc
```

### 2c. Find the `NODELIST`

Check the SLURM queue for your FedID:

```bash
squeue -u <yourfedid>
```

You should see output similar to the following:

```text
JOBID PARTITION NAME USER ST TIME NODES NODELIST(REASON)
1240588 main notebook <yourfedid> R 0-00:01:50 1 host-192-168-222-102
```

Make a note of the host shown under `NODELIST`. You will need its IP address to connect to the notebook.

### 2d. Connect to the notebook

From a new terminal on your local machine, create an SSH connection to the compute node. Replace `<yourfedid>` with your DLS FedID and `<NODE_IP>` with the IP address from the `NODELIST` output above.

```bash
ssh -L 9500:<NODE_IP>:9500 -J <yourfedid>@ssh.diamond.ac.uk <yourfedid>@cepheus-slurm.diamond.ac.uk
```

For example, if the node is `host-192-168-222-102`, then `<NODE_IP>` is `192.168.222.102`.

```{note}
If you are already connected to a Diamond machine (e.g. using NoMachine) you do not need to do a proxy jump. Run the following command instead: `ssh -L 9500:<NODE_IP>:9500 <yourfedid>@cepheus-slurm.diamond.ac.uk`
```

### 2e. Open Jupyter

On your local machine's browser, open a window and go to:

```text
http://localhost:9500
```

## Step 3 — Configure BulkDock

Configure BulkDock with your working directory and Fragalysis submission details. Replace `<WORKDIR>` with the name of your working directory and the values in quotation marks with your own details.

```bash
cd $HOME2
cp $XCHEM_FFF/scripts/run_python.sh $HOME2/slurm/
sed -i 's/__YOUR_WORKDIR__/<WORKDIR>/g' $HOME2/slurm/run_python.sh

python -m bulkdock configure DIR_SLURM_LOGS "$HOME2/logs"
python -m bulkdock configure FRAGALYSIS_EXPORT_SUBMITTER_NAME "YOUR NAME"
python -m bulkdock configure FRAGALYSIS_EXPORT_SUBMITTER_EMAIL "YOUR EMAIL"
python -m bulkdock configure FRAGALYSIS_EXPORT_SUBMITTER_INSTITUTION "YOUR INSTITUTION"
python -m bulkdock configure SLURM_PYTHON_SCRIPT "$HOME2/slurm/run_python.sh"
python -m bulkdock configure EMAIL_ADDRESS "YOUR EMAIL"
python -m bulkdock create-directories
```

Finally, in case you would be working with private targets, make sure the permissions of your BulkDock directory are set to you and group only.

```bash
chmod -R 770 $BULK
```

```{note}
If you don't want anyone else but you to have access to the data, run `chmod -R 700 $BULK` instead.
```

(setup-step4)=

## Step 4 - Start a Jupyter notebook job on Squonk

Before continuing, make sure you followed the {ref}`Squonk Onboarding <squonk-access>` steps to get access to Squonk.

### 4a. Create a Project

On [Squonk](https://data-manager-ui.xchem.diamond.ac.uk/data-manager-ui) home page, after signing in, you should see a box next to the gear icon with the following information:
```
Org: Default
Unit:
Project:
```
To create a project, click on "Settings" logo (the gear icon) next to the box, a panel should show up. If the Unit box (box on the top right) field is empty, write your username or use the dropdown menu to choose your username. Underneath the Unit box, you should see:
* **Delete Unit** (with a bin icon with a cross on it)
* **Edit Unit** (with a pen icon)
* **Create Project** (with a file icon with a + sign on it)

Click on "Create Project", a small panel should pop up. Write the project name (you can name it whatever you want) and choose the Tier using the dropdown menu (Bronze should be enough for Knitwork). You can choose whether you want to keep this Project private or not by ticking the private box. Once you have chosen the name, tier and privacy, click `CREATE`. If successful, you should see your project under `Project Stats`:
```
Project Stats
    Name            Creator         Admins          Tier            Usage           Instances used      Storage used        Allowance           Actions
    Search box      Search box      Search box      Search box      Search box      Search box          Search box          Search box          Search box
O   <Project_Name>  <Username>      <Username>      <Selected_Tier> Green box       0                   0                   10000               3 icons
```
Click on the window icon with an arrow under "Actions" (the first icon) to open your Project in a new window. You will be redirected to a new window, and you will land into your `Project Data` tab. You can consider your Project Data like a file explorer, you can create new directories, upload and download files from there. The Project Data hone is also your starting directory when running a Notebook.

### 4b. Running a Notebook job

To start a Notebook, in your Squonk Project, click on th `Run` tab (next to Project Data). You should see all the applications and jobs available for your to run. To run Knitwork, you need to launch a `JupyterNotebook` app, click on `RUN` in the JupyterNotebook app (should be the first one). A panel should pop up. Fill the `Instance Name` (you can use any name), and click `RUN` (you can leave the other values to default). 

Once started, you should see the `<Instance_name>` you have chosen underneath `RUN`, you can click on it. It should redirect your JupyterNotebook app in the `Results` tab (next to Run). To open it click `OPEN` (next to `TERMINATE`). If `OPEN` isn't available yet, it means that your job hasn't launched yet and is still queueing. If that is the case, wait a little bit and refresh the page, `OPEN` will appear eventually. In the `Results` tab, you should be able to see all the apps and jobs you are currently running under your Project. 

(setup-knitwork)=

### 4c. Setup Knitwork

In the `Results` tab, open the JupyterNotebook app using the `OPEN` button. This will open a Jupyter Notebook in a new window. In the landing page of the Notebook, select `Terminal` under "Other", this should open a "Terminal 1" tab. Install knitwork by running:
```bash
git clone https://github.com/xchem/Knitwork.git
cd Knitwork
pip install -e .
```
Now configure Knitwork by running the following commands:
```bash
python -m knitwork configure GRAPH_LOCATION <graph_location>
python -m knitwork configure GRAPH_USERNAME <graph_username>
python -m knitwork configure GRAPH_PASSWORD <graph_password>
```
If you don't have the `<graph_location>`, `<graph_username>`, `<graph_password>`, contact your FFF coordinator.

Please look at [Squonk documentations](https://data-manager-ui.xchem.diamond.ac.uk/data-manager-ui/docs/guided-tour) if you need more help and guidance or contact your FFF coordinator.


## Next steps

You are now ready to start your first FFF project. Continue to the [Design](design.md) page.