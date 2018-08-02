---
title: BEST Using the Standalone Function
permalink: /docs/BESTstandaloneTutorial/
---

This guide has been adapted from the instructions by [justinrpilot](https://github.com/justinrpilot/BESTAnalysis/tree/master)
and [demarley](https://github.com/demarley/lwtnn/tree/CMSSW_8_0_X-compatible#cmssw-compatibility).

## Introduction

BEST: Booosted Event Shape Tagger is a neural network that was trained to identify AK8 jets that come
from heavy Standard Model particles. This neural network was developed by CMS analysts and is
avaliable on GitHub from [justinrpilot](https://github.com/justinrpilot/BESTAnalysis/tree/master). 
Information about how BEST was trained is avaliable on [arxive](https://arxiv.org/pdf/1606.06859.pdf).
This guide describes how to use BEST as a function in a CMS EDAnalyzer.

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
EDAnalyzer in CMS uses C++, so BEST requires an additional package in order to become a C++ function. This package can be
found [here](https://github.com/demarley/lwtnn/tree/CMSSW_8_0_X-compatible). The first step in setting up BEST is installing
this package [2]:

```bash
cd CMSSW_9_4_8/src/
mkdir lwtnn
cd lwtnn
git clone https://github.com/demarley/lwtnn.git
cd lwtnn
git checkout CMSSW_8_0_X-compatible
scram b -j8
```

Now, BEST can be installed and compiled [1].

```bash
cd CMSSW_9_4_8/src/
git clone https://github.com/cms-ttbarAC/BESTAnalysis.git 
cd BESTAnalysis
git checkout master
cd BoostedEventShapeTagger
scram b -j8
```

This configuration allows for BEST to be use as a function in CMS's EDAnalyzer [1].

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
add the necessary functions to the ``BuildFile.xml`` [1].

```xml
<use name="lwtnn/lwtnn"/>
<use name="BESTAnalysis/BoostedEventShapeTagger"/>
```
In the same directory, the DemoAnalyzer.cc file must also be altered [1]. 

```cpp
// In the include files
#include "FWCore/ServiceRegistry/interface/Service.h"
#include "CommonTools/UtilAlgos/interface/TFileService.h"
#include "TTree.h"
#include "TFile.h"
#include "BESTAnalysis/BoostedEventShapeTagger/interface/BoostedEventShapeTagger.h"
#include "DataFormats/PatCandidates/interface/Jet.h"

// In the class, under private:
BoostedEventShapeTagger *m_BEST;
TTree *bestTree;
edm::EDGetTokenT<std::vector<pat::Jet> > ak8JetsToken_;
std::map<std::string, float> treeVars;
std::vector<std::string> listOfVars;

// In the constructor pass the name of the configuration file
m_BEST = new BoostedEventShapeTagger( "/full_path/BESTAnalysis/BoostedEventShapeTagger/data/config.txt" );
   // Use TFile service to create a tree to store histogram variables
edm::Service<TFileService> fs;
bestTree = fs->make<TTree>("bestTree","bestTree");
   // Create tree variables and branches
listOfVars.push_back("jet1_particleType");
listOfVars.push_back("jet1_dnn_w");
listOfVars.push_back("jet1_dnn_z");
listOfVars.push_back("jet1_dnn_h");
listOfVars.push_back("jet1_dnn_top");
listOfVars.push_back("jet1_dnn_qcd");
for (unsigned i = 0; i < listOfVars.size(); i++){
   treeVars[ listOfVars[i] ] = -999.99;
   bestTree->Branch( (listOfVars[i]).c_str() , &(treeVars[ listOfVars[i] ]), (listOfVars[i]+"/F").c_str() );
}
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
int nJets = 0;
for (std::vector<pat::Jet>::const_iterator jetBegin = ak8Jets->begin(), jetEnd = ak8Jets->end(), ijet = jetBegin; ijet != jetEnd; ++ijet){
      // These are the requirements for a Jet to be used with BEST
   if(ijet->numberOfDaughters() >= 2 && ijet->pt() >= 500 && ijet->userFloat("ak8PFJetsCHSSoftDropMass") > 40 ){
      nJets++;
      std::map<std::string,double> NNresults = m_BEST->execute(*ijet);  // ijet is a pat::Jet
      int particleType = m_BEST->getParticleID();                      // automatically calculate the particle classification
      if(nJets == 1){
         treeVars["jet1_particleType"] = particleType;
         treeVars["jet1_dnn_w"] = NNresults["dnn_w"];
         treeVars["jet1_dnn_z"] = NNresults["dnn_z"];
         treeVars["jet1_dnn_h"] = NNresults["dnn_higgs"];
         treeVars["jet1_dnn_top"] = NNresults["dnn_top"];
         treeVars["jet1_dnn_qcd"] = NNresults["dnn_qcd"];
      }
   }
}
   // Fill the tree that stores the NNresults
bestTree->Fill();

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

import FWCore.ParameterSet.Config as cms


process = cms.Process("run")

process.load("FWCore.MessageService.MessageLogger_cfi")

process.maxEvents = cms.untracked.PSet( input = cms.untracked.int32(-1) )

process.source = cms.Source("PoolSource",
    # replace 'myfile.root' with the source file you want to use
    fileNames = cms.untracked.vstring(
        "file:myfile.root"
        )
)
process.MessageLogger.cerr.FwkReport.reportEvery = 1000

process.run = cms.EDAnalyzer('DemoAnalyzer')

process.TFileService = cms.Service("TFileService", fileName = cms.string("best_results.root") )

process.out = cms.OutputModule("PoolOutputModule",
                               fileName = cms.untracked.string("ana_out.root"),
                               SelectEvents   = cms.untracked.PSet( SelectEvents = cms.vstring('p') ),
                               outputCommands = cms.untracked.vstring('drop *',
                                                                      'keep *_*run*_*_*'
                                                                      #, 'keep *_goodPatJetsCATopTagPF_*_*'
                                                                      #, 'keep recoPFJets_*_*_*'
                                                                      )
                               )
process.outpath = cms.EndPath(process.out)

process.p = cms.Path(process.run)
``` 

Now the EDAnalyzer can be run!

```bash
cd CMSSW_9_4_8/src/Demo/DemoAnalyzer/test/
cmsenv
cmsRun run.py
```
The output will be ``best_results.root`` which contains the BEST probabilities for the first
AK8 jet identified in an event. This process can be expanded to include the BEST probability
results for all of the AK8 jets in an event. Below is a table explaining what each of the 
NNresults strings mean [1].

| String       | Definition       |
|--------------|------------------|
|``dnn_w``     | BEST's calculated probability that the AK8 jet is from a W boson |
|``dnn_z``     | BEST's calculated probability that the AK8 jet is from a Z boson |
|``dnn_higgs`` | BEST's calculated probability that the AK8 jet is from a Higgs boson |
|``dnn_top``   | BEST's calculated probability that the AK8 jet is from a top quark |
|``dnn_qcd``   | BEST's calculated probability that the AK8 jet is from a QCD background |

## References

1. justinrpilot, BESTAnalysis, https://github.com/justinrpilot/BESTAnalysis/tree/master
2. demarley, lwtnn, https://github.com/demarley/lwtnn/tree/CMSSW_8_0_X-compatible#cmssw-compatibility

