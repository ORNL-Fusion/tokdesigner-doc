Run IPS-FASTRAN/CESOL + engineering components
==============================================

This step excutes a large set of simulations for the sample points generated in :doc:`Step 1<sample>`. This utilizes the massive-serial workflow of IPS-FASTARN/CESOL with the container technology to minimize burden to the file system of NERSC.  

**Files**

#. ``tokdesigner.config`` : define TokDesigner simulation
#. ``cori_haswell_node.conf``: define cori node configuration 
#. ``input/inXXXX`` : input file for component XXXX, for example ``infastran`` is input file for the FASTRAN transport solver
#. ``inscan`` : sample points generated in :doc:`Step 1<sample>`

#. ``ips_massive_serial.config`` : define massive serial workflow
#. ``submitjob.cori`` : SLURM job submission file

**Submit job example:**  

.. literalinclude:: submitjob.cori 
   :language: bash

