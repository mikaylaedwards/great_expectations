.. _validation_operators_and_actions:

.. warning::

  As part of the new modular expectations API in Great Expectations, Validation Operators are being folded into the Checkpoints functionality. We will continue to update this documentation as that change is made broadly available.


#############################################
Validation Operators And Actions Introduction
#############################################

The `validate` method evaluates one batch of data against one expectation suite and returns a dictionary of validation results. This is sufficient when you explore your data and get to know Great Expectations.
When deploying Great Expectations in a real data pipeline, you will typically discover additional needs:

* validating a group of batches that are logically related
* validating a batch against several expectation suites
* doing something with the validation results (e.g., saving them for a later review, sending notifications in case of failures, etc.).

Validation Operators provide a convenient abstraction for both bundling the validation of multiple expectation suites and the actions that should be taken after the validation.

***************************************************
Finding a Validation Operator that is right for you
***************************************************

Each Validation Operator encodes a particular set of business rules around validation. Currently, two Validation Operator implementations are included in the Great Expectations distribution.

The classes that implement them are in :py:mod:`great_expectations.validation_operators` module.

.. _action_list_validation_operator:

ActionListValidationOperator
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

ActionListValidationOperator validates each batch in the list that is passed as `assets_to_validate` argument to its `run` method against the expectation suite included within that batch and then invokes a list of configured actions on every validation result.

Actions are Python classes with a `run` method that takes a result of validating a batch against an expectation suite and does something with it (e.g., save validation results to the disk or send a Slack notification). Classes that implement this API can be configured to be added to the list of actions for this operator (and other operators that use actions). Read more about actions here: :py:class:`great_expectations.validation_operators.actions.ValidationAction`.

An instance of this operator is included in the default configuration file `great_expectations.yml` that `great_expectations init` command creates.

A user can choose the actions to perform in their instance of the operator.

Read more about ActionListValidationOperator here: :py:class:`great_expectations.validation_operators.ActionListValidationOperator`.

.. warning_and_failure_expectation_suites_validation_operator:

WarningAndFailureExpectationSuitesValidationOperator
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

WarningAndFailureExpectationSuitesValidationOperator implements a business logic pattern that many data practitioners consider useful.

It validates each batch of data against two different expectation suites: "failure" and "warning". Group the expectations that you create about a data asset into two expectation suites. The critical expectations that you are confident that the data should meet go into the expectation suite that this operator calls "failure" (meaning, not meeting these expectations should be considered a failure in the pipeline). The rest of expectations go into the "warning" expectation suite.

The failure Expectation Suite contains expectations that are considered important enough to justify stopping the pipeline when they are violated.

WarningAndFailureExpectationSuitesValidationOperator retrieves two expectation suites for every data asset in the `assets_to_validate` argument of its `run` method.

After completing all the validations, it sends a Slack notification with the success status. Note that it doesn't use an Action to send its Slack notification, because the notification has also been customized to summarize information from both suites.

Read more about the :py:class:`~great_expectations.validation_operators.validation_operators.WarningAndFailureExpectationSuitesValidationOperator` here.

.. _validation_actions:

*****************
ValidationActions
*****************

The Validation Operator implementations above invoke actions.

An action is a way to take an arbitrary method and make it configurable and runnable within a data context.

The only requirement from an action is for it to have a `run` method.  The `run` method will consume and return a `payload`, which can be used to pass information from one ValidationAction to another. The `payload` is a dictionary that will have the name of the ValidationAction as a key, and a nested dictionary containing the `class` of the ValidationAction and any additional parameters returned by the ValidationAction as its value.

GE comes with a list of actions that we consider useful and you can reuse in your pipelines. Most of them take in validation results and do something with them.
