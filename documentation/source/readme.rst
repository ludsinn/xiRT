xiRT - Introduction
===================

xiRT is a deep learning tool to predict the separation behaviour (i.e. retention times) of linear and crosslinked peptides
from single to multiple fractionation dimensions (e.g. reversed-phase chromatography, coupled to the mass spectrometer).
xiRT was developed with a combination of SCX / hSAX / RP chromatography. However, xiRT supports all
available chromatographic and other peptide separation methods.

Description
***********

xiRT is meant to be used for generating additional information about crosslink spectrum matches (CSMs). We used xiRT within machine learning-based
rescoring frameworks but the usage can be extended to generate improved spectral libraries, design targeted acquisitions etc.

xiRT offers several training / prediction  modes that need to be configured
depending on the use case. At the moment training, prediction, crossvalidation are the supported
modes.
- *training*: trains xiRT on the input CSMs (using 10% input for validation) and stores a trained model
- *prediction*: use a pretrained model and predict RTs for the input CSMs
- *crossvalidation*: load/train a model and predict RTs for all data points without using them
in the training process. Requires the training of several models during CV.

**Note**: all modes can be supplemented by using a pre-trained model ("transfer learning").
