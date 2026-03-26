---
modified:
  - 2026-03-26T13:40:18+01:00
created: 2026-03-26T13:40:06+01:00
---
Hide navigation sidebar

Hide table of contents sidebar

[Skip to content](https://emdgroup.github.io/baybe/stable/userguide/transfer_learning.html#furo-main-content)

[

![Light Logo](https://emdgroup.github.io/baybe/stable/_static/logo1.svg)

](https://emdgroup.github.io/baybe/stable/index.html)

- [Version: stable](https://emdgroup.github.io/baybe/versions)

Documentation

- [Readme](https://emdgroup.github.io/baybe/stable/index.html)
- [User Guide](https://emdgroup.github.io/baybe/stable/userguide/userguide.html)
    - [Getting Recommendations](https://emdgroup.github.io/baybe/stable/userguide/getting_recommendations.html)
    - [Campaigns](https://emdgroup.github.io/baybe/stable/userguide/campaigns.html)
    - [Active Learning](https://emdgroup.github.io/baybe/stable/userguide/active_learning.html)
    - [Asynchronous Workflows](https://emdgroup.github.io/baybe/stable/userguide/async.html)
    - [Constraints](https://emdgroup.github.io/baybe/stable/userguide/constraints.html)
    - [Environment Vars](https://emdgroup.github.io/baybe/stable/userguide/envvars.html)
    - [Insights](https://emdgroup.github.io/baybe/stable/userguide/insights.html)
    - [Objectives](https://emdgroup.github.io/baybe/stable/userguide/objectives.html)
    - [Parameters](https://emdgroup.github.io/baybe/stable/userguide/parameters.html)
    - [Recommenders](https://emdgroup.github.io/baybe/stable/userguide/recommenders.html)
    - [Search Spaces](https://emdgroup.github.io/baybe/stable/userguide/searchspace.html)
    - [Serialization](https://emdgroup.github.io/baybe/stable/userguide/serialization.html)
    - [Simulation](https://emdgroup.github.io/baybe/stable/userguide/simulation.html)
    - [Surrogates](https://emdgroup.github.io/baybe/stable/userguide/surrogates.html)
    - [Targets](https://emdgroup.github.io/baybe/stable/userguide/targets.html)
    - [Transformations](https://emdgroup.github.io/baybe/stable/userguide/transformations.html)
    - [Transfer Learning](https://emdgroup.github.io/baybe/stable/userguide/transfer_learning.html#)
    - [Utilities](https://emdgroup.github.io/baybe/stable/userguide/utils.html)
- [Examples](https://emdgroup.github.io/baybe/stable/examples/examples.html)
- [FAQ](https://emdgroup.github.io/baybe/stable/faq.html)

- [baybe](https://emdgroup.github.io/baybe/stable/_autosummary/baybe.html)

Misc

- [Contribute](https://emdgroup.github.io/baybe/stable/misc/contributing_link.html)
- [Contributors](https://emdgroup.github.io/baybe/stable/misc/contributors_link.html)
- [Known Issues](https://emdgroup.github.io/baybe/stable/known_issues.html)
- [Changelog](https://emdgroup.github.io/baybe/stable/misc/changelog_link.html)
- [Github](https://github.com/emdgroup/baybe/)
- [License](https://emdgroup.github.io/baybe/stable/misc/license_link.html)

[Back to top](https://emdgroup.github.io/baybe/stable/userguide/transfer_learning.html#)

[View this page](https://emdgroup.github.io/baybe/stable/_sources/userguide/transfer_learning.md.txt "View this page")

Toggle Light / Dark / Auto color theme

# Transfer Learning

BayBE offers the possibility to mix data from multiple, _similar but not identical_ campaigns in order to accelerate optimization – a procedure called **transfer learning**. This feature is automatically enabled when using a [Gaussian process surrogate model](https://emdgroup.github.io/baybe/stable/_autosummary/baybe.surrogates.gaussian_process.core.GaussianProcessSurrogate.html#baybe.surrogates.gaussian_process.core.GaussianProcessSurrogate "baybe.surrogates.gaussian_process.core.GaussianProcessSurrogate") in combination with a [`TaskParameter`](https://emdgroup.github.io/baybe/stable/_autosummary/baybe.parameters.categorical.TaskParameter.html#baybe.parameters.categorical.TaskParameter "baybe.parameters.categorical.TaskParameter").

Terminology

In the scientific community, the term **transfer learning** is used in many different ways. Within BayBE, it refers to combining data from multiple campaigns that are from similar contexts, which we also refer as **tasks**. Depending on the field, this might also be known as **contextual learning**.

## Unlocking Data Treasures Through Transfer Learning

A straightforward approach to combining data from different campaigns is to quantify the differences between their contexts via one or few explicitly measured parameters and then constraining these parameters in the active campaign to the relevant context.

Examples where this is possible:

- **Optimization of a chemical reaction at different temperatures:**  
    Data obtained from a chemical reaction optimized at a certain temperature can be used in a new campaign, where the same reaction needs to be optimized again at a different temperature.
    
- **Optimization of a simulation involving a particle size:**  
    Data obtained at a smaller particle size can be utilized when starting a new optimization for a larger particle size or vice versa.
    

In these examples, the temperature and the particle size take the role of _aligning_ the individual measurement campaigns along their corresponding context dimension. That is, the context is static _within_ each campaign (i.e., each campaign is executed at its fixed context parameter value) but the parameter establishes an explicit relationship between the data gathered _across_ campaigns. Transfer of knowledge from one campaign to another can thus simply happen through the existing mechanisms of a surrogate model by feeding the context parameter as an additional regular input to the model.

Terminology Continued

Technically speaking, the above mechanism already enables information transfer across campaigns. However, the information transfer is established through the regular generalization capabilities of the involved surrogate and does not require extending the model architecture itself. Therefore, to distinguish from the approach detailed below, we do **not** refer it as transfer learning.

Unfortunately, there are many situations where it can be difficult to quantify the differences between the campaigns via explicit context parameters in the first place. This might be the case if the parameters distinguishing the contexts

1. have not been recorded and cannot be measured anymore,
    
2. are too many and explicitly modelling them is out of question or
    
3. are simply unknown.
    

Examples for situations where explicit quantification of the context can be difficult:

- **Cell culture media optimization for different cell types:**  
    Cell types differ among many possible descriptors, and it is not known a priori which ones are relevant to a newly started campaign.
    
- **Optimization in industrial black-box contexts:**  
    When materials (such as cell lines or complex substances) stem from customers, they can come uncharacterized.
    
- **Transfer of a complicated process to another location:**  
    The transferred machinery will likely require a new calibration/optimization, which could benefit from the other location’s data. However, is not necessarily clear what parameters differentiate the location context.
    

**Transfer learning** in BayBE offers a solution for situations such as the latter, because it abstracts each context change between campaigns into a single dimension encoded by a [`TaskParameter`](https://emdgroup.github.io/baybe/stable/_autosummary/baybe.parameters.categorical.TaskParameter.html#baybe.parameters.categorical.TaskParameter "baybe.parameters.categorical.TaskParameter"). Over the course of an ongoing campaign, the relationship between current campaign data and data from previous campaigns can then be _learned_ instead of requiring hard-coded context parameters, effectively enabling you to utilize your previous data through an additional machine learning model component. In many situations, this can unlock data treasures coming from similar but not identical campaigns accumulated over many years.

Expectations

Because of the need to _learn_ the relationship between tasks, transfer learning is not a magic method for zero-shot learning. For effective information transfer, it will always need data from the ongoing campaign to understand how other campaigns’ data are related. Otherwise, it can only build upon general patterns/trends identified in the previous campaigns, without knowing if these patterns actually reoccur in the new campaign. (**Note:** This can still help to jump-start the new campaign since the most influential parameter configurations from old campaigns will then drive the initial exploration.) Overall, if correlated task data are provided, the optimization of new campaigns can experience a dramatic speedup.

Adding Irrelevant Data

Because of the ability to learn the task relationships, it might be tempting to add arbitrary data to a transfer learning enabled campaigns. We caution against this, as uncorrelated data can actually decrease the performance of the optimization. Even a simple preliminary correlation filter to find suitable contexts can already increase robustness.

## The Role of the TaskParameter

The [`TaskParameter`](https://emdgroup.github.io/baybe/stable/_autosummary/baybe.parameters.categorical.TaskParameter.html#baybe.parameters.categorical.TaskParameter "baybe.parameters.categorical.TaskParameter") is used to “mark” the context of individual experiments and thus to “align” different campaigns along their context dimension. The set of all possible contexts is provided upon the initialization of a [`TaskParameter`](https://emdgroup.github.io/baybe/stable/_autosummary/baybe.parameters.categorical.TaskParameter.html#baybe.parameters.categorical.TaskParameter "baybe.parameters.categorical.TaskParameter") by providing them as `values`. In the following example, the context might be one of several reactors in which a chemical experiments can be conducted.

from baybe.parameters import TaskParameter

TaskParameter(name="Reactor", values=["ReactorA", "ReactorB", "ReactorC"])

If not specified further, a campaign using the [`TaskParameter`](https://emdgroup.github.io/baybe/stable/_autosummary/baybe.parameters.categorical.TaskParameter.html#baybe.parameters.categorical.TaskParameter "baybe.parameters.categorical.TaskParameter") as specified above would now make recommendations for all possible values of the parameter. Using the `active_values` argument upon initialization, this behavior can be changed such that the `campaign` only makes recommendations for the corresponding values.

The following example models a situation where experimentation data from three different reactors are available, but new experiments should only be conducted in `ReactorC`.

from baybe.parameters import TaskParameter

TaskParameter(
    name="Reactor",
    values=["ReactorA", "ReactorB", "ReactorC"],
    active_values=["ReactorC"],
)

The same pattern can be easily applied to other scenarios such as changing substrates (while screening the same reaction conditions) or formulating mixtures for different cell lines:

TaskParameter(
    name="Substrate",
    values=["3,5-dimethylisoxazole", "benzo[d]isoxazole", "5-methylisoxazole"],
    active_values=["3,5-dimethylisoxazole"],
)
TaskParameter(
    name="Cell_Line",
    values=["Liver cell", "Heart cell", "Hamster brain cell"],
    active_values=["Liver cell"],
)

Technology

Note that the transfer learning mechanism can be implement in many different ways, for example, using multiple models, shared architectures, special kernels in a single model, etc. Encoding the context information via a [TaskParameter] is not restricted to any of these methods, even though BayBE currently only implements a variant via kernels, specifically: the Intrinsic Coregionalization Model [[BCW07](https://emdgroup.github.io/baybe/stable/userguide/transfer_learning.html#id3 "Edwin V Bonilla, Kian Chai, and Christopher Williams. Multi-task gaussian process prediction. In J. Platt, D. Koller, Y. Singer, and S. Roweis, editors, Advances in Neural Information Processing Systems, volume 20. Curran Associates, Inc., 2007.")].

## Seeing Transfer Learning in Action

A full example demonstrating BayBE’s transfer learning capabilities can be found [here](https://emdgroup.github.io/baybe/stable/examples/Transfer_Learning/basic_transfer_learning.html). Nonetheless, we would like to briefly highlight the results of this example in the user guide.

The goal in the example is to optimize an analytical function. We apply transfer learning by providing additional data that was obtained from evaluating a negated noisy variant of the same function.

The following plot demonstrates the effect that providing this additional data has on the optimization:

![../_images/basic_transfer_learning_light1.svg](https://emdgroup.github.io/baybe/stable/_images/basic_transfer_learning_light1.svg)

[[BCW07](https://emdgroup.github.io/baybe/stable/userguide/transfer_learning.html#id1)]

Edwin V Bonilla, Kian Chai, and Christopher Williams. Multi-task gaussian process prediction. In J. Platt, D. Koller, Y. Singer, and S. Roweis, editors, _Advances in Neural Information Processing Systems_, volume 20. Curran Associates, Inc., 2007.

[

Next

Utilities

](https://emdgroup.github.io/baybe/stable/userguide/utils.html)[

Previous

Transformations



](https://emdgroup.github.io/baybe/stable/userguide/transformations.html)

Copyright © 2022-2025 Merck KGaA, Darmstadt, Germany and/or its affiliates. All rights reserved.

Made with [Sphinx](https://www.sphinx-doc.org/) and [@pradyunsg](https://pradyunsg.me/)'s [Furo](https://github.com/pradyunsg/furo)

On this page

- - [Unlocking Data Treasures Through Transfer Learning](https://emdgroup.github.io/baybe/stable/userguide/transfer_learning.html#unlocking-data-treasures-through-transfer-learning)
    - [The Role of the TaskParameter](https://emdgroup.github.io/baybe/stable/userguide/transfer_learning.html#the-role-of-the-taskparameter)
    - [Seeing Transfer Learning in Action](https://emdgroup.github.io/baybe/stable/userguide/transfer_learning.html#seeing-transfer-learning-in-action)