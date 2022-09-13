==============
Build database
==============

**Command example**

.. code-block:: bash

   python $TOKDESIGNER/data/fastran/makedb.py --input=makedb.json --output=db.dat --rdir=summary

**Arguments**

* ``--input`` : input json file name (defalut="makedb.json")
* ``--output`` : output databse file name (default="db.dat")
* ``--rdir`` : output directory of :doc:`Step 2<run>`  (default="summary")

**Example makedb.json**

.. note::

   Define variables to be collected from the simulations 

.. literalinclude:: makedb.json
   :language: json

**Input - Line by Line**

Collect *we* (electron stored energy) from the fastran component

::

   "we" : ["fastran", "we", "output"],



