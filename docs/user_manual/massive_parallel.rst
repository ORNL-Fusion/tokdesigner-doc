================
Massive Parallel
================

The section shows how to setup a large ensemble simulation using the Massive Parallel workflow with the :doc:`the random sampling example </quick_start_guide/scan_random>`.   

Template 
--------

`example6 <https://github.com/ORNL-Fusion/tokdesigner-doc/tree/main/examples/example1>`_

Prepare a scan
---------------

**Command line**

.. code-block:: bash

    sample.py --input=sample.json --nsample=1024 --inscan

This generates the input file *inscan* for the Massive Serial workflow. The number of sample points is specified by the ``--nsample`` option.

**Job submission file** (CORI)

.. code-block:: bash

  #!/bin/bash -l
  #SBATCH --image=registry.services.nersc.gov/parkjm/ips-fastran:latest
  #SBATCH --volume="/global/cscratch1/sd/parkjm/tmpfiles:/tmp:perNodeCache=size=10G"
  #SBATCH -p regular
  #SBATCH -N 4
  #SBATCH -t 00:30:00
  #SBATCH -J ips_fastran
  #SBATCH -e ips.err
  #SBATCH -o ips.out
  #SBATCH -C haswell
  
  module load python
  source activate /global/common/software/atom/cori/cesol_conda/latest
  
  ips.py \
  --config=ips_massive_parallel.config \
  --platform=cori_haswell.conf \
  --log=ips.log

This example uses 4 nodes (each node has 32 cores, total 128 cores) and executes **128** IPS-FASTRAN runs simultaneously. As soon as any IPS-FASTRAN run finished, a new IPS-FASTRAN instance in the task pool will be launched until the total 1024 simulations are completed.

.. code-block:: bash

  #SBATCH --image=registry.services.nersc.gov/parkjm/ips-fastran:latest

The Massive Serial workflow uses the IPS-FASTRAN/CESOL Docker image, which can be specified by the ``--image`` option.

.. code-block:: bash

  #SBATCH --volume="/global/cscratch1/sd/parkjm/tmpfiles:/tmp:perNodeCache=size=10G"

It is important for the large ensemble simulation to use the temporary file system (XFS) to minimize overhead to the file system. 
The `/tmp` directory in the computatio node is mounted to the user's ``$SCRATCH`` directory as specified by the option ``--volume=<your $SCRATH>/tmpfiles:/tmp:perNodeCache=size=10G``

**Configuration file for Massive Parallel**

.. code-block:: bash

  # ======================================================================
  # TEMPLATE FOR IPS_MASSIVE_SERIAL
  
  # ======================================================================
  # SIMULATION INFO SECTION
  # ======================================================================
  
  RUN_ID = ips_massive_parallel
  
  SIM_NAME = fastran_scan
  
  OUTPUT_PREFIX =
  LOG_FILE = ${RUN_ID}.log
  LOG_LEVEL = DEBUG
  
  SIM_ROOT = ${PWD}/RUN
  INPUT_DIR_SIM = ${PWD}
  
  RUN_COMMENT = ips-fastran scan with massive parallel
  TAG =
  
  SIMULATION_MODE = NORMAL
  
  USE_PORTAL = True
  PORTAL_URL = http://lb.ipsportal.production.svc.spin.nersc.org
  
  # ======================================================================
  # PLASMA STATE SECTION
  # ======================================================================
  
  STATE_WORK_DIR = ${SIM_ROOT}/work/plasma_state
  STATE_FILES =
  
  # ======================================================================
  # PORTS SECTION
  # ======================================================================
  
  [PORTS]
      NAMES = INIT DRIVER
      POSTS =
  
      [[INIT]]
          IMPLEMENTATION = dummy_init
  
      [[DRIVER]]
          IMPLEMENTATION = ips_massive_parallel
  
  # ======================================================================
  # COMPONENT CONFIGURATION SECTION
  # ======================================================================
  
  [dummy_init]
      CLASS = fastran
      SUB_CLASS =
      NAME = dummy_init
      MODULE = fastran.dummy.dummy_init
      SCRIPT =
      BIN_PATH =
      BIN =
      NPROC =
      INPUT_DIR =
      INPUT_FILES =
      OUTPUT_FILES =
      RESTART_FILES =
  
  [ips_massive_parallel]
      CLASS = fastran
      SUB_CLASS =
      NAME = ips_massive_parallel
      MODULE = fastran.driver.ips_massive_parallel
      SCRIPT =
      BIN_PATH =
      BIN =
      NPROC =
      DASK_NODES = ${SLURM_NNODES}
      INPUT_DIR = $INPUT_DIR_SIM
      SIMULATION = fastran_scenario.config
      INSCAN = inscan
      INPUT_FILES = ${SIMULATION} $INSCAN
      SUMMARY = ${PWD}/SUMMARY
      ARCHIVE_FILES =
      OUTPUT_FILES =
      RESTART_FILES =
      CLEAN_AFTER = 0
      TIME_OUT = 3600
      TMPXFS = /tmp
      TASK_PPN = 32 
      TASK_NPROC = 1 


This launch the IPS-FASTRAN workflow with the simulation configuration file ``SIMULATION = fastran_scenario.config``.

``TASK_NPROC`` is the maximum number of cores to be used in each IPS-FASTRAN simulation (1 core). 
``TASK_PPN`` is the number of concurrent runs on each node (32 tasks). 

.. note::

   More cores can be assigned to each IPS-FASTRAN simulation, for example ``TASK_NPROC = 32`` and ``TASK_PPN = 1`` (single concurrent IPS-FASTRAN simulation per node) to speed up the FASTRAN solver component with TGLF and the NUBEAM component with more Monte-Carlo particles. Over subscription (for example,  ``TASK_NPROC = 8`` and ``TASK_PPN = 6``) often improves the overall load balancing if the IPS-FASTRAN includes many serial code components such as TORAY, EFIT etc.

Collect output files
--------------------

The `/tmp` directory is not accessible after the simulation finishes. The output files will be copied to the directory specified in the Massive Serial configuration file (``SUMMARY = ${PWD}/SUMMARY``) as a compressed tar file (``*.tar.gz``), which in default contains the FASTRAN netcdf output (``f*.*``), the EFIT geqdsk (``g*.*``), and the instate file (``i*.*``).

Additional files to be archived can be also specified. For example, the nubeam output files:

.. code-block:: bash

  ARCHIVE_FILES = run?????/simulation_results/fastran_nb_nubeam*/*
