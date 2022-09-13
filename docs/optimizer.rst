=====================
Constrained Optimizer
=====================

**Command example**

.. code-block:: bash

   python $TOKDESIGNER/driver/optimizer.py --model=model.json --solver=solver.json --constraint=constraint.json

**Arguments**

* ``--model`` : input file name for the model description  (defalut="model.json")
* ``--solver`` : input file name for the solver description (default="solver.json")
* ``--constraint`` : input file name for the constraint description (default="constraint.json")

**Input example**: ``constraint.json``

Minimize `pinj` (total injection power) with the constraints:

* `fbs` (bootstrap current fraction) > 0.8
* `pmhd` (average pressure) > 300 kPa
* `q_parallel` (parallel heat flux) > 50.
* `pressure_sol` (SOL pressure) > 15.0 kPa 

.. literalinclude:: constraint.json
   :language: json

