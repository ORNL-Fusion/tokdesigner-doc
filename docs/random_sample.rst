=========================
Scan with random sampling
=========================

Reactor design involves an optimization in multi-dimensional parameter space. The "Curse of Dimensionality" makes the grid scan impractical for very high dimensional problem. This section illustrates how to generate a scan with random sampling. More advanced topic with `adaptive sampling <https://github.com/ORNL-Fusion/tokdesigner-doc/tree/main/docs/under_construction.rst>`_ will be discussed later.

Template
--------

`example2 <https://github.com/ORNL-Fusion/tokdesigner-doc/tree/main/examples/example1>`_

This example samples the input parameters in 6-dimensional space: aspect ratio ``aratio``, injection power ``pinj``, Greenwald density fraction at pedestal ``fgw_ped``, density peaking factor ``nepeak``, pedestal betaN ``betan_ped``, and H98 ``h98``.

.. note::

  One should argue that, for example, the toroidal field ``bt`` needs to be included in the scan in a way that lower ``aratio`` allows higher ``bt`` under the constraint of the maximum field at the TF coil. Note that this example is built for the purpose of workflow illustration. 
  

Prepare a scan
---------------

**Command line**

.. code-block:: bash

    sample.py --input=sample.json --nsample=128 --nsim=32

This will generate the input file for DAKOTA (dakota.in) and massive serial (inscan). The option ``--nsim`` specifies the number of concurrent runs for DAKOTA. The number of sample points is specified by ``--nsample``. 

.. note::

    The number of sample points in this example (128) is for illustration purpose, not likely enough to capture the parametric dependency. Suggested typical sampling is a few hundreds points for each dimension, for example 6x100 ~ 6x300 sample points.  

**sample.json**

.. code-block:: python

  {
    "scan": {
      "aratio": {"type": "range", "ymin":2.5, "ymax":3.5},
      "pinj"  : {"type": "range", "ymin":20.0, "ymax":50.0},
      "fgw_ped"  : {"type": "range", "ymin":0.7, "ymax":1.0},
      "nepeak"  : {"type": "range", "ymin":1.5, "ymax":2.0},
      "betan_ped"  : {"type": "range", "ymin":0.8, "ymax":1.3},
      "h98"  : {"type": "range", "ymin":0.9, "ymax":1.5}
    },
  
    "const": {
      "a"       : 1.3333,
      "kappa"   : 2.0,
      "delta"   : 0.6,
  
      "bt"      : 4.0,
      "ip"      : 8.1, 
  
      "xwid"    : 0.08,
      "xmid"    : 0.96,
  
      "f_nesep" : 0.5,
  
      "f_pinj_e": 0.75
   },

   ...
  
  }


The format of ``sample.json`` is same to the ``grid.json`` in the previous section, except addtional section ``filter``. Note that some of the variables in the ``const`` section moved to the ``scan`` section. See the `filter <https://github.com/ORNL-Fusion/tokdesigner-doc/tree/main/docs/under_construction.rst>`_ section description for details.

Build a database
----------------

**Command line**

.. code-block:: bash

    makedb.py --input=makedb.json --output=db.dat --rdir=SUMMARY

**makedb.json**

A few more variables are added starting from ``makedb.json`` of the previous section. The net electricity ``pnet`` uses a simple TokDesigner model [1], where the power conversion efficiency ``eta_th`` and ``eta_cd`` required for the ``pnet`` *base* model are defined in the ``const`` section.

.. code-block:: python

  {
    "variable": {
        "r"          : ["instate" , "r0"        , "input" ],
        "a"          : ["instate" , "a0"        , "input" ],
        "bt"         : ["instate" , "b0"        , "input" ],
        "ip"         : ["instate" , "ip"        , "input" ],
        "q95"        : ["aeqdsk"  , "q95"       , "output"],
        "li"         : ["aeqdsk"  , "li"        , "output"],
        "betap"      : ["aeqdsk"  , "betap"     , "output"],
        "betat"      : ["aeqdsk"  , "betat"     , "output"],
        "betan_ped"  : ["instate" , "betan_ped" , "input" ],
        "ne_ped"     : ["instate" , "ne_ped"    , "input" ],
        "ne_axis"    : ["instate" , "ne_axis"   , "input" ],
        "nebar"      : ["fastran" , "nebar"     , "output"],
        "h98"        : ["instate" , "h98_target", "input" ],
        "betan"      : ["fastran" , "betan"     , "output"],
        "pfuse"      : ["fastran" , "pfuse"     , "output"],
        "pfusi"      : ["fastran" , "pfusi"     , "output"],
        "prfe"       : ["fastran" , "prfe"      , "output"],
        "prfi"       : ["fastran" , "prfi"      , "output"],
        "pnbe"       : ["fastran" , "pnbe"      , "output"],
        "pnbi"       : ["fastran" , "pnbi"      , "output"],
        "ibs"        : ["fastran" , "ibs"       , "output"]
    },
    "const": {
        "eta_cd"     : 0.25,
        "eta_th"     : 0.33
    },
    "model": {
        "aratio"     : ["expr", "r / a" ],
        "pfus"       : ["expr", "5.0 * ( pfuse + pfusi )"],
        "ngw"        : ["base", {}],
        "fgw_ped"    : ["expr", "ne_ped / ngw"],
        "nepeak"     : ["expr", "ne_axis / ne_ped"],
        "fbs"        : ["expr", "ibs / ip"],
        "pinj"       : ["expr", "pnbe + pnbi + prfe + prfi"],
        "pnet"       : ["base", {}]
    }
  }

Filtering
---------

Filtering can be applied to identify design candidates using the database generated in this section. This is a "Direct filtering" workflow. See TokDesinger Basic. 

.. note::

  In most case, the filtering and constrainted optimization employs the reduced model approach (see Step 5 and Step 7). 

**Command line**

.. code-block:: bash

    filter.py --input=filter.json --dbfile=db.dat --output=filter.dat


**filter.json**

.. code-block:: python

 {
   "filter": {
     "fbs"  : ["min", 0.8],
     "fbs"  : ["max", 1.0],
     "pnet" : ["min", 50.0]
   }
 }

This is an example for the bootstrap current ``0.8 <= fbs <= 1.0`` and the net electricity ``pnet >= 50 MW``. 
The output database file contains the filtered points satisfying these constraints.

