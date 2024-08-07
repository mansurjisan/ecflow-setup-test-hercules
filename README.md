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

# Running ecflow

## Set environment variables
export ECF_HOME=/home/mjisan/ecflow_server

export ECF_INCLUDE=/home/mjisan/ecflow_server/include

export ECF_FILES=/home/mjisan/ecflow_server/files

export ECF_HOST=hercules-login-2.hpc.msstate.edu

export ECF_PORT=11799

export ECF_SCRIPT_CMD="bash %ECF_SCRIPT%"

## Create necessary directories
mkdir -p /home/mjisan/ecflow_server
mkdir -p /home/mjisan/ecflow_server/include
mkdir -p /home/mjisan/ecflow_server/files

## Create mock script files for each task
echo 'echo "Mock script for prepare_environment"' > /home/mjisan/ecflow_server/files/prepare_environment.ecf
echo 'echo "Mock script for copy_static_files"' > /home/mjisan/ecflow_server/files/copy_static_files.ecf
echo 'echo "Mock script for create_param_nml"' > /home/mjisan/ecflow_server/files/create_param_nml.ecf
echo 'echo "Mock script for create_bctides_in"' > /home/mjisan/ecflow_server/files/create_bctides_in.ecf
echo 'echo "Mock script for create_river_forcing_nwm"' > /home/mjisan/ecflow_server/files/create_river_forcing_nwm.ecf
echo 'echo "Mock script for create_surface_forcing_gfs"' > /home/mjisan/ecflow_server/files/create_surface_forcing_gfs.ecf
echo 'echo "Mock script for create_surface_forcing_hrrr"' > /home/mjisan/ecflow_server/files/create_surface_forcing_hrrr.ecf
echo 'echo "Mock script for create_obc_3dth_nudge"' > /home/mjisan/ecflow_server/files/create_obc_3dth_nudge.ecf
echo 'echo "Mock script for create_restart_file"' > /home/mjisan/ecflow_server/files/create_restart_file.ecf

## Restart the EC-Flow server
ecflow_stop.sh
ecflow_start.sh

## Delete the existing suite
ecflow_client --delete force=yes /stf3d_prep

## Load the new workflow definition
ecflow_client --load=stf3d_prep.def

## Begin the workflow
ecflow_client --begin=stf3d_prep

## Check the log file
tail -f /home/mjisan/ecflow_server/hercules-login-2.hpc.msstate.edu.11799.ecf.log


