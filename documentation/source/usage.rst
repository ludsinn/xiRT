General Usage
=============
The command line interface (CLI) requires three inputs:

1) input spectrum matches (CSMs or PSMs)
2) a `YAML <https://docs.ansible.com/ansible/latest/reference_appendices/YAMLSyntax.html>`_ file to configure the neural network architecture
3) another YAML file to configure the general training / prediction behaviour, called setup-config

Configs are available via `github <https://github.com/Rappsilber-Laboratory/xiRT/tree/master/default_parameters>`_).
Alternatively, up-to-date configs can be generated using the xiRT package itself and then edited by the user:

.. code-block:: console

    > xirt -p learning_params.yaml
    >
    > xirt -s xirt_params.yaml

To use xiRT, these files are called as shown below:

.. code-block:: console

    >xirt(.exe) -i CSMs.csv -o out_dir -x xirt_params.yaml -l learning_params.yaml

To adapt xiRT's parameters, a yaml config file needs to be prepared. The configuration file
is used to determine network parameters (number of neurons, layers, regularization) but also for the
definition of the prediction task (classification, regression, ordinal regression). Most of the
values do not need to adapted in standard use cases.
Depending on the decoding of the target variable, the output layers need to be adapted. For standard RP
prediction, regression is essentially the only viable option. For SCX/hSAX (i.e. any classification
from fractionation experiments), the prediction task can be formulated as classification,
regression or ordinal regression. For the usage of regression, it is recommended
to use estimated eluent concentrations as target variable for the prediction (raw
fraction numbers are possible as well). Please see below for examples on different parameterizations.

Quick start
'''''''''''

The GitHub repository contains a few example files. Download the following files from  `HERE <https://github.com/Rappsilber-Laboratory/xiRT/tree/master/sample_data>`_:

- DSS_xisearch_fdr_CSM50percent_minimal.csv
- xirt_params_rp.yaml
- learning_params_training_cv.yaml

This set of files can now be used to perform a RP (only) prediction on crosslink data.
To run xiRT on the data, call the main function as follows after successfull installation:

.. code-block:: console

    > xirt -i DSS_xisearch_fdr_CSM50percent_minimal.csv -o xirt_results/ -x xirt_params_rp.yaml -l learning_params_training_cv.yaml



Examples
========

This section covers a few examples. Please check the :ref:`Parameters <linking-parameters>` section to gain
a better understanding for each of the variables.


Reversed-phase Prediction
'''''''''''''''''''''''''
While xiRT was developed for multi-dimensional RT prediction it can also be used for a single
dimension. For this, the xiRT YAML parameter file needs to be adapted as follows::

    output:
      rp-activation: linear
      rp-column: rp
      rp-dimension: 1
      rp-loss: mse
      rp-metrics: mse
      rp-weight: 1

    predictions:
      continues:
        - rp
      # simply write fractions: [] if no fraction prediction is desired
      fractions: []

This configuration assumes that the target column in the input data is named "rp" and that the
scale is continuous (*rp-activation: linear*). If that is the case, the other parameters should
not be changed (dimension, loss, metric, weight).

2D RT Prediction - Ordinal Task
'''''''''''''''''''''''''''''''

Many studies apply one pre-fractionation method (e.g. SEC, SCX) and then acquire the samples on LC-MS.
For this experimental setup, the xiRT config could look like this::

    output:
      rp-activation: linear
      rp-column: rp
      rp-dimension: 1
      rp-loss: mse
      rp-metrics: mse
      rp-weight: 1

      scx-activation: sigmoid
      scx-column: scx-ordinal
      scx-dimension: 15
      scx-loss: binary_crossentropy
      scx-metrics: mse
      scx-weight: 50

    predictions:
      continues:
        - rp
      # simply write fractions: [] if no fraction prediction is desired
      fractions: [scx]


In this config, 15 fractions (or pools) were acquired. While RP prediction is modeled as regression
problem, the SCX prediction is handled as ordinal regression. This type of regression performs
classification but the magnitude of the classification errors is taken into account. E.g. in normal
classification it does not make a difference if an observed PSM in fraction 5, got predicted to
elute in fraction 10 or in fraction 4. The error would just count as *false classification*.
However, in ordinal regression the margin of error is incorporated to the loss function and thus
(theoretically) ordinal regression should perform better than classification. The weight here defines
how the losses from the two prediction tasks are added to derive the final loss. This parameter
needs to be adapted for differences in scale and type of the output.

2D RT Prediction - Classification Task
''''''''''''''''''''''''''''''''''''''

Despite the theoretical advantage of ordinal regression, classification also delivered good
results during the development of xiRT. Therefore, we kept this as an option.

For this experimental setup, the xiRT config could look like this::

    output:
      rp-activation: linear
      rp-column: rp
      rp-dimension: 1
      rp-loss: mse
      rp-metrics: mse
      rp-weight: 1

      scx-activation: softmax
      scx-column: scx_1hot
      scx-dimension: 15
      scx-loss: categorical_crossentropy
      scx-metrics: accuracy
      scx-weight: 50

    predictions:
      continues:
        - rp
      # simply write fractions: [] if no fraction prediction is desired
      fractions: [scx]

Here we have the same experimental setup as above but the scx prediction task is modeled
as classification. For classification, the activation function, column name and loss function must be defined as in the
example above.

Transfer Learning
'''''''''''''''''
xiRT supports multiple types of transfer-learning. For instance,
training with the exact same architecture (dimensions, sequence lengths) on a data set (e.g. BS3
crosslinked) and then fine-tune the learned weights on the actual data set (e.g. DSS crosslinked)
is possible.
This requires a simple change in the learning (-l parameter) config file:
The *pretrained_model* parameter needs to be adapted for the location of the weights file from the BS3 model.

