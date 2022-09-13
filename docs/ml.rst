======================
Generate Reduced Model
======================

**Command example**

.. code-block:: bash

   python $TOKDESIGNER//data/base/preprocess.py --data=db.dat
   python $TOKDESIGNER/data/base/fitdb.py --train=train.dat --test=test.dat --input=fitdb.json

**Arguments**

* ``--data`` : database file name (defalut="db.dat")
* ``--train`` : database file name for training (default="train.dat")
* ``--test`` : database file name for validation (default="test.dat")
* ``--input`` : input file to define the ML model (defalut="fitdb.json")


**Input example** (``fitdb.json``)

Loglinear 

.. literalinclude:: fitdb_loglin.json
   :language: json

Neutral Network 

.. literalinclude:: fitdb_nn.json
   :language: json


