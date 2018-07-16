---
title: Monte Carlo Production with the 2016 CMS Scheme
permalink: /docs/MonteCarloProduction2016/
---

## General Steps for Monte Carlo Production

Monte Carlo simulations use statistical Monte Carlo methods on various physical models to predict the results of 
a particle collision and then simulate the detector response to those particles. For CMS, this is accomplished by
using programs like MadGraph and Pythia to calculate differential cross sections and final states, then passing the 
results to a program like GEANT which simulates the detector response. Most of this can be done using cmsdriver to 
create automated configuration files.

I preformed a Monte Carlo simulation with two main steps. First, I used MadGraph data cards as inputs for gridpack 
generation. Then, I used cmsdriver to generate configuration files that used pythia to finish the calculations and
GEANT to simulate the detector response. Additionally, cmsdriver converts the results of this simulation into the 
miniAOD file format.

## Gridpack Creation

A gridpack is a tarball (compressed) file that contains a bunch of calculation results stored in away that can be 
read with cmsdriver. I have only produced gridpacks using MadGraph, but this can be done with other programs like 
PowHeg. A detailed guide for this, is the [MadGraphGuide](https://twiki.cern.ch/twiki/bin/viewauth/CMS/QuickGuideMadGraph5aMCatNLO). 
The Genproductions group is in charge of producing gridpacks that are requested by each physics group, their data 
cards are stored on [GitHub, genproductions](https://github.com/cms-sw/genproductions/tree/pre2017)

Producing a MadGraph gridpack requires four main files: ``MC_proc_card.dat``, ``MC_run_card.dat``, ``MC_customizecards.dat`` 
and ``MC_extramodels.dat``. Examples of these files can be found on the Genproductions GitHub repository. These examples 
are excellent templates for making gridpacks. Gridpacks can then be produced locally or by submitting to a computing grid. 
Below is a tutorial based on how I created gridpacks.

### MadGraph Gridpack Tutorial

These programs should be run on CERN's lxplus or Fermilab's lpc. Gridpacks must be created outside of CMSSW and without 
cmsenv. To obtain the programs necessary for gridpack creation and some example data cards, please clone my GitHub
repository.

```git
git clone https://github.com/bregnery/MadGraphCardsForCMSSW
```

Now, permissions need to be updated in order to run shell scripts

```bash
chmod u+x gridpack_generation.sh
```

Then, update ``gridpack_generation.sh`` to make sure all fifo pipes are directed to /tmp director. Now, a gridpack can 
be generated using the desired data card.

```bash
./gridpack_generation.sh <name of process card without _proc_card.dat> <folder containing cards relative to current location>
```

For example,

```bash
./gridpack_generation.sh Radion_hh_WWWW_jets_narrow_M3500 cards/Radion_hh_WWWW_jets_narrow_M3500/
```

#### Using Cluster to Create Gridpacks

To submit jobs on lxplus and store them locally, use the ``submit_gridpack_generation_local.sh``

```bash
./submit_gridpack_generation_local.sh <memory> <queue for master job> <name of process card without _proc_card.dat> <folder containing cards relative to current location> <queue>
```

For example,

```bash
./submit_gridpack_generation_local.sh 15000 2nw Radion_hh_WWWW_jets_narrow_M3500 cards/Radion_hh_WWWW_jets_narrow_M3500/ 8nh
```

#### Notes about Creating/Editing Data Cards

When creating or editing data cards, make sure to create four cards with the same name as your repository.

```bash
<repository>_proc_card.dat
<repository>_run_card.dat
<repository>_customizecards.dat
<repository>_extramodels.dat
```

Make sure to list models needed in extra models if generating a sample with something other than the Standard Model. 
The number of events is listed listed in 
``run_card.dat``. When using the gridpacks **DO NOT generate more events than what is listed in** ``run_card.dat``.

## From Gridpacks to MiniAOD

After creating (or obtaining) a gridpack, it must be converted into a MiniAOD sample. To obtain the proper workflow for 
your specific case, find similar Monte Carlo datasets on the [McM website](https://cms-pdmv.cern.ch/mcm/). Then click the 
icon that looks like a bar graph to view the work flow. Below are details for the 2016 Monte Carlo production process.

The work flow for the 2016 Monte Carlo production process is:
1. Gridpack (or pythia configuration with cmsdriver)
2. GEN-SIM
3. GEN-SIM-RAW
4. AODSIM
5. MiniAOD

Each of the next steps involves using cmsdriver.py to generate configuration files that are then run with cmsRun. The 
commands to produce these files for various gridpacks are located on the [McM website](https://cms-pdmv.cern.ch/mcm/). 
To obtain the cmsdriver commands, click on the "Get Set Up commands". You should record the commands for each PrepID in the 
work flow. For example, I used the Radion_hh_hVVbb MadGraph cards 
as a template, so I searched for the dataset "Radion_hh_hVVbb*" on McM to find and use the corresponding setup commands. For 
2016 Monte Carlo production, the setup commands can be found under the following PrepID's:

* **RunIISummer15wmLHEGS** - These commands use a gridpack to produce GEN-SIM and LHE files
* **RunIISummer16DR80Premix** - These commands use GEN-SIM files produce GEN-SIM-RAW files, add in pile-up, and then produce 
  AODSIM files (along with an outdated MiniAOD file)
* **RunIISummer16MiniAODv2** - These commands use AODSIM to produce the up-to-date MiniAOD files

Make sure to change input files to the proper storage location. In particular, make sure that you are using the correct gridpack! 
For the step adding pile-up, I encountered some difficulties. DAS has recently changed their client, so the pile-up files will 
need to be manually entered into the cfg file. The tutorial below shows how I created the cmsdriver configuration
files.

### CMSSW cmsdriver Commands Tutorial

Obtain the proper CMSSW release, clone this repository, and compile.

```bash
cmsrel CMSSW_7_1_30
cd CMSSW_7_1_30/src/
git clone https://github.com/bregnery/hhMCgenerator.git
scram b
cmsenv
cd hhMCgenerator
```

Make sure to copy your gridpack into this directory. Now, obtain the proper python fragments (these should also be included
on the McM page).

```bash
curl -s --insecure https://cms-pdmv.cern.ch/mcm/public/restapi/requests/get_fragment/B2G-RunIISummer15wmLHEGS-01167 --retry 2 --create-dirs -o Configuration/GenProduction/python/B2G-RunIISummer15wmLHEGS-01167-fragment.py 
[ -s Configuration/GenProduction/python/B2G-RunIISummer15wmLHEGS-01167-fragment.py ]
scram b
```

Now modify the python fragment to input the proper grid pack. In 
``CMSSW_X_X_X/src/Configuration/Genproduction/python/B2G-RunIISummer15wmLHEGS-01167-fragment.py`` line 5 should 
be altered to include the gridpack tarball file.

```python
args = cms.vstring('<gridpack_tarball>.tar.xz'),
```

Also make sure that the fragment contains a number of events less than or equal to the number of events used in the 
MadGraph cards that produced the gridpack.

Now, a cfg file can be produced using cmsdriver with the python fragment.

```bash
cmsDriver.py Configuration/GenProduction/python/B2G-RunIISummer15wmLHEGS-01167-fragment.py --fileout file:B2G-RunIISummer15wmLHEGS-01167.root --mc --eventcontent RAWSIM,LHE --customise SLHCUpgradeSimulations/Configuration/postLS1Customs.customisePostLS1,Configuration/DataProcessing/Utils.addMonitoring --datatier GEN-SIM,LHE --conditions MCRUN2_71_V1::All --beamspot Realistic50ns13TeVCollision --step LHE,GEN,SIM --magField 38T_PostLS1 --python_filename B2G-RunIISummer15wmLHEGS-01167_1_cfg.py --no_exec -n 97
```

Then, use the configuration file to produce GEN-SIM and LHE files.

```bash
scram b
cmsRun B2G-RunIISummer15wmLHEGS-01167_1_cfg.py
```

If the program runs correctly, you should have a .root file to use with the next step.


Production of GEN-SIM-RAW and AODSIM files requires CMSSW_8_0_21.
Obtain this CMSSW release, clone this repository, and compile.

```bash
cmsrel CMSSW_8_0_21
cd CMSSW_8_0_21/src/
git clone https://github.com/bregnery/hhMCgenerator.git
scram b
cmsenv
cd hhMCgenerator
```

Make sure to copy your GEN-SIM and LHE files from the previous step to the new directory.

```bash
cp <your_path>/CMSSW_7_1_30/src/hhMCgenerator/B2G-RunIISummer15wmLHEGS-01167_inLHE.root .
cp <your_path>/CMSSW_7_1_30/src/hhMCgenerator/B2G-RunIISummer15wmLHEGS-01167.root .
```

Now, another cfg file can be produced using cmsdriver. Note that the DAS client has changed so you will need to copy the pile up files from ``B2G-RunIISummer16DR80Premix-01267_1_cfg.py``.

```bash
cmsDriver.py step1 --fileout file:Radion_hh_wwwww_M3500_GEN-SIM-RAW.root  --pileup_input "dbs:/Neutrino_E-10_gun/RunIISpring15PrePremix-PUMoriond17_80X_mcRun2_asymptotic_2016_TrancheIV_v2-v2/GEN-SIM-DIGI-RAW" --mc --eventcontent PREMIXRAW --datatier GEN-SIM-RAW --conditions 80X_mcRun2_asymptotic_2016_TrancheIV_v6 --step DIGIPREMIX_S2,DATAMIX,L1,DIGI2RAW,HLT:@frozen2016 --nThreads 4 --datamix PreMix --era Run2_2016 --python_filename B2G-RunIISummer16DR80Premix-01267_1_cfg.py --no_exec --customise Configuration/DataProcessing/Utils.addMonitoring -n 97

cmsDriver.py step2 --filein file:Radion_hh_wwwww_M3500_GEN-SIM-RAW.root --fileout file:B2G-RunIISummer16DR80Premix-Radion_hh_wwww_M3500_AODSIM.root --mc --eventcontent AODSIM --runUnscheduled --datatier AODSIM --conditions 80X_mcRun2_asymptotic_2016_TrancheIV_v6 --step RAW2DIGI,RECO,EI --nThreads 4 --era Run2_2016 --python_filename B2G-RunIISummer16DR80Premix-AODSIM_2_cfg.py --no_exec --customise Configuration/DataProcessing/Utils.addMonitoring -n 97
```

Now, use the configuration file to produce GEN-SIM-RAW.

```bash
scram b
cmsRun B2G-RunIISummer16DR80Premix-01267_1_cfg.py
```

If the program was successful the output will be Radion_hh_wwwww_M3500_GEN-SIM-RAW.root. Then, use the next configuration file to produce AODSIM.

```bash
scram b
cmsRun B2G-RunIISummer16DR80Premix-AODSIM_2_cfg.py
```

Now, there should be ``B2G-RunIISummer16DR80Premix-Radion_hh_wwww_M3500_AODSIM.root``

Next, create the last cfg file using cmsdriver.

```bash
cmsDriver.py step1 --fileout file:Radion_hh_wwww_M3500_MiniAOD.root --mc --eventcontent MINIAODSIM --runUnscheduled --datatier MINIAODSIM --conditions 80X_mcRun2_asymptotic_2016_TrancheIV_v6 --step PAT --nThreads 4 --era Run2_2016 --python_filename B2G-RunIISummer16MiniAODv2_Radion_hh_wwww_M3500-MiniAOD_1_cfg.py --no_exec --customise Configuration/DataProcessing/Utils.addMonitoring -n 82
```

Now use the configuration file to produce MiniAOD.

```bash
scram b
cmsRun B2G-RunIISummer16MiniAODv2_Radion_hh_wwww_M3500-MiniAOD_1_cfg.py
```

If the whole process was successful, the output should be ``Radion_hh_wwww_M3500_MiniAOD.root``.

## Remarks

After creating the necessary configuration files using cmsdriver, I created a work flow that utilized crab and stored the simulation files
on the LPC using EOS. This is documented on my [hhMCgenerator repository](https://github.com/bregnery/hhMCgenerator).

