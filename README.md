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

## Force delete 

ecflow_client --delete force /stf3d_prep_new

## Loading suite definition
ecflow_client --load=test.def
ecflow_client --begin=test

## to replace test suite or to replace part of the suite:
ecflow_client --replace=/test test.def
ecflow_client --replace=/test/t2 test.def

## to suspend a suite
ecflow_client --suspend=/test

## to replace 
ecflow_client --replace=/test test.def

## check status
ecflow_client --get_state=/test

## Force Delete
ecflow_client --delete force=yes /test

## Load and Begin
ecflow_client --load=test.def
ecflow_client --begin=test


## def file 
# Definition of the suite test.
suite test
   edit ECF_INCLUDE "/home/mjisan/course"   # replace '$HOME' with the path to your home directory
   edit ECF_HOME    "/home/mjisan/course"
   family f1
      task t1
         edit SLEEP 20
      task t2
         edit SLEEP 20
   endfamily
endsuite


## def file 
# Definition of the suite test.
suite test
   edit ECF_INCLUDE "/home/mjisan/course"   # replace '$HOME' with the path to your home directory
   edit ECF_HOME    "/home/mjisan/course"
   family f1
      task t1
         edit SLEEP 20
      task t2
         edit SLEEP 20
   endfamily
endsuite


## example case: example namelist generation

### 1. Create a definition file

I used a Python script to generate a definition file.

create_namelist_def.py

import os
from ecflow import Defs, Suite, Family, Task, Edit

# Define the home directory
home = os.path.join(os.getenv("HOME"), "course")

# Create the suite definition
defs = Defs(
    Suite('namelist_suite',
          Edit(ECF_INCLUDE=home, ECF_HOME=home),
          Family('preprocess',
                 Task('create_param_nml',
                      Edit(SLEEP=20, ECF_SCRIPT=os.path.join(home, "preprocess", "create_param_nml.ecf"))
                 )
          )
    )
)

# Check job creation
job_check_errors = defs.check_job_creation()
if job_check_errors:
    print("Job creation failed with the following errors:")
    print(job_check_errors)
else:
    print("Job creation successful.")

# Save the suite definition to a file
with open("namelist.def", "w") as f:
    f.write(str(defs))
    print("Suite definition written to namelist.def")

this will generate namelist.def file 


### 2. Create the necessary directories and files:

mkdir -p ~/course/preprocess
touch ~/course/preprocess/create_param_nml.ecf
touch ~/course/head.h
touch ~/course/tail.h

### 3. head.h file (needs to be in home dir i.e. /home/mjisan/course; copied from ecflow website) 

#!/bin/bash

set -e          # stop the shell on first error
set -u          # fail when using an undefined variable
set -x          # echo script lines as they are executed
set -o pipefail # fail if last(rightmost) command exits with a non-zero status

# Defines the variables that are needed for any communication with ECF
export ECF_PORT=%ECF_PORT%    # The server port number
export ECF_HOST=%ECF_HOST%    # The host name where the server is running
export ECF_NAME=%ECF_NAME%    # The name of this current task
export ECF_PASS=%ECF_PASS%    # A unique password, used for job validation & zombie detection
export ECF_TRYNO=%ECF_TRYNO%  # Current try number of the task
export ECF_RID=$$             # record the process id. Also used for zombie detection
# export NO_ECF=1             # uncomment to run as a standalone task on the command line

# Define the path where to find ecflow_client
# make sure client and server use the *same* version.
# Important when there are multiple versions of ecFlow
#export PATH=/usr/local/apps/ecflow/%ECF_VERSION%/bin:$PATH
export PATH=/home/mjisan/.conda/envs/myecflow/bin:$PATH

# Tell ecFlow we have started
ecflow_client --init=$$


# Define a error handler
ERROR() {
   set +e                      # Clear -e flag, so we don't fail
   wait                        # wait for background process to stop
   ecflow_client --abort=trap  # Notify ecFlow that something went wrong, using 'trap' as the reason
   trap 0                      # Remove the trap
   exit 0                      # End the script cleanly, server monitors child, an exit 1, will cause another abort and zombie
}


# Trap any calls to exit and errors caught by the -e flag
trap ERROR 0


# Trap any signal that may cause the script to fail
trap '{ echo "Killed by a signal"; ERROR ; }' 1 2 3 4 5 6 7 8 10 12 13 15

### 4. tail.h file

wait                      # wait for background process to stop
ecflow_client --complete  # Notify ecFlow of a normal end
trap 0                    # Remove all traps
exit 0                    # End the shell


### 5. ecflow file (needs to be in working directory, i .e. /home/mjisan/course/preprocess)

create_param_nml.ecf

%include <head.h>
echo "Generating namelist parameter"
cat << EOF > %ECF_HOME%/param.nml
&NML_PARAM
  dt = 150
  rnday = 2
/
EOF
%include <tail.h>

### 6. Running the namelist_suite 

