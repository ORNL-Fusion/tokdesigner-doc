============
Introduction
============

If you are new to IPS-FASTRAN, see first `Quick start guide for IPS-FASTRAN 
<https://github.com/ORNL-Fusion/tokdesigner-doc/tree/main/docs/under_construction.rst>`_.

In this tutorial, we will use the TokDesigner base template `tokdesigner.config 
<https://github.com/ORNL-Fusion/tokdesigner-doc/tree/main/docs/under_construction.rst>`_. 
or the IPS-FASTRAN base template `fastran_scenario.config 
<https://github.com/ORNL-Fusion/tokdesigner-doc/tree/main/docs/under_construction.rst>`_. 
A few modifications to these templates (basically turn on/off necessary components or change constraints/parameters) will be made at each step as needed. 

-----------------------------------
Problem setup for quick start guide
-----------------------------------

CAT-like reactor

* R = 4 m, **A** = 3, 𝜅 = 2, 𝛿 = 0.6
* BT = 7 T, Ip = 8.1 MA
* P = 38 MW

Determine **A** in the range of 2.5 ~ 3.5 for P_net > 50 MW

---------
Overviews
---------

:doc:`Step 1. Equilibrium at the reference point<step1>`

:doc:`Step 2. Scan and build a database<step2>`

:doc:`Step 3. Scan with random sampling<random>`

:doc:`Step 4. Generate reduced model<reduce>`

:doc:`Step 5. Filtering<filtering>`

:doc:`Step 6. Add low fidelity engineering modeling<external0>`

:doc:`Step 7. Constrained optimization<optimize>`

:doc:`Step 8. Add full phyiscs model: H/CD<heating>`

:doc:`Step 9. Add full phyiscs model: TGLF+EPED<physics>`

:doc:`Step 10. Add medium fidelity engineering modeling<external1>`



