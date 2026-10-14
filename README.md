# Hotel Booking Cancellation Prediction --- Altair AI Studio Workflow

## Overview

This repository contains an Altair AI Studio (`.rmp`) process for
comparing classification models that predict whether a hotel booking
will be canceled. The target attribute is `is_canceled`.

The workflow retrieves the `hotel_bookings` dataset from the Altair AI
Studio repository and evaluates several classical machine-learning
models and three neural-network configurations.

## Dataset

-   **Dataset name:** `hotel_bookings`
-   **Repository entry in the process:** `../data/hotel_bookings`
-   **Prediction target / label:** `is_canceled`
-   **Task:** Binary classification --- predict whether a booking is
    canceled.
-   **Note:** The process file references a repository dataset; it does
    not embed the dataset itself. Make sure `hotel_bookings` is
    available at the expected repository location before running the
    process.

## Workflow at a glance

1.  Retrieve `hotel_bookings`.
2.  Remove duplicate examples.
3.  Run **Generate Attributes**. 
4.  Remove `assigned_room_type`, `reservation_status`, and
    `reservation_status_date` using **Select Attributes**.
5.  Apply **Numerical to Binominal** to the configured numeric
    attributes.
6.  Set `is_canceled` as the label.
7.  Use **Multiply** to route the prepared data into the
    model-validation branches.
8.  Evaluate the classical models and neural-network configurations
    using split validation.
9.  Compare the resulting performance vectors in Altair AI Studio.

## Data preparation operators

  -----------------------------------------------------------------------
  Operator                            Configuration / purpose
  ----------------------------------- -----------------------------------
  **Retrieve**                        Loads `hotel_bookings` from
                                      `../data/hotel_bookings`.

  **Remove Duplicates**               Checks all regular attributes for
                                      duplicate examples. The saved
                                      configuration does not treat
                                      missing values as duplicates.

  **Generate Attributes**             Added in the Process
                                     

  **Select Attributes**               Excludes `assigned_room_type`,
                                      `reservation_status`, and
                                      `reservation_status_date`.

  **Numerical to Binominal**          Configured to process numeric
                                      attributes using the operator's
                                      saved value-type and range
                                      settings.

  **Set Role**                        Sets `is_canceled` to the `label`
                                      role.

  **Multiply**                        Distributes the prepared example
                                      set to multiple modeling branches.
  -----------------------------------------------------------------------

### Neural-network-specific preprocessing

The neural-network branch applies additional preprocessing before
training:

-   **Select Attributes (2):** excludes `company`, `agent`, and
    `country`.
-   **Replace Missing Values:** replaces missing values in `children`
    using the operator's configured default replacement method
    (`value`).
-   **Nominal to Numerical:** uses dummy coding.
-   **Normalize:** uses Z-transformation.
-   **Neural Net:** trains each configured network within its own
    validation process.

These neural-network-specific steps are not a description of the
preprocessing used by every classical model branch.

## Partition and reproducibility

Each saved **Split Validation** operator uses the following settings:

-   **Partition:** 80% training data / 20% test data.
-   **Sampling type:** automatic.
-   **Local random seed:** `6461`.
-   **Process-level random seed:** `6461`.

The 20% test partition is used by each validation branch to assess
predictions on held-out examples. For reproducibility, keep the seed and
partition settings unchanged when comparing models. The actual number of
examples in each partition depends on the number of rows remaining after
data preparation.

## Models evaluated

### Classical models

  -----------------------------------------------------------------------
  Model                               Saved configuration
  ----------------------------------- -----------------------------------
  **Logistic Regression**             H2O Logistic Regression operator;
                                      automatic solver; mean imputation
                                      configured for missing values;
                                      maximum iterations set to the
                                      operator default/unlimited setting.

  **Naive Baseline**                  Default Model using the `mode`
                                      method.

  **Naive Bayes**                     Naive Bayes classifier with its
                                      saved default configuration.

  **Decision Tree**                   Gain-ratio criterion; maximum depth
                                      `3`; minimum leaf size `2`; minimum
                                      size for split `4`; minimum gain
                                      `0.01`.

  **Random Forest**                   Three forest configurations are
                                      evaluated: `50`, `100`, and `200`
                                      trees. Each uses gain ratio,
                                      maximum depth `10`, subset ratio
                                      `0.2`, and confidence voting. 
  -----------------------------------------------------------------------

### Neural-network configurations

The neural-network branch uses three architectures:

  -----------------------------------------------------------------------------
  Configuration   Hidden        Training     Learning     Momentum   Local seed
                  layers          cycles         rate              
  --------------- --------- ------------ ------------ ------------ ------------
  **NN A**        8 neurons          200         0.01          0.9         6461

  **NN B**        16                 200         0.01          0.9         6461
                  neurons                                          

  **NN C**        16 → 8             200         0.01          0.9         6461
                  neurons                                          
  -----------------------------------------------------------------------------

All three configurations use dummy coding for nominal attributes and
Z-transformation in their validation subprocesses. The neural-network
operator has shuffling enabled and error epsilon `1.0E-4`. The
operator's own `normalize` option is disabled because a separate
**Normalize** operator is present in the preprocessing chain.

## Evaluation

Each model branch applies the trained model to the held-out partition
and uses a performance operator to produce evaluation results.

Performance results can be reviewed in Altair AI Studio's Results view.
Depending on the performance operator and its settings, review the
available measures such as:

-   Accuracy
-   AUC, when available/configured
-   Precision
-   Recall
-   Confusion matrix (TP, FP, FN, TN)

Use the same data split and evaluation definitions when comparing
models. This README documents the process configuration; it does not
claim a particular model is the winner or report performance values.

## How to run

1.  Open Altair AI Studio.
2.  Confirm the `hotel_bookings` dataset is available in the repository
    at `../data/hotel_bookings`.
3.  Open the `.rmp` process file.
4.  Check that the label is `is_canceled` and that the validation
    settings are still 80/20 with seed `6461`.
5.  Run the process.
6.  Open the result tabs for the model performance vectors and compare
    the metrics.

## Notes and limitations

-   The `.rmp` file references a repository dataset; the data file
    itself may need to be supplied separately.
-   The workflow is a model-comparison exercise. The best model should
    be selected based on the actual validation results and the purpose
    of the analysis, not on model type alone.
