=========================
Scan and build a database
=========================

This section  illustrates how to 1) prepare a scan, 2) submit a job with DAKOTA or massive serial, 3) collect output files, and 4) build a database.
We will use a 2D scan example - aspect ratio and heating power. 

Template 
--------

`example1 <https://github.com/ORNL-Fusion/tokdesigner-doc/tree/main/examples/example1>`_

Prepare a scan
---------------

**Command line**

.. code-block:: bash

    grid.py --input=grid.json --nsim=32

This will generate the input file for DAKOTA (dakota.in) and massive serial (inscan).
The option ``--nsim`` specifies the number of cocurrent runs for DAKOTA.

**grid.json**

.. code-block:: python

  {
    "scan": {
      "aratio": {"type": "range", "ymin":2.5, "ymax":3.5, "ndata":8},
      "pinj"  : {"type": "list", "data":[20.0, 30.0, 40.0, 50.0]}
    },

    "const": {
      "a"       : 1.3333,
      "kappa"   : 2.0,
      "delta"   : 0.6,

      "bt"      : 4.0,
      "ip"      : 8.1,

      "xwid"    : 0.08,
      "xmid"    : 0.96,

      "nepeak"  : 1.75,
      "fgw_ped" : 0.9,
      "f_nesep" : 0.5,

      "h98"     : 1.35,

      "f_pinj_e": 0.75
   },

   "model": {
      "r"           : ["expr", "a * aratio"                 ],
      "betan_ped"   : ["base", {}                           ],
      "te_ped"      : ["base", {"dependency":"betan_ped"}   ],
      "ti_ped"      : ["expr", "te_ped"                     ],
      "te_axis"     : ["expr", "5.75 * te_ped"              ],
      "ti_axis"     : ["expr", "5.0 * ti_ped"               ],
      "ngw"         : ["base", {}                           ],
      "ne_ped"      : ["expr", "fgw_ped * ngw"              ],
      "ne_axis"     : ["expr", "nepeak * ne_ped"            ],
      "ne_sep"      : ["expr", "f_nesep * ne_ped"           ],
      "pinj_e"      : ["expr", "pinj * f_pinj_e"            ],
      "pinj_i"      : ["expr", "pinj * ( 1.0 - f_pinj_e ) " ]
    },

   "io": {
      "index"       : ["efit"        , "TIME_ID"    ],

      "r"           : ["fastran_init", "R0"         ],
      "a"           : ["fastran_init", "A0"         ],
      "kappa"       : ["fastran_init", "KAPPA"      ],
      "delta"       : ["fastran_init", "DELTA"      ],

      "bt"          : ["fastran_init", "B0"         ],
      "ip"          : ["fastran_init", "IP"         ],

      "xwid"        : ["fastran_init", "XWID"       ],
      "xmid"        : ["fastran_init", "XMID"       ],

      "ne_axis"     : ["fastran_init", "NE_AXIS"    ],
      "ne_ped"      : ["fastran_init", "NE_PED"     ],
      "ne_sep"      : ["fastran_init", "NE_SEP"     ],

      "te_axis"     : ["fastran_init", "TE_AXIS"    ],
      "te_ped"      : ["fastran_init", "TE_PED"     ],

      "ti_axis"     : ["fastran_init", "TI_AXIS"    ],
      "ti_ped"      : ["fastran_init", "TI_PED"     ],

      "h98"         : ["fastran_init", "H98_TARGET" ],

      "pinj_e"      : ["hcd_model"   , "INHCD_PE_0" ],
      "pinj_i"      : ["hcd_model"   , "INHCD_PI_0" ]
    }
  }

The TokDesigner variables consist of the variables defined in the ``scan``, ``const``, and ``model`` sections. 

**Scan section**

* ``aratio`` (aspect ratio) : ndata points (8) between the minimum value (2.5) and the maximum value (3.5)
* ``pinj`` (injection power, MW) : 4 points are given by the user input list [20.0, 30.0, 40.0, 50.0]
* Number of total scan =  8 * 4 = 32

