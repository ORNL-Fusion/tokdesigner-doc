Generate sample points
======================

**Command example**

.. code-block:: bash

   python $TOKDESIGNER/scan/fastran/sample.py --input=sample.json --output=inscan --nscan=1024

**Arguments**

* ``--input`` : input json file name (defalut="sample.json")
* ``--output`` : output inscan file name (default="inscan")
* ``--nscan`` : number of sample points (default=24)

**Input example** (``sample.json``)

- Define scan variables/range, constant variables, and maps to the ips-fastran/cesol variables

.. literalinclude:: sample.json
   :language: json

**Input - Line by Line**

Scan variable in the ``scan`` section

::

"r": [1.0, 2.5] # ramjor radius = random sampleing of r in 1.0 <= r <= 2.5

Constant variable in the ``constant`` section

::

"te_sep": 0.075 # temperature at separatrix = 0.075 keV

Filtering variable in the ``filter`` section

::

"fbs": [0, 1.0] # sampling include points only in 0 <= fbs <= 1

Derived variable in the ``model`` section

::

"pped": {"type":"loglinear", "file":"eped_shpd_ver0.fit"}

The pedestal pressure *pped* is calculated by the reduced model defined by *eped_shpd_ver0.fit* using the *loglinear* model.
