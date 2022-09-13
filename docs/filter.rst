========================
Filtering design points
========================

**Command example**

.. code-block:: bash

   python $TOKDESIGNER/driver/filter.py --model=model.json --solver=solver.json --ndata=10000

**Arguments**

* ``--model`` : input file name for the model description  (defalut="model.json")
* ``--solver`` : input file name for the solver description (default="solver.json")
* ``--nscan`` : number of sample points (default=1000)
* ``--output`` : output file name (default="outdb.dat")

**Input example**: ``model.json``

- Describe the reduced models obtained from :doc:`Step 3<ml>`

.. literalinclude:: model.json
   :language: json

**Input example**: ``solver.json``

- Describe the solver: range of scan variables, unknowns, output variables, ... 

.. literalinclude:: solver.json
   :language: json


