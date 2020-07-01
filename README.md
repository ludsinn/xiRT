![logo](./documentation/xiRT_logo.png) 

[![GitHub](https://flat.badgen.net/github/license/compomics/ms2pip_c)](https://www.apache.org/licenses/LICENSE-2.0)
[![Twitter](https://flat.badgen.net/twitter/follow/rappsilberlab?icon=twitter)](https://twitter.com/compomics)
[![Python 3.6](https://img.shields.io/badge/python-3.6-blue.svg)](https://www.python.org/downloads/release/python-360/)
![coverage](./documentation/coverage.svg)  

A python package for multi-dimensional retention time prediction for linear and crosslinked 
peptides using a (siamese) deep neural network architecture.
---

- [Overview](#overview)
- [Description](#Description)
- [Installation](#Installation)

---
## overview

xiRT is a deep learning tool to predict the RT of linear and cross-linked peptides from multiple
fractionation dimensions including RP typtically coupled to the mass spectrometer. xiRT requires the
columns shown in the table below. Importantly, the xiRT framework requires that CSM are sorted
such that in the Peptide1 - Peptide2, Peptide1 is the longer or lexicographically larger one.

![xiRT Architecture](documentation/xiRT.PNG)

## Description
[xiRT is currently under construction to make it (painlessly) work for any chromatography combination.]

# Usage

## Installation 
To install xiRT a pip package is under development. Future release can be installed via:
>pip install xirt

To enable CUDA support, the specific libraries need to be installed manually. The python 
dependencies are covered via the pip installation.

To use xiRT a simple command line script gets installed with pip. To run the predictions ...

## config file
To adapt the xiRT parameters a yaml config file needs to be prepared. The configuration file
is used to determine network parameters (number of neurons, layers, regularization) but also for the
definition of the prediction task (classification, regression, ordered regression). Depending
on the decoding of the target variable the output layers need to be adapted. For standard RP 
prediction, regression is essentially the only viable option. For SCX/hSAX (general classification)
prediction the prediction task can be formulated as classification, regression or ordered regression.
For the usage of regression for fractionation it is recommended that the estimated salt concentrations
are used as target variable for the prediction (raw fraction numbers are possible too).


## input format
| short name         | explicit column name | description                                                                    | Example     |
|--------------------|----------------------|--------------------------------------------------------------------------------|-------------|
| peptide sequence 1 | Peptide1             | Alpha peptide sequence for crosslinks                                        | PEPRTIDER   |
| peptide sequence 2 | Peptide2             | Beta peptide sequence for crosslinks, or empty                                 | ELRVIS      |
| link site 1        | LinkPos1             | Crosslink position in the peptide (0-based)                                    | 3           |
| link site 2        | LinkPos2             | Crosslink position in the beta peptide (0-based                                | 2           |
| precursor charge   | Charge               | Precursor charge of the crosslinked peptide                                    | 3           |
| score              | Score                | Single score from the search engine                                            | 17.12       |
| unique id          | CSMID                | A unique index for each entry in the result table                              | 0           |
| decoy              | isTT                 | Binary column which is True for any TT identification and False for TD, DD ids | TT          |
| fdr                | fdr                  | Estimated false discovery rate                                                 | 0.01        |
| fdr level          | fdrGroup             | String identifier for heteromeric and self links (splitted FDR)                | heteromeric |

The first four columns should be self explanatory, if not check the sample input *#TODO*. 
The fifth column ("CSMID") is a unique(!) integer that can be used as to retrieve CSMs. In addition, 
depending on the number retention time domains that should to be learned/predicted the RT columns 
should have a prefix "xirt_". For instance if 2 dimensions were measured in SCX and RP, the input
should contain "xirt_RP" and "xirt_SCX". The same naming needs to be adapted for the network config.

## CLI interface
[under construction]
## network config
## xiRT config

### Contributors
- Sven Giese


### Citation
If you consider xiRT helpful for your work please consider citing our manuscript. Currently, only
available on bioarchive "xiRT: Retention Time Prediction using Neural Networks increases Identifications in Crosslinking Mass Spectrometry".