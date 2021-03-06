.. _usage-plugins-energyFields:

Energy Fields
-------------

This plugin computes the total energy contained in the electric and magnetic field of the entire volume simulated.
The energy is computed for user specified time steps.

.cfg file
^^^^^^^^^

By setting the PIConGPU command line flag ``--fields_energy.period`` to a non-zero value the plugin computes the total field energy. 
The default value is ``0``, meaning that the total field energy is not stored.
By setting e.g. ``--fields_energy.period 100`` the total field energy is computed for time steps *0, 100, 200, ...*.

Memory Complexity
^^^^^^^^^^^^^^^^^

Accelerator
"""""""""""

negligible.

Host
""""

negligible.

Output
^^^^^^

The data is stored in ``fields_energy.dat``.
There are two columns.
The first gives the time step.
The second is the total field energy in **Joule**.
The first row is a comment describing the columns:

.. code::

   #step total[Joule] Bx[Joule] By[Joule] Bz[Joule] Ex[Joule] Ey[Joule] Ez[Joule]
   0     2.5e+18      0         0         0         2.5e+18   0         0
   100   2.5e+18      2.45e-22  2.26e-08  2.24e-08  2.5e+18   2.29e-08  2.30e-08

.. attention::

   The output of this plugin computes a *sum over all cells* in a very naive implementation.
   This can lead to significant errors due to the finite precision in floating-point numbers.
   Do not expect the output to be precise to more than a few percent.
   Do not expect the output to be deterministic due to the statistical nature of the implemented reduce operation.

   Please see `this issue <https://github.com/ComputationalRadiationPhysics/picongpu/issues/523#issuecomment-70630415>`_ for a longer discussion and possible future implementations.

Example Visualization
^^^^^^^^^^^^^^^^^^^^^

Python example snippet:

.. code:: python

   import numpy as np
   import matplotlib.pyplot as plt


   simDir = "path/to/simOutput/"

   # Ekin in Joules (see EnergyParticles)
   e_sum_ene = np.loadtxt(simDir + "e_energy_all.dat")[:, 0:2]
   p_sum_ene = np.loadtxt(simDir + "p_energy_all.dat")[:, 0:2]
   C_sum_ene = np.loadtxt(simDir + "C_energy_all.dat")[:, 0:2]
   N_sum_ene = np.loadtxt(simDir + "N_energy_all.dat")[:, 0:2]
   # Etotal in Joules
   fields_sum_ene = np.loadtxt(simDir + "fields_energy.dat")[:, 0:2]

   plt.figure()
   plt.plot(e_sum_ene[:,0], e_sum_ene[:,1], label="e")
   plt.plot(p_sum_ene[:,0], p_sum_ene[:,1], label="p")
   plt.plot(C_sum_ene[:,0], C_sum_ene[:,1], label="C")
   plt.plot(N_sum_ene[:,0], N_sum_ene[:,1], label="N")
   plt.plot(fields_sum_ene[:,0], fields_sum_ene[:,1], label="fields")
   plt.plot(
       e_sum_ene[:,0],
       e_sum_ene[:,1] + p_sum_ene[:,1] + C_sum_ene[:,1] + N_sum_ene[:,1] + fields_sum_ene[:,1],
       label="sum"
   )
   plt.legend()
   plt.show()
