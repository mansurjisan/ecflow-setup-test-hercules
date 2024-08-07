# ecflow-setup-test-hercules

This repository documents the setup and installation process of ECMWF's ecFlow on the MSU Hercules cluster.

## Environment Setup

### Modules Used

The following modules were loaded for the installation:

1. glx/1.4
2. libxkbcommon/1.4.0
3. zlib/1.2.13
4. sqlite/3.39.4
5. qt/5.15.8
6. cmake/3.26.3
7. miniconda3/24.3.0

To load these modules, use:

module load glx/1.4 libxkbcommon/1.4.0 zlib/1.2.13 sqlite/3.39.4 qt/5.15.8 cmake/3.26.3 miniconda3/24.3.0

### Installation Process
### Initial Attempt: CMake

Initially, an attempt was made to install ecFlow using CMake commands. However, this method was unsuccessful due to version compatibility issues with CMake.

### Successful Method: Conda

After the CMake attempt, the installation was successfully completed using Conda. Here are the steps:

### Create a new Conda environment:
conda create -n myecflow

### Activate the new environment:
conda activate myecflow

### Install ecFlow from the conda-forge channel:
conda install -c conda-forge ecflow

### Verify the installation:
ecflow_client --version




