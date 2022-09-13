=================================
Run Environement on CORI at NERSC
=================================

The TokDesigner conda environment is maintained for the *atom* project created using the *shareable environment* method. You can activate it and run TokDesigner by:

.. code-block:: bash

  module load python
  source activate /global/common/software/atom/cori/cesol_conda/v0.11b

**Command example**

.. code-block:: bash

  python $TOKDESIGNER/scan/fastran/sample.py --input=sample.json --output=inscan --nscan=1024
  python $TOKDESIGNER/data/fastran/makedb.py --input=makedb.json --output=db.dat --rdir=summary

**Command type**

``sample.py`` : sampling

``makedb.py`` : generate database

``fitdb.py`` : ML to generate reduced models

``filter.py`` : filtering design points

``optimizer.py`` : constrained optimizer
