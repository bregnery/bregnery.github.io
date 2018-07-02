---
title: BEST Using the Standalone Function
permalink: /docs/BESTstandaloneTutorial/
---

## Introduction

## Instructions

### Dependencies

BEST requires CMSSW and was configured on CERN's lxplus and Fermilab's lpc. The BEST standalone function can be used with ``CMSSW_9_4_X``.
In order to install CMSSW on lxplus, execute the following

```bash
cmsrel CMSSW_9_4_8
cd CMSSW_9_4_8/src/
scram b
cmsenv
```

### Installation

BEST was programmed in python and trained using [scikit-learn](http://scikit-learn.org/stable/index.html). However, the standard 
EDAnalyzer in CMS uses C++, so BEST requires an additional package in order to become a C++ function. This package was can be
found [here](https://github.com/demarley/lwtnn/tree/CMSSW_8_0_X-compatible). The first step in setting up BEST is installing
this package:

```bash
cd CMSSW_9_4_8/src/
mkdir lwtnn
cd lwtnn
git clone https://github.com/demarley/lwtnn.git
cd lwtnn
git checkout CMSSW_8_0_X-compatible
scram b -j8
```

Now, BEST can be installed and compiled.

```bash
cd CMSSW_9_4_8/src/
git clone https://github.com/cms-ttbarAC/BESTAnalysis.git 
cd BESTAnalysis
git checkout master
cd BoostedEventShapeTagger
scram b -j8
```

This configuration allows for BEST to be use as a function in CMS's EDAnalyzer. These instructions were adapted 
from [BEST](https://github.com/justinrpilot/BESTAnalysis) and [lwtnn](https://github.com/demarley/lwtnn/tree/CMSSW_8_0_X-compatible).

### Using the BEST function

EDAnalyzer modules allow analysts to interact with the CMS framework in order to study physics data. I will not go
into great detail about how to use EDAnalyzer, because there is already a great tutorial in the 
[CMS work book](https://twiki.cern.ch/twiki/bin/view/CMSPublic/WorkBookWriteFrameworkModule). I did
include a couple of useful instructions from this tutorial to get the BEST function up and running.

#### Getting an EDAnalyzer Template

A template EDAnalyzer can be automatically created by CMSSW.

```bash
cd CMSSW_9_4_8/src/
mkdir Demo
cd Demo
mkedanalyzer DemoAnalyzer
cd DemoAnalyzer
scram b
```
#### Edit the EDAnalyzer

The directory ``CMSSSW_9_4_8/src/Demo/DemoAnalyzer/plugins`` contains several files that need to be altered. First, 
add the necessary functions to the BuildFile.xml

```xml
<use name="lwtnn/lwtnn"/>
<use name="BESTAnalysis/BoostedEventShapeTagger"/>
```