Optionally, the underlying model can be changed even more. This might become necessary when the
training was done with e.g. 10 fractions but only 5 got selected for new data acquisition. In this
scenario, the weights from the last layers cannot be used. Instead, the *pretrained_weights* and
the *pretrained_model* parameters need to be defined in the learning (-l) config.

The files in the repository ("sample_data" and "DSS_transfer_learning_example" folder)
provide examples on how to achieve transfer learning. Two calls to xiRT are necessary:

**Example:**

1) Train the reference model without any crossvalidation:

.. code-block:: console

    >xirt -i sample_data\DSS_xisearch_fdr_CSM50percent.csv \
    -x sample_data\xirt_params_3RT_best_ordinal.yaml \
    -l sample_data\learning_params_training_nocv.yaml \
    -o models/3DRT_full_nocv

2) Use the model for the transfer-learning:

.. code-block:: console

    >xirt -i sample_data\DSS_xisearch_fdr_CSM50percent_transfer_scx17to23_hsax2to9.csv \
    -x models/3DRT_full_nocv/callbacks/xirt_params_3RT_best_ordinal_scx17to23_hsax2to9.yaml \
    -l models/3DRT_full_nocv/callbacks/learning_params_training_nocv_scx17to23_hsax2to9.yaml \
    -o models\3DRT_transfer_dimensions

Further extensions
''''''''''''''''''

To further expand the prediction tasks, two more steps need to be taken.
First, the *predictions* section needs to be adapted such that a list of values, for example, [scx, hsax] is supplied.
Second, each entry in the *predictions* section needs to have a matching set of entries in the *output*
section. Carefully adjust the combination of activation, loss and column parameters as shown above.
xiRT allows to have 3x regression tasks, 1x regression task + 1x classification task, etc.

In principle the learning and prediction is agnostic to input data type. That means
that not only RTs can be learned but also other experimentally observed properties. Simply follow
the notation and decoding of the training parameters to add additional separation method columns.

Note
''''
It is important to follow the conventions above. Otherwise, learning results might vary a lot.

For classification, always use the following setup:

.. code-block:: console

    output:
        scx-activation: softmax
        scx-column: scx_1hot
        scx-dimension: 15
        scx-loss: categorical_crossentropy
        scx-metrics: accuracy

For **ordinal regression** always use the following setup:

.. code-block:: console

    output:
        scx-activation: sigmoid
        scx-column: scx_ordinal
        scx-dimension: 15
        scx-loss: binary_crossentropy
        scx-metrics: mse

For **regression** always use the following setup:

.. code-block:: console

    output:
        rp-activation: linear
        rp-column: rp
        rp-dimension: 1
        rp-loss: mse
        rp-metrics: mse