**Const section**

The ``const`` section defines the constant TokDesigner variables. 

**Model section**

The variables in the model section are determined by the predefined models or user defined formula. 
For instance, the major radius ``r`` has the type ``expr`` (expression type), so will be specified by the expression ``r = aratio * a``. 
Note that we defined ``aratio`` as a scan variable and ``a`` as a constant variable to calculate ``r``.

----

Exercise: aspect ratio scan with the fixed major radius ``r``

.. code-block:: python

  {
    "scan": {
      "aratio": {"type": "range", "ymin":2.5, "ymax":3.5, "ndata":8},
      ...
    }
    "constant" {
      "r: 4.0,
      ...
    }
    "model": {
      "a": ["expr", "r / aratio"],
      ...
      }
    ...
  }

----

TokDesigner provides a range of simplified **models**. In this example, the pedestal betan ``betan_ped`` is a ``base`` model variable, where ``betan_ped`` is calculated by the TokDesigner model (Phil Snyder's simple triangularity ``delta`` scaling):

.. code-block:: python

  import numpy as np
  from kernel.base.tokamak_parameter import TokamakParameter

  # normalized beta at pedestal
  class betan_ped_model(TokamakParameter):
      def __init__(self, model='base'):
          TokamakParameter.__init__(self, model)
          self.dependency = ["delta"]

      def base_model(self, ps):
          delta =  ps["delta"]
          betanped = 0.2 + 1.3*delta
          return betanped

Then, the pedestal electron temperature ``te_ped`` is calculated by the ``base`` model, which is a simple conversion from ``betan_ped`` to ``te_ped``. Instead, users may use the euivalent ``expr``:

.. code-block:: python

  "model": {
      ...
      "te_ped": ["expr", "1.2424 * ip * bt / ( ne_ped * a )"],
      ...
  }

This example assumes that the ion pedestal temperature ``ti_ped`` is same to ``te_ped``.

.. code-block:: python

   "model": {
      ...
      "ti_ped" : ["expr", "te_ped"],
      ...
   }

.. note::

   The *model* is also outcome of the TokDeisgner workflows, which can be used to generate the next scan. This iterative and recursive process is one of the key concepts of TokDesigner.

**Io section**

This defines a map between the TokDesigner variables and the IPS-FASTRAN/CESOL simulation variables. 
For example, the ``R0`` value of the instate file in the fastran_init component will be updated by the TokDesigner Variable ``r``. 

.. code-block:: python

    "r" : ["fastran_init", "R0"],

See more details :ref:ingrid_io_reference.

Job submission on CORI
----------------------

This example ``submitjob.ex2`` uses a DAKOTA scan with the scan file ``dakota.in`` generated by ``grid.py``. Single CORI node (32 cores) is allocated, which will be shared by 32 single core IPS-FASTRAN runs simultaneously. 

.. code-block:: bash

  #!/bin/bash -l
  #SBATCH -p debug
  #SBATCH -N 1
  #SBATCH -t 00:10:00
  #SBATCH -J ips_fastran
  #SBATCH -e ips.err
  #SBATCH -o ips.out
  #SBATCH -C haswell

  module load python
  source activate /global/common/software/atom/cori/cesol_conda/t0.14b

  export SHOT_NUMBER=000001
  export TIME_ID=00001

  ips_dakota_dynamic.py --dakotaconfig=dakota.in --simulation=fastran_scenario.config --platform=cori_haswell_node.conf --log=ips_sweep.log

  conda deactivate

See also `how to use a massive serial for a larger ensemble of simulation <https://github.com/ORNL-Fusion/tokdesigner-doc/tree/main/docs/under_construction.rst>`_. 

Collect output files
--------------------

**Command line**

.. code-block:: bash

  collect.py --rdir0=. --rdir=SCAN --sdir=SUMMARY --input=collect.json

This will collect output files in the simulation directory ``SCAN`` into the summary directory ``SUMMARY``. 

See `convention of the directory structure and how to modify the run directory name <https://github.com/ORNL-Fusion/tokdesigner-doc/tree/main/docs/under_construction.rst>`_.

**collect.json**

.. code-block:: python

  {
     "output" : [
         ["fastran_tr0_fastran", "fastran.nc"   , "f", "result" ],
         ["fastran_eq0_efit",    "g??????.?????", "g", "result" ],
         ["fastran_eq0_efit",    "a??????.?????", "a", "result" ],
         ["fastran_eq0_efit",    "s??????.?????", "s", "result" ],
         ["fastran_tr0_fastran", "i??????.?????", "i", "result" ]
     ],
     "input" : [
         "fastran_scenario.config"
     ]
  }


The ``collect.json`` defines which output files to archive. For example, the ``fastran0`` solver component (``[fastran0]``), which is identified as ``fastran_tr0_fastran`` (``CLASS`` + ``_`` + ``SUB_CLASS`` + ``_`` + ``NAME``), archives ``fastran.nc`` with the name: ``f`` + ``SHOT_NUMBER`` + ``.`` + ``TIME_ID`` (like f123456.00001), where the ``TIME_ID`` is the identifier in ``dakota.in`` or ``inscan``.  

See the ``fastran0`` componenet definition in the ``fastran_scenario.config``.

.. code-block:: bash

  [fastran0]
      CLASS = fastran
      SUB_CLASS = tr0
      NAME = fastran
      ...
      OUTPUT_FILES = fastran.nc xfastran.log ${CURRENT_INSTATE}
      ....
      
Build a database
----------------

**Command line**

.. code-block:: bash

    makedb.py --input=makedb.json --output=db.dat --rdir=SUMMARY

This will generate a database file ``db.dat`` using the IPS-FASTRAN/CESOL output files archived in the ``SUMMARY`` directory (note ``collect.py .. --sdir=SUMMARY ..``)

**makedb.json**

The ``makedb.json`` defines the database variables. For the illustration purpose, only a few variables are included - r, a, aratio, bt, ip, pinj, q95, betan, fbs, li, tau98, tauth, h98, pfus.

.. code-block:: python

  {
    "variable": {
        "r"          : ["instate" , "r0"       , "input" ],
        "a"          : ["instate" , "a0"       , "input" ],
        "bt"         : ["instate" , "b0"       , "input" ],
        "ip"         : ["instate" , "ip"       , "input" ],
        "q95"        : ["aeqdsk"  , "q95"      , "output"],
        "li"         : ["aeqdsk"  , "li"       , "output"],
        "betap"      : ["aeqdsk"  , "betap"    , "output"],
        "betat"      : ["aeqdsk"  , "betat"    , "output"],
        "ne_ped"     : ["instate" , "ne_ped"   , "input" ],
        "nebar"      : ["fastran" , "nebar"    , "output"],
        "betan"      : ["fastran" , "betan"    , "output"],
        "pfuse"      : ["fastran" , "pfuse"    , "output"],
        "pfusi"      : ["fastran" , "pfusi"    , "output"],
        "ibs"        : ["fastran" , "ibs"      , "output"]
    },
    "model": {
        "aratio"     : ["expr", "r / a" ],
        "pfus"       : ["expr", "5.0 * ( pfuse + pfusi )"],
        "fbs"        : ["expr", "ibs / ip"]
    }
  }

* The major radius ``r`` is read from the ``instate`` file (``i<shot_number>.<time_id>``)
* The q95 value ``q95`` is read from the EFIT "aeqdsk`` file (``a<shot_number>.<time_id>``)
* The alpha fusion power to electron ``pfuse`` is read from the ``fastran`` output netcdf file (``f<shot_number>.<time_id>``)

Note that the ``model`` method can be used as in the ``grid.py``.

* The aspect ratio ``araio = r / a``
* The bootstrap current ``fbs = ibs / ip``

See the `table <https://github.com/ORNL-Fusion/tokdesigner-doc/tree/main/docs/under_construction.rst>`_ for the typical TokDesigner variables.







