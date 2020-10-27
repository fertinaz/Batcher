## Batcher: The Stupid Batch Manager
Welcome to the documentation page of `batcher`. This guide provides detailed information about the application and its usage.

### What is Batcher
`batcher` is a command-line application designed to run HPC jobs from your local environment.

### How to install
`batcher` is a single binary and all of its dependencies are embedded in it. Therefore its installation procedure is very simple. Users only need to download the appropriate binary pre-compiled for their platform and locate it in a directory they want.

`Also, batcher` is a cross-platform tool. It can be used on `Linux`, `MacOS` and `Windows` systems with `amd64` architecture. It is developed using `C#` and `DotNetCore3.1` and heavily relies on `Azure Batch SDK`.

### Usage
Following is a sample job submission command:
```
$ batcher submit --file job.yaml                          
```

In this command `job.yaml` is the input file that contains all of the information required to run an HPC simulation. Here is how a typical job file looks like:
```
$ cat job.yaml
Nodes:         2
NodeType:      "Standard_d4_v3"
WallClockTime: "02:00:00"
CaseDirectory: "/fullPathToMyCase/motorBike"
Software:      "openfoam-7"
Command:       "blockMesh && decomposePar -copyZero && mpirun -np 4 simpleFoam -parallel"
```

When the `batcher submit` command is executed, first the input job file is being validated. If that is successful, then a cluster is deployed based on the `Nodes` and `NodeType` information. In this example, user requests a 2 `Standard_d4_v3` type nodes.


Based on this example, following steps are processed when we execute `batcher submit` command:
* Deploy a 1-node cluster.
* Upload case folder `/FullPath/To/YourCase/motorBike` to Azure Storage.
* Pull `openfoam-19.12` container image from Azure Container Registry.
* Mount `motorBike` case under the `$AZ_BATCH_NODE_MOUNTS_DIR` folder in the cluster.
* Run `simpleFoam 2>&1 | tee log.simpleFoam` in this cluster environment.
* Download results into the case directory specified in the job file.

#### Submit

### Packages
Following list of packages are containerized and available for simulations.

#### CP2K
Available CP2K versions are:
* cp2k-7.1

Environment variables:
* `CP2K_DIR=/opt/cp2k-7.1`
* `CP2K_BIN=/opt/cp2k-7.1/bin`
* `CP2K_LIB=/opt/cp2k-7.1/lib`
* `CP2K_DATA=/opt/cp2k-7.1/data`

External libraries and dependencies:
* gcc-7.3
* openmpi-4.0.4
* fftw-3.3.4
* spfft-0.9.8
* libint-2.0.6
* libvdwxc-0.4.0
* openblas-0.3.6
* scalapack-2.1.0

#### Gromacs
Available Gromacs versions are:
* gromacs-2016.5

Environment variables
* `GMX_DIR=/opt/gromacs-2016.5`
* `GMX_BIN=/opt/gromacs-2016.5/bin`
* `GMX_LIB=/opt/gromacs-2016.5/lib`

#### Nektar++
Available Nektar++ versions are:
* nektar-7.1

#### OpenFOAM
Available OpenFOAM versions are:
* openfoam-20.06
* openfoam-19.12
* openfoam-7

### Support or Contact
Having trouble with Pages? Check out our [documentation](https://docs.github.com/categories/github-pages-basics/) or [contact support](https://github.com/contact) and we’ll help you sort it out.
