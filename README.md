## Batcher: The Stupid Batch Manager
Welcome to the documentation page of `batcher`. This guide provides detailed information about the application and its usage.

### What is Batcher
`batcher` is a command-line application designed to run HPC jobs from your local environment.

### How to install
`batcher` is a single binary and all of its dependencies are embedded in it. Therefore its installation procedure is very simple. Users only need to download the appropriate executable pre-compiled for their `Linux`, `MacOS` or `Windows` system and locate it in a directory they want.

`batcher` is developed using `C#` and `DotNetCore3.1` and heavily relies on `Azure Batch SDK`. It is currently supported on `amd64` architectures.

For `Linux` and `MacOS` users, a sample installation procedure looks like:
```
curl -LO <URL-HERE>/batcher
sudo mv batcher /usr/local/bin
```

If you don't have `sudo` privileges in your environment, you can also use:
```
curl -LO <URL-HERE>/batcher
mkdir -p $HOME/bin # If doesn't exist
mv batcher $HOME/bin
export PATH=$HOME/bin:$PATH # If not included
```

### Usage
Following command show a sample job submission:
```
$ batcher submit --file job.yaml                          
```

In this command, `job.yaml` is the input file that contains all of the information required to run an HPC simulation. Here is how a typical job file looks like:
```
$ cat job.yaml
Project:       "OpenFOAM motorBike case"
Nodes:         2
NodeType:      "Standard_d4_v3"
WallClockTime: "02:00:00"
CaseDirectory: "/fullPathToMyCase/motorBike"
DownloadPath:  "/fullPathToDownloadLocation"
Software:      "openfoam-7"
Command:       "blockMesh && decomposePar -copyZero && mpirun -np 4 simpleFoam -parallel"
```

Each item in the job file represents a specific information regarding the HPC job that user wants to initiate.
* Project: A short description or note to keep track of your simulations.
* Nodes: Number of nodes requested in the cluster.
* NodeType: Type of the node instances. We follow the naming convention created by Azure. According to the [official documentation](https://docs.microsoft.com/en-us/azure/virtual-machines/dv3-dsv3-series?toc=/azure/virtual-machines/linux/toc.json&bc=/azure/virtual-machines/linux/breadcrumb/toc.json), the node type given in the example has 4 vCPUs and 16 GiB memory.
* WallClockTime: Time estimation required to complete the simulation. This restriction has two purposes; it protects users to be over-charged from cases like deadlock, and also **it allows us to check if the users have enough credits in their `batcher` account to complete the job.** One important note; it's certainly safe to write a clock time that is larger than the simulation time. For instance, if a user roughly estimates that simulation takes about 4-6 hours to complete, s/he can specify `WallClockTime` as 12 hours. Then, if the job is completed in, say 5.5 hours, s/he is charged for the actual consumption which is 5.5 hours, not for the `WallClockTime` that is written. However, when the user sends a job, we initially check the credit amount based on the `WallClockTime`. As a result, to get a job started, users need to have enough credits for the maximum cost that job requires. Based on the same example, even if the job finishes in 5.5 hours, user needs to have enough credits to complete a 12-hour job because `WallClockTime` is 12 hours.
* CaseDirectory: Full path to the case folder that includes all the input files that are required to run the simulation. This folder will be uploaded to the `Storage Account` that is created for you on the cloud environment and will then be mounted on the cluster deployed. **The entire process is encrypted**, therefore users don't have to worry about the security and confidentiality of their case files. One restriction for the naming: Folder names with special characters such as `-`, `!` are not supported. 
* DownloadPath: Location of the directory that the results are downloaded to. We want to keep the input directory as original.
* Software: Name of the application that you want to use. These packages are containerized by our team and pushed to our internal container registry. You can find the complete list of tools we support in this guide. We're constantly updating this list, so please check it regularly.
* Command: Commands you need to run to your batch job.

Based on the given example, following steps are processed when `batcher submit` command is executed:
* 2 `Standard_d4_v3` nodes are invoked and a cluster is created. Communication between these nodes are enabled for parallel processing.
* Case folder `/fullPathToMyCase/motorBike` is read and its content is uploaded to a storage container.
* Pull `openfoam-7` container image from our container registry.
* A cloud task is created with the commands specified in the job file. The maximum time for this task is the `WallClockTime`.
* Task is executed.
* Download results into the local file system.

### Packages
Following list of packages are containerized and available for simulations. A summary of the available container images:
* cp2k-7.1
* gromacs-2016.5
* lammps-3mar2020
* nektar-5.0
* openfoam-7
* openfoam-19.12
* openfoam-20.06
* su2-7.0.7

Each tool is compiled from scratch with certain compiler flags for optimization. Detailed information can be in the following subsections.

#### CP2K
Available CP2K versions are:
* cp2k-7.1

Environment variables:
* `CP2K_DIR=/opt/cp2k-7.1`
* `CP2K_BIN=/opt/cp2k-7.1/bin`
* `CP2K_LIB=/opt/cp2k-7.1/lib`
* `CP2K_DATA=/opt/cp2k-7.1/data`

Compiled with:
* gcc-7.3
* openmpi-4.0.4

External libraries and dependencies:
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

Compiled with:
* gcc-4.8
* openmpi-4.0.4

#### Lammps
Available Lammps versions are:
* lammps-3mar2020

Name of the binaries:
* lammps
* lmp_g++_openmpi
* lmp_mpi

Environment variables
* `LAMMPS_DIR=/opt/lammps-3Mar2020`
* `LAMMPS_BIN=$LAMMPS_DIR/bin`

Compiled with:
* gcc-4.8
* openmpi-4.0.4

External libraries and dependencies:
* asphere
* cgdna
* class2
* colloid
* coreshell
* diffraction
* granular
* kspace
* manybody
* mc
* meamc
* misc
* molecule
* qeq
* replica
* rigid
* reaxc
    
#### Nektar++
Available Nektar++ versions are:
* nektar-5.0

Environment variables
* `NEKTAR_DIR=/opt/nektar++-5.0.0`
* `NEKTAR_BIN=$NEKTAR_DIR/bin`
* `NEKTAR_LIB=$NEKTAR_DIR/lib`
* `NEKTAR_INC=$NEKTAR_DIR/include`

Compiled with:
* gcc-4.8
* openmpi-4.0.4

External libraries and dependencies:
* openlas
* boost
* scotch
* fftw
* hdf5
* arpack
* metis
* petsc

#### OpenFOAM
Available OpenFOAM versions are:
* openfoam-7
* openfoam-19.12
* openfoam-20.06

OpenFOAM is sourced by default in all containers. Each container image has the same installation procedure and their bashrc file is located under:
* /opt/OpenFOAM/OpenFOAM-7/etc/bashrc
* /opt/OpenFOAM/OpenFOAM-19.12/etc/bashrc
* /opt/OpenFOAM/OpenFOAM-20.06/etc/bashrc

Compiled with:
* gcc-4.8
* openmpi-4.0.4

#### SU2
Available SU2 versions are:
* su2-7.0.7

Environment variables
* `SU2_DIR=/opt/su2-7.0.7`
* `SU2_BIN=$SU2_DIR/bin`
* `SU2_RUN=$SU2_BIN`

Compiled with:
* gcc-9.3
* mpich-3.3.2

External libraries and dependencies:
* openlas-0.3.6
* python-3.6

### Support or Contact
If you need more information or would like to ask further question about `batcher`, please do not hesitate to reach out.
* GH issues: https://github.com/fertinaz/Batcher/issues
* Direct contact with project maintainer: Fatih Ertinaz - fertinaz [at] gmail [dot] com
