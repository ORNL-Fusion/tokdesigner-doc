==================================
Equilibrium at the reference point
==================================

This step illustrates one-point reactor simulation, where we formulate key physics assumptions. We start with a *system-code-like model* but with more consistent/accurate calculation of the bootstrap current, fusion reaction etc, self-consistent with the exact MHD equilibrium and more realistic plasma profiles. Note that users can raise the fidelity of each component or add/remove components as neede, for instance, use TGLF for core transport and EPED for pedestal, rather than specify the constraint H98. 
`Later  <https://github.com/ORNL-Fusion/tokdesigner-doc/tree/main/docs/under_construction.rst>`_.
we will come back to use the full theory-based models.

.. note::

   The **component** and **constraint** are one of the most important concepts of TokDesigner, which will appear in many places of the TokDesigner workflows. 

Template 
--------

`example1 <https://github.com/ORNL-Fusion/tokdesigner-doc/tree/main/examples/example1>`_

Instate
-------

The major machine configuration is specified in the *instate* file: 

**Machine size/plasma shape**

.. code-block:: fortran

   R0 = 4.0 ! major radius [m]
   A0 = 1.33 ! minor radius, so aspect ratio = R0/A0 = 3.0
   KAPPA = 2.0 ! elongation
   DELTA = 0.6 ! triangularity

**Magnetic field/plasma current**

.. code-block:: fortran

   BT = 4.0 ! toroidal magentic field [T]
   IP = 8.1 ! Plasma current [MA]

**Plasma composition**

.. code-block:: fortran

  DENSITY_MODEL = 1 ! 1 = from impurity density

  N_ION = 2         ! number of main plasma ions
  Z_ION = 1         ! charge number of main plasma ions; array of N_ION; integer ; D + T
  A_ION = 2 3       ! mass number of main plasma ions; array of N_ION; integer; D + T
  F_ION = 0.5 0.5   ! fraction of each main ion; array of N_ION; D 50% + T 50%
  N_IMP = 3         ! number of impurities
  Z_IMP = 2 4 18    ! charge number of impurities; array of N_IMP; integer; He + Be + Ar
  A_IMP = 4 9 40    ! mass number of impurities; array of N_IMP; integer; He + Be + Ar
  F_IMP = 0.0 0.02 0.0005 ! (impurity denisty)/ne for each impurity species

**Global parameters**

A set of global parameters can be specified for example, ``betaN`` or ``H98`` with the prefix ``_TARGET``. 
The IPS-FASTARN constraint component will match the given target parameters.  

.. code-block:: fortran

   BETAN_TARGET = 3.5 ! btanN target
   H98_TARGET = 1.3 ! H98 input   

Note: If ``H98_TARGET > 0``, ``betaN`` is adjusted to match the given ``H98_TARGET``. 
If ``H98_TARGET < 0``, ``betaN`` is adjusted to match the given ``BETAN_TARGET``. 

IPS-FASTRAN workflow
--------------------

The IPS-FASTRAN model `fastran_scenario.conf <https://github.com/ORNL-Fusion/tokdesigner-doc/tree/main/docs/under_construction.rst>`_ in `example1 <https://github.com/ORNL-Fusion/tokdesigner-doc/tree/main/examples/example1>`_ uses ``EFIT``, ``FASTRAN solver`` , ``Model equilibrium constraint``,  and ``Model heating/current drive`` components, which iteratively finds a MHD equilibrium with given constraints.

.. code-block:: bash

   PORTS = INIT DRIVER EQ0 HCD0 TR0 BC0 

  [[TR0]]
      IMPLEMENTATION = fastran0

  [[EQ0]]
      IMPLEMENTATION = efit0

  [[HCD0]]
      IMPLEMENTATION = hcd_model

  [[BC0]]
      IMPLEMENTATION = modeleq_constraint

In this example, the ``modeleq_constraint`` component impose the H98 and broad current profile constraints.

.. code-block:: bash

  [modeleq_constraint]
      CLASS = fastran
      SUB_CLASS =
      NAME = modeleq_constraint
      MODULE = fastran.driver.modeleq_constraint
      ...

The heating profile is specified by the Gaussian profile in the ``hcd_model`` component

.. code-block:: bash

  [hcd_model]
      CLASS = fastran
      SUB_CLASS = ic
      NAME = hcd_model
      MODULE = fastran.heating.hcd_model
      ...
      INPUT_FILES = inhcd
      ...

, using the input file **inhcd**

.. code-block:: fortran

  &inhcd
    nsrc = 1  ! Number of H/CD systems
    Pe = 30.0 ! Electron power [MW]
    Pi = 8.0  ! Ion power [MW] 
    xmid = 0.0 ! Center of Gaussian heating profile 
    xwid = 0.3 ! Width of Gaussian heating profile

    j0_seed = 0.0 ! Peak current density [MA/m^2]
    x0_seed = 0.0 ! Center of Gaussian current profile 
    drho_seed = 0.3 ! Width of Gaussian current profile 
  /

Run
---

This simulation can be excuted on the login node of CORI using single core, taking only a couple of minuites. 
The bash script for run **submitjob.ex1**:

.. code-block:: bash

  #!/bin/bash -l
  module load python
  source activate /global/common/software/atom/cori/cesol_conda/t0.14b

  export SHOT_NUMBER=000001
  export TIME_ID=00001

  ips.py --simulation=fastran_scenario.config --platform=cori_haswell_node.conf --log=ips.log 1> ips.out 2> ips.err &

  conda deactivate