a. create .def file
   python create_namelist_def.py

b. Load the suite definition into the ecFlow server:
   ecflow_client --load=namelist.def

c. ecflow_client --load=namelist.def
   ecflow_client --begin=namelist_suite

d. Start the suite:
   ecflow_client --get_state=/namelist_suite

e. Monitor the suite:
ecflow_client --get_state=/namelist_suite

GUI interface
![2024-08-08 08_53_49-ecFlowUI (5 13 3) - (menu_ admin)@hercules-login-2 hpc msstate edu](https://github.com/user-attachments/assets/34903685-d903-40d2-a396-e41fcab5b2bc)



# ecFLOW  with PySCHISM

ecflow_client --port 3141 --load=/home/mjisan/pyschism_suite.def
ecflow_client --port 3141 --begin=pyschism_suite
    task gen_manning
      edit ECF_SCRIPT '/home/mjisan/course/preprocess/gen_manning.ecf'
    task gen_bctides
      edit ECF_SCRIPT '/home/mjisan/course/preprocess/gen_bctides.ecf'
    task gen_sflux_era5
      edit ECF_SCRIPT '/home/mjisan/course/preprocess/gen_sflux_era5.ecf'


### 7. ecFLOW port change

ecflow_start.sh -p 3141


## ecFlow based UFS-coastal

### Definition file

(myecflow) [mjisan@hercules-01-06 ~]$ cat ufs_coastal_suite.def
suite ufs_coastal_suite
  edit ECF_HOME '/home/mjisan/workflow'
  edit ECF_INCLUDE '/home/mjisan/workflow'
  edit WAVE_COUPLING 'ON'  # Can be 'OFF' or 'ON'
  edit UFS_CLUSTER 'hercules'  # Adjust as needed
  edit RUN_TYPE 'coastal_ike_shinnecock'  # Can be customized for different run types

  family compile
    task compile_model
      edit ECF_SCRIPT '%ECF_HOME%/ecf/compile_model.ecf'
      edit APP 'CSTLS'
      edit USE_ATMOS 'ON'
      edit NO_PARMETIS 'OFF'
      edit OLDIO 'ON'
      edit BUILD_UTILS 'ON'
    task compile_model_with_waves
      edit ECF_SCRIPT '%ECF_HOME%/ecf/compile_model_with_waves.ecf'
      edit APP 'CSTLSW'
      edit USE_ATMOS 'ON'
      edit USE_WW3 'ON'
      edit NO_PARMETIS 'OFF'
      edit OLDIO 'ON'
      edit PDLIB 'ON'
      edit BUILD_UTILS 'ON'
  endfamily


### load definition suite

ecflow_client --port 3141 --load=/home/mjisan/ufs_coastal_suite.def

### alter the variable 

ecflow_client --port 3141 --alter change variable WAVE_COUPLING ON /ufs_coastal_suite
ecflow_client --port 3141 --alter change variable WAVE_COUPLING OFF /ufs_coastal_suite

### begin the task
### new suite with wave modules

suite ufs_coastal_suite
  edit ECF_HOME '/home/mjisan/workflow'
  edit ECF_INCLUDE '/home/mjisan/workflow'
  edit UFS_CLUSTER 'hercules'  # Adjust as needed
  edit RUN_TYPE 'coastal_ike_shinnecock'  # Can be customized for different run types

  family compile
    task compile_model
      event wave_coupling_off  # Event defined at task level
      trigger compile_model:wave_coupling_off == set  # Trigger when wave_coupling_off is set
      edit ECF_SCRIPT '%ECF_HOME%/ecf/compile_model.ecf'
      edit APP 'CSTLS'
      edit USE_ATMOS 'ON'
      edit NO_PARMETIS 'OFF'
      edit OLDIO 'ON'
      edit BUILD_UTILS 'ON'

    task compile_model_with_waves
      event wave_coupling_on  # Event defined at task level
      trigger compile_model_with_waves:wave_coupling_on == set  # Trigger when wave_coupling_on is set
      edit ECF_SCRIPT '%ECF_HOME%/ecf/compile_model_with_waves.ecf'
      edit APP 'CSTLSW'
      edit USE_ATMOS 'ON'
      edit USE_WW3 'ON'
      edit NO_PARMETIS 'OFF'
      edit OLDIO 'ON'
      edit PDLIB 'ON'
      edit BUILD_UTILS 'ON'
  endfamily
endsuite



ecflow_client --port 3141 --begin /ufs_coastal_suite

### force delete
ecflow_client --delete force /ufs_coastal_suite

### updated

ecflow_client --port 3141 --load=/home/mjisan/ufs_coastal_suite.def

ecflow_client --port 3141 --begin /ufs_coastal_suite

ecflow_client --port 3141 --force=set /ufs_coastal_suite/compile/compile_model:wave_coupling_off
ecflow_client --port 3141 --force=set /ufs_coastal_suite/compile/compile_model_with_waves:wave_coupling_on


