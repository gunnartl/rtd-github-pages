
Setup CTSM
===========

Accounting
----------------

For running CTSM, you usually need to have access to a High-Performance Computer.

If you are working in Norway, you can use either saga, fram, or betzy (depending on demands on parallel computing) and need to be part of an active account:

- Notur

To check which project you can use:

:: 

  projects


This will return something like:

::

  nn2806k
  nn1000k

In the example above, two projects can be used (nn2806k and nn1000k). Then make sure you choose the right project when running CTSM.

 
 Recommended spin-up routine for CTSM
=====================================
Single cell example: Brazil


Why do we do a spin-up?
~~~~~~~~~~~~~~~~~~~~~~~~  
- The idea behind doing a so-called ***spin-up*** is to reach an equilibrium state in your model. 
- Typically any perturbation (e.g. 2x CO2) experiment starts from such a state. 
- Running our perturbed system into equilibrium again will give you the maximum impact of the perturbation.
  
Which variables are important to check?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
- Since we are interested in the equilibrium state of our system, the "slowest" variables matter the most (e.g. terrestrial soil carbon). 
- If you are running in BioGeoChemistry (BGC) mode, it is important to check the "health" of your vegetation. 
- The variable directly associated with the health is Total Leaf Area Index (TLAI). 
- If this is 0 your vegetation is dead and something went really wrong.
  
How long should a spin-up be?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
- The overall duration (years before an equilibrium state is reached) varies. 
- As mentioned above, the variables with the slowest turnover rate will define the time you'd need to spend for your spin-up. 
- The rule of thumb for single-cell simulation is the higher the latitude the longer the spin-up. 
- Spinning up a single cell case in the Tropics will approximately take 2x100 years after which the terrestrial carbon pools are in equilibrium (Fig.1). 
- In the polar regions, it will take a couple of 1000 years!

What does equilibrium mean?
~~~~~~~~~~~~~~~~~~~~~~~~~~~
- Typically you'd cycle through a range of given meteorological years - referred to as time-slice (e.g. 1990-2000). 
- After an initial rapid increase/decrease in your variables, you will start to see the interannual variability of the chosen time slice (wiggling). 
- Equilibrium is reached as soon as consecutive time slices differ by less than a threshold that you have to choose (< 0.1-1%).

How to perform a spin-up?
~~~~~~~~~~~~~~~~~~~~~~~~~
- Usually, we do a two-staged spin-up. 
- See the necessary XML change commands below.

1. Accelerated decomposition (reduced size of carbon pools)
   
.. code-block:: bash

    ./xmlchange CLM_FORCE_COLDSTART="on"
    ./xmlchange CLM_NML_USE_CASE="2000_control"
    ./xmlchange DATM_CLMNCEP_YR_START="1991"
    ./xmlchange DATM_CLMNCEP_YR_END="2010"
    ./xmlchange DATM_PRESAERO="clim_2000"
    ./xmlchange CCSM_CO2_PPMV="369."
    ./xmlchange STOP_OPTION="nyears"
    ./xmlchange RUN_REFDATE="0001-01-01"
    ./xmlchange RUN_STARTDATE="0001-01-01"
    #Additional xml changes for AD mode
    ./xmlchange CLM_ACCELERATED_SPINUP="on"
    #Number of years for spin-up 
    #For Tropics only STOP_N=100 is needed!
    ./xmlchange STOP_N=100
    #Frequency of restart files (1/4 of STOP_N)
    ./xmlchange REST_N=25
    #Setting wall clock time (on saga 5 years / 30 min)
    #./xmlchange --subgroup case.st_archive JOB_WALLCLOCK_TIME=36:00:00
    ./xmlchange JOB_WALLCLOCK_TIME=36:00:00
    ./case.setup
    ./preview_namelists


2. Normal spin-up (original size of carbon pools)
   
.. code-block:: bash
    
    # Switch off cold start
    ./xmlchange CLM_FORCE_COLDSTART="off"
    ./xmlchange CONTINUE_RUN="FALSE"
    #Re-use directory but re-define reference time
    ./xmlchange RUN_REFDATE="0401-01-01"
    ./xmlchange RUN_STARTDATE="0401-01-01"
    #Additional xml changes for AD mode
    ./xmlchange CLM_ACCELERATED_SPINUP="off"
    ./xmlchange STOP_N=100
    #Frequency of restart files (1/4 of STOP_N)
    ./xmlchange REST_N=25
    #Setting wall clock time
    #./xmlchange --subgroup case.st_archive JOB_WALLCLOCK_TIME=36:00:00
    ./xmlchange JOB_WALLCLOCK_TIME=36:00:00
    ./case.setup
    #Point to restart file (IMPORTANT: Do not forget the single-quotation marks '<path to restart file>')
    echo "finidat = '${restart_file}'" >> user_nl_clm
    echo "Restart file: ${restart_file}"

----------------

