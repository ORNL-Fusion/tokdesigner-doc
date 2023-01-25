========================
Constrained optimization
========================

Starting from the model developed in the previous sections, this section illustrates how to use a constrained optimizer in the TokDesigner workflows.

Template
--------

`example6 <https://github.com/ORNL-Fusion/tokdesigner-doc/tree/main/examples/example6>`_

Optimization
------------

**Command line**

.. code-block:: bash

  optimize.py --input=optimize.json --output=solution.json

**optimize.json**

.. code-block:: python

  {
    "scan": {
      "aratio"    : {"type": "range", "ymin":2.5, "ymax":3.5},
      "fgw_ped"   : {"type": "range", "ymin":0.7, "ymax":1.0},
      "nepeak"    : {"type": "range", "ymin":1.5, "ymax":2.0},
  
      "d_blanket" : {"type": "range", "ymin":0.2, "ymax":0.4},
      "d_tf"      : {"type": "range", "ymin":0.4, "ymax":1.2},
      "d_cs"      : {"type": "range", "ymin":0.2, "ymax":0.6},
      "f_wp"      : {"type": "range", "ymin":0.5, "ymax":0.8},
      "f_r_tf_out": {"type": "range", "ymin":2.5, "ymax":4.5}
    },
  
    "const": {
      "a"           : 1.3333,
      "kappa"       : 2.0,
      "delta"       : 0.6,
      "bt"          : 7.0,
      "ip"          : 8.1,
  
      "pinj"        : 38.0,
      "h98"         : 1.4,
  
      "eta_cd"      : 0.25,
      "eta_th"      : 0.33,
  
      "d_sol"       : 0.1,
      "d_str"       : 0.1,
      "gap_vv"      : 0.05,
      "d_vv"        : 0.15,
      "gap_shield"  : 0.05,
      "d_shield"    : 0.225,
      "gap_tf"      : 0.083,
      "gap_cs"      : 0.08,
  
  
      "t_helium"    : 4.5,
      "strain_sc"   : -0.005,
      "f_sc_copper" : 0.69,
      "f_sc_helium" : 0.46,
      "f_sc_conductor": 0.28
    },
  
    "model": {
      "r"         : ["expr", "a * aratio"                 ],
      "betan_ped" : ["base", {}   ],
      "te_ped"    : ["base", {"dependency":"betan_ped"}   ],
      "ti_ped"    : ["expr", "te_ped"                     ],
      "ngw"       : ["base", {}                           ],
      "ne_ped"    : ["expr", "fgw_ped * ngw"              ],
      "ne_axis"   : ["expr", "nepeak * ne_ped"            ],
      "betan"     : ["file", "fitout.json" ],
      "pfus"      : ["file", "fitout.json" ],
      "fbs"       : ["file", "fitout.json" ],
      "pnet"      : ["base", {}],
  
      "d_wp"      : ["expr", "f_wp * d_tf"],
      "r_tf"      : ["expr", "r - ( a + d_sol + d_blanket + d_str + gap_vv + gap_shield + d_shield + gap_tf + d_tf )"],
      "r_cs"      : ["expr", "r_tf - ( gap_cs + d_cs )"],
      "r_tf_out"  : ["expr", "r + f_r_tf_out * a"],
  
      "bmax"      : ["base", {}],
      "itf"       : ["base", {}],
      "jtf"       : ["base", {}],
      "tf_stress_axial" : ["base", {}],
      "tf_stress_hoop"  : ["base", {}],
      "tf_stress" : ["expr", "tf_stress_axial + tf_stress_hoop"],
      "jsc_crit"  : ["base", {}],
      "jtf_crit"  : ["expr", "jsc_crit * f_sc_copper * f_sc_helium * f_sc_conductor" ],
      "f_jtf_crit": ["expr", "jtf / jtf_crit"]
    },
  
    "constraint": {
      "fbs": ["min", 0.8],
      "pnet": ["min", 50.0],
      "f_jtf_crit": ["max", 1.0],
      "tf_stress": ["max", 500.0]
  
    },
  
    "objective": ["aratio", "min"]
  }


Note that the ``constraint`` and ``objective`` sections are added to the ``evaluation.json`` in the previous section. This example finds a minimum aspect ratio ``aratio`` as requested in the ``objective`` section

.. code-block:: python

  "objective": ["aratio", "min"]


in the parameter space defined in the ``scan`` secation:

.. code-block:: python

  "scan": {
    "aratio"    : {"type": "range", "ymin":2.5, "ymax":3.5},
    "fgw_ped"   : {"type": "range", "ymin":0.7, "ymax":1.0},
    "nepeak"    : {"type": "range", "ymin":1.5, "ymax":2.0},

    "d_blanket" : {"type": "range", "ymin":0.2, "ymax":0.4},
    "d_tf"      : {"type": "range", "ymin":0.4, "ymax":1.2},
    "d_cs"      : {"type": "range", "ymin":0.2, "ymax":0.6},
    "f_wp"      : {"type": "range", "ymin":0.5, "ymax":0.8},
    "f_r_tf_out": {"type": "range", "ymin":2.5, "ymax":4.5}
  },


for the given ``pinj`` = 38 MW  and ``h98`` = 1.4 in the ``constant`` section

.. code-block:: python

  "const": {
     ...

    "pinj" : 38.0,
    "h98"  : 1.4,

    ...
  }

This is a constrained optimization that satisfies the contraints (``fbs`` > 0.8, ``pnet`` > 50 MW, ``tf_stress`` < 500 MPa, and ``jtf`` < ``jtf_crit``) defined in the ``constraint`` section.

.. code-block:: python

  "constraint": {
    "fbs": ["min", 0.8],
    "pnet": ["min", 50.0],
    "f_jtf_crit": ["max", 1.0],
    "tf_stress": ["max", 500.0]
  }

The solution can be found in the output file ``solution.json`` 

.. code-block:: python


  {
      "aratio": 3.411401491370441,
       ...
      "fbs": 0.8525004751239108,
      "pnet": 50.00000018126549,
       ...
      "tf_stress": 367.942266197282,
      "f_jtf_crit": 0.6737642561473158
  }

