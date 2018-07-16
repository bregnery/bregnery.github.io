---
title: FEWZ Tutorial
permalink: /docs/FEWZtutorial/
---

# FEWZ: Fully Exclusize W and Z Production

## Introduction

[FEWZ](http://gate.hep.anl.gov/fpetriello/FEWZ.html) is a fortran-based Vegas simulator for calculating leading order (LO), next-to-leading order (NLO), and next-next-leading order (NNLO)
Drell-Yan processes [1]. In the past, I used this to calculate the background Drell-Yan shape for the Higgs to dimuons search. 

## Instructions

All of the files necessary for compiling and using FEWZ are contained in my [GitHub Repository](https://github.com/bregnery/FEWZforHiggs2mumu),
so first clone this repository. Please note that the files in that repository are the programs for the tutorial and may be out-of-date. 
An up-to-date version of FEWZ can be found on the
[FEWZ website](http://gate.hep.anl.gov/fpetriello/FEWZ.html).

```bash
git clone https://github.com/bregnery/FEWZforHiggs2mumu.git
```

### Installation

These installation instructions are from [FEWZ](http://gate.hep.anl.gov/fpetriello/FEWZ.html) [1]. 
Please note that you do not need to use Condor for submission, a shell submission
script can be easily written for other systems. I included an example near the end of this tutorial.

```bash
cd FEWZforHiggs2mumu/FEWZ/FEWZ_3.1.b2/
make fewz # makes local executable
make condor_fewz # makes executable for Condor systems
make # makes both, an error will be thrown if you do not have Condor installed
```

All makes will compile the necessary CUBA library provided automatically

## Organization

The ``input.txt`` file contains the adjustable parameters about the process being simulated. For example: the collision energy,
particle masses, pT and eta ranges for different particles, PDF information, and particle couplings. The ``histogram.txt`` file 
contains bounds and bins for the values that are calculated by FEWZ. The output will be stored in ``.dat`` files.

For some examples, click [here](https://github.com/bregnery/FEWZforHiggs2mumu/tree/master/FEWZ/FEWZ_3.1.b2/bin). The files 
labeled ``mu13tev-higgs*.txt`` contain parameters used to calculate the Drell-Yan background for various mass ranges at 
CMS with collision energies of 13TeV. These files also use the LHAPDF set which are included in this repository [2].

### Running Locally

In order to run the FEWZ program, first change into the FEWZ bin directory. Then copy the input.txt 
and histograms.txt from the desired directory to the bin directory. Now run the local version of 
FEWZ in the bin directory by using the shell script local_run.sh with the following command

```bash
./local_run.sh z <run_dir> input.txt histograms.txt results.dat .. <number_of_processors>
```

The ``run_dir`` is a directory created in order to store information from the run. The default ``number_of_processors`` is one.

```bash
./finish.sh <run_dir> <order_prefix>.<name_of_results_file>.dat
```

The possible order prefixes are LO, NLO, and NNLO.

## Example Running on the HiPerGator

This is an example for running FEWZ on a server that uses SLURM (Simple Linux Utility for Resource Management).
Specifically, this is for the University of Florida's HiPerGator 2.0.
First, load the necessary modules and enter development mode.

```bash
module load ufrc
module load gcc
srundev --time=04:00:00
```

The submit script used in this example is ``FEWZforHiggs2mumu/FEWZ/FEWZ_3.1.b2/bin/dy_nlo_submit.sh``. Now, use this to submit
to the HiPerGator (or other SLURM system).

```bash
sbatch dy_nlo_submit.sh
squeue -u <username>
```

More information about the University of Florida's HiPerGator can be found the [UFRC Wiki](https://help.rc.ufl.edu/doc/UFRC_Help_and_Documentation).

## Remarks

FEWZ is a useful tool for calculating backgrounds; other programs can be used to fit the shape of these backgrounds. In the
search for Higgs to dimuons, we used FEWZ to calculate our Drell-Yan background and found a function to fit the background shape 
for the dimuon invariant mass. To see this in further detail, please see appendix A in our [CMS analysis note](http://cms.cern.ch/iCMS/jsp/db_notes/noteInfo.jsp?cmsnoteid=CMS%20AN-2017/098)

For an example fitting program, please see my 
[python modules repository](https://github.com/bregnery/PythonModulesForCMSfits/tree/master/example/FEWZstudies)

## References

1. R. Gavin, Y. Li, F. Petriello, and S. Quackenbush, Comput. Phys. Commun. **182**, 2388-2403 (2011)
2. A. Buckley, J. Ferrando, S. Lloyd, K. Nordstrom, B. Page, M. Rufenacht, M. Schonerr, and G. Watt, Eur. Phys. J. **C75**, 132 (2015)
