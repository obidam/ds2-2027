# Coiled memo for students

## Setup

### Coiled account and 2026 class workspace:
- Create a Coiled account with your academic email at:
  https://cloud.coiled.io/signup

- Send your Coiled account login by email to gmaze@ifremer.fr to get access to shared resources from the "class-2026" Coiled workspace.

### Local Python environment

#### For MacOS and Linux systems

- Download the *ds2-coiled-2026* environment definition we created for the class from here:  
  https://github.com/obidam/ds2-2026/blob/main/practice/environment/coiled/environment-coiled-pinned.yml

- Install and activate this Python 3.11 environment with a package manager like miniconda:  
```bash
miniconda env create -f environment-coiled-pinned.yml
miniconda activate ds2-coiled-2026
```

#### For Windows system

- Download the *ds2-coiled-2026* environment definition **for Windows** we created for the class from here:  
  https://github.com/obidam/ds2-2026/blob/main/practice/environment/coiled/environment-coiled-pinned-windows.yml

- Install and activate this Python 3.11 environment with a package manager like miniconda:  
```bash
miniconda env create -f environment-coiled-pinned-windows.yml
miniconda activate ds2-coiled-2026
```

- Finally, since the google-cloud-sdk library is not available with a package manager, it must be installed manually, and you need to install the specific **release 550** from:
  https://storage.googleapis.com/cloud-sdk-release/google-cloud-cli-550.0.0-windows-x86_64.zip

- Note that we won't actually use the Google Cloud SDK from the notebooks, but it is required by Coiled to set up the Dask cluster on the Google Cloud Perform. So, you don't need to configure the SDK with ``gcloud init``.

### Connect to Coiled

- You now need to connect to your Coiled account with the appropriate workspace:
```bash
coiled login --workspace class-2026
```

- Note that you don't have to follow the Step 2 "Connect your cloud" of the Coiled *Get Started* workflow.  
  Only the Step 1 "Authenticate your compute" must be validated.

- If you need packages that are missing from the environment, email gmaze@ifremer.fr to ask for its addition and installation.  
  The environment *ds2-coiled-2026* must be updated and uploaded to all cluster workers on the GCP.

### Add the new environment kernel to Jupyter

```bash
python -m ipykernel install --user --name=ds2-coiled-2026
```

## Usage

### Connect to a cluster

You can connect to one existing clusters, typically:

- ``ds2-highcpu``: 160 vCPUs, 3.75 GB of system memory per vCPU
- ``ds2-highmem``: 20 vCPUs, 6.5 GB of system memory per vCPU

You can connect to the cluster and make it available to Dask for your computation like this:
```python
import coiled
cluster = coiled.Cluster(workspace="class-2026", name="ds2-highcpu")
client = cluster.get_client()
```

**Computation examples**
```python
def inc(x):
    return x + 1
future = client.submit(inc, 10)
future.result() # returns 11
```

```python
import dask.array as da
x = da.random.normal(10, 0.1, size=(20000,20000), chunks= (1000,1000))
y = x.mean(axis=0)[::100]
y.compute();
```


### Jupyter notebook

From your computer, you can use Coiled to create a Jupyter Lab instance in the cloud that will synchronise your local files to the cloud instance. 

For the synchronisation to work, you're required to install the [``mutagen``](https://mutagen.io/documentation/introduction/installation) software for your OS.

After that you can navigate to a specific folder and ask coiled to create a Jupyter lab instance in the cloud that will synchronize notebooks to your local folder.
```bash
conda activate ds2-coiled-2026
coiled notebook start --sync --workspace class-2026 --idle-timeout '1 hour' --vm-type n1-highmem-2 --name notebook-user --software ds2-coiled-2026
```

From a notebook in the remote Jupyter Lab instance, you can connect to the class clusters like this:
```python
import coiled
from dask.distributed import Client
cluster = coiled.Cluster(workspace="class-2026", name="ds2-highcpu")
client = cluster.get_client()
```
