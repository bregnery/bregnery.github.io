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
In the same directory, the DemoAnalyzer.cc file must also be altered. 

```cpp
// In the include files
#include "BESTAnalysis/BoostedEventShapeTagger/interface/BoostedEventShapeTagger.h"
#include "DataFormats/PatCandidates/interface/Jet.h"

// In the class, under private:
BoostedEventShapeTagger *m_BEST;

// In the constructor pass the name of the configuration file
m_BEST = new BoostedEventShapeTagger( "/full_path/BESTAnalysis/BoostedEventShapeTagger/data/config.txt" );
   // Define input tags
edm::InputTag ak8JetsTag_;
ak8JetsTag_ = edm::InputTag("slimmedJetsAK8", "", "PAT");
ak8JetsToken_ = consumes<std::vector<pat::Jet> >(ak8JetsTag_);

// In the destructor
delete m_BEST;

// In the event loop
using namespace edm;
using namespace std;
   // Find objects corresponding to the token and link to the handle
Handle< std::vector<pat::Jet> > ak8Jets;
iEvent.getByToken(ak8JetsToken_, ak8Jets);
   // loop over the jets
for (std::vector<pat::Jet>::const_iterator jetBegin = ak8Jets->begin(), jetEnd = ak8Jets->end(), ijet = jetBegin; ijet != jetEnd; ++ijet){
   std::map<std::string,double> NNresults = m_BEST->execute(*ijet);  // ijet is a pat::Jet
   int particleType = m_BEST->getParticleID();                      // automatically calculate the particle classification
}
```

Now, in order to properly compile everything, the BESTAnalyzer and BESTProducer directories must be
removed as they are not compatable with CMSSW_9_X.

```bash
cd CMSSW_9_4_8/src/
rm -r -f BESTAnalysis/BESTAnalyzer/
rm -r -f BESTAnalysis/BESTProducer/
```

Finally, in the file ``BESTAnalysis/BoostedEventShapeTagger/data/config.txt`` the ``dnnFile`` must be updated
to include the full path. After this, everything can be compiled.

```bash
cd CMSSW_9_4_8/src/
scram b -j8
```

Now a ``run.py`` file must be created in order to use the EDanalyzer. The run file should be made in 
``CMSSW_9_4_8/src/Demo/DemoAnalyzer/test`` and should include the following:

```python

``` 



