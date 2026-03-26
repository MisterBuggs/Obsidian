---
modified:
  - 2026-03-26T15:31:38+01:00
created: 2026-03-26T15:27:47+01:00
---
Below is how each manuscript relates to your optimization problem and what you could appropriate from it. I’ll focus on aspects most relevant to noisy, expensive, multi‑factor biological optimization with possible “dead runs” and robustness across cell lines.

---

## Daulton et al. – Robust Multi-Objective Bayesian Optimization Under Input Noise (MARS / MVAR)

**How it relates**

- Directly addresses robust optimization under **input noise**, i.e. settings where the implemented conditions (your gas pressures, growth factor concentrations, etc.) deviate from the nominal values due to uncontrolled perturbations.
- Treats **multiple objectives and constraints**, both affected by input noise, and defines a robust analogue of the Pareto frontier (the **MVAR set**, multivariate value-at-risk).
- Explicitly models **black-box constraints** that can become infeasible under small perturbations in inputs—a strong analogue of your “dead runs” where cells die under certain parameter combinations.
- Provides methods that are **sample-efficient** and also have **parallel batch** variants—important given your 4-experiment batches and long run times.

**What you can appropriate**

Conceptual / modeling aspects:

- Think of your **culture protocol plus implementation variability** as:
  - Nominal design: x (e.g. setpoints of O₂, CO₂, growth factor concentrations, media ratios).
  - Perturbed implementation: x ⊕ ξ, where ξ describes the distribution of deviations (pipetting variation, gas regulation tolerances, etc.).
- View your experimental readout as a stochastic response:
  - Main outcome: gene expression y(x ⊕ ξ).
  - Constraint: viability / successful culture c(x ⊕ ξ) > 0 (no “dead run”).
- You likely want robust designs that:
  - Maximize gene expression across cell lines, **subject to** high probability of viability.
  - Are not just high at the nominal x but maintain performance when x is perturbed.

Specific robustness criterion:

- They use **value-at-risk (VAR)** and its multivariate version (MVAR):
  - In your single main outcome case, scalar VAR is also applicable.
  - For robustness you might specify: “Choose conditions x that maximize the α‑VAR of gene expression,” i.e., choose x such that the **worst α‑quantile** of expression under input perturbations is as high as possible, while viability holds with probability ≥ α.
  - If you treat multiple objectives (e.g. expression in several cell lines, or expression vs. proliferation), MVAR becomes directly applicable: find designs that maximize a **set** of robust trade-offs that are met with probability ≥ α.

Dead runs / viability:

- They explicitly consider **feasibility-weighted objectives**: multiply objectives by an indicator of constraints being satisfied. This is a clean way to include cell death:
  - Objective ~ expression(x ⊕ ξ) × 1[viability(x ⊕ ξ) > 0].
  - Designs near viability boundaries may have good nominal expression but poor **yield** once perturbations are accounted for—a direct analog of your “works only in a narrow region” situation.
- Their examples (e.g. disc brake, penicillin production) show how nominally optimal solutions can have poor yield once input noise and constraints are considered—a pattern you should expect in your cultures.

Algorithmic ideas you can reuse conceptually:

- **MARS (MVAR Approximation via Random Scalarizations)**:
  - They show that instead of optimizing MVAR directly (which is computationally prohibitive), you can:
    - Randomly pick a scalarization (for single objective, this can be trivial; for multiple objectives, a Chebyshev scalarization).
    - Optimize the **VAR** of this scalarized objective with a standard single‑objective BO engine.
  - For you: if you decide to treat gene expression in several cell lines as separate objectives, MARS provides a practical pattern: multi-objective, robust to input perturbations, using standard BO machinery.

- **Noisy acquisition design**:
  - They work with **noisy inputs + noisy outputs** and demonstrate variants of Expected Improvement and Upper Confidence Bound adapted to VAR and MVAR.
  - They consider **parallel batch acquisition** (q‑NEI variants), relevant to your limited parallelization.

Practical behaviors relevant to biology:

- They explicitly warn that optimizing **expectation** (mean performance) often yields poor robust performance in presence of sharp regions and asymmetric noise; your description of narrow peaks and large dead regions matches this. Their results show:
  - Expectation-based methods can converge to regions that look good on average but are highly sensitive to perturbations.
  - VAR/MVAR-based methods tend to pick more conservative but much more **reliable** operating points.
- They discuss how robust solutions often live **away from the nominal optimum**, trading some peak performance for insensitivity to input perturbations. This parallels moving a culture condition away from the critical edge of viability to get more consistent results across cell lines and day-to-day variability.

Methodological choices for your context:

- Use a **Gaussian Process (GP)** surrogate with input noise modeled explicitly (you can specify or estimate ξ from lab tolerances).
- Incorporate **black-box viability constraints** into BO:
  - Either model viability as a separate GP and use feasibility-weighted objectives (set expression to zero when viability is predicted unlikely).
  - Or model viability probability and enforce constraints in the acquisition.
- Use a **risk parameter α** with a clear experimental interpretation:
  - e.g. α = 0.8: “We want conditions such that at least 80% of runs (across perturbations / days) achieve at least this expression level and remain viable.”

---

## Slautin et al. – Measurements with Noise: Bayesian Optimization for Co-Optimizing Noise and Property Discovery in Automated Experiments

**How it relates**

- Focuses on **measurement noise** and the trade-off between:
  - Experimental duration / cost (e.g. exposure time in microscopy, number of repetitions),
  - And **signal-to-noise ratio** of the measured property.
- Uses BO to **simultaneously optimize the target property and measurement settings**, with special emphasis on automated workflows.
- Their setting is 1D physical parameter + measurement time, but the principle is directly relevant: in your cell culture, measurement noise can be tuned via:
  - Number of cells sampled,
  - Number of technical replicates,
  - Assay exposure times, etc.

**What you can appropriate**

Co-optimization of property and measurement protocol:

- Rather than treat “measurement reliability” as fixed, you can explicitly include **measurement time / number of replicates** as a design dimension:
  - v = [ culture conditions x, measurement settings t ], where t might be:
    - qPCR cycles / read time,
    - Number of wells per condition,
    - Number of technical replicates,
    - For imaging-based assays, exposure time or number of frames.
- Their approach models the **measurement noise level as a function of t** (e.g., noise ∼ A 1 / √ t + A 0), allowing BO to learn:
  - Where extra measurement time drastically reduces uncertainty,
  - Where additional time gives diminishing returns.

Reward-based noise optimization:

- They define a **reward function** R(t, noise) that trades off:
  - Longer measurement duration (worse: higher cost),
  - Higher noise (worse: lower SNR).
- They then use this reward to weight a standard acquisition function for the property of interest.
- For your setting, you could design an analogous reward:
  - R(number of replicates, estimated SE(expression)) that balances:
    - More replicates against the opportunity cost of using those wells / time for exploring new conditions.

Double-optimization acquisition:

- They show a “double-optimization” approach combining:
  - An acquisition for the **target property** (e.g., gene expression),
  - An acquisition for **measurement time / noise** based on the reward.
- This is relevant if you want explicit **BO-driven decisions about how many replicates to use at each new condition**, not just hand-tune replication.

Use of structured GP for noise:

- They distinguish between:
  - A primary GP for the underlying property f(x),
  - A separate noise model GP for Noise_f(t), learning noise as a function of measurement time.
- This is a useful pattern if you want to:
  - Model **heteroskedastic measurement noise** (variance depends on t, or on expression level, or both),
  - And incorporate this noise model into acquisition functions like NEI / UCB.

Practical implications for your experiments:

- If gene expression measurements are expensive and variable, Slautin et al.’s ideas suggest you should:
  - Not fix replication a priori.
  - Let the BO algorithm **adapt measurement effort**:
    - Early in the campaign: cheaper, noisier measurements to explore condition space faster.
    - Near promising optima: additional replicates or longer measurements to reduce uncertainty, particularly important when decisions will affect future large-scale expansion under those conditions.
- Their work also shows experimentally how controlling measurement noise can reduce the **risk of mis-ranking** good vs. bad conditions—critical in a biological setting with high variance and occasional failures.

---

## Chen & Han – Design Optimization for Robust Engineering Design with Noise Factors (Bayesian Optimization with General Loss Functions)

*(The text is garbled but enough structure remains to see the main ideas.)*

**How it relates**

- Focused on **robust parameter design** (Taguchi-style) with **noise factors**—uncontrolled variables that affect performance.
- Uses **Bayesian optimization** to optimize a **general loss function** rather than just a quadratic loss.
  - Loss is a function of the deviation of the outcome from target, possibly non-quadratic and bounded.
- Includes **noise factors** explicitly, draws Monte Carlo samples of them, and uses EI and UCB-like acquisition functions adapted to this robust loss.
- This is conceptually aligned with your need to design conditions robust to:
  - Environmental or assay-to-assay variation,
  - Batch-to-batch variability in media, gas regulation, etc.

**What you can appropriate**

Robust loss functions:

- They generalize beyond the standard quadratic loss L(y) = (y − target)² and use **bounded, problem-specific loss** built from a probability density function g(y), with a maximum value M.
- For your gene expression:
  - You might wish to define a **loss that saturates** once expression is above a biologically meaningful threshold, or:
  - A loss that heavily penalizes low expression and zero viability but is indifferent between “very high” and “extremely high”.
- Their framework allows you to define such a **non-quadratic loss** and then perform BO on the **expected loss over noise factors**.

Noise factors and Monte Carlo integration:

- They distinguish between:
  - Control variables (your design x),
  - Noise variables (e.g. environmental fluctuations, operator variability).
- Their method:
  - Samples noise variables (like your ξ),
  - Computes the **expected loss** E_noise[L(f(x, noise))] via Monte Carlo for each candidate x,
  - Uses **EI (EI-GL) and UCB (UCB-GL)** acquisition functions adapted to this expected general loss.
- You can adopt the idea of:
  - Treating viability failures and poor expression as a unified **loss**,
  - Integrating over plausible environmental / implementation noise when computing that loss.

Cost-aware sampling over noise:

- They discuss Monte Carlo sampling cost for computing acquisition functions.
- Although they focus on small problems, the idea generalizes: in your setting you might not know the noise distribution explicitly, but you could:
  - Use **empirical noise models** (from replicated runs) to approximate ξ,
  - And then approximate E[L] using a limited number of noise samples per candidate.

Interpretation for your “dead runs”:

- By embedding **cell death** into the loss (e.g. large penalty when viability is zero), and integrating over noise factors, their framework ensures that conditions that are close to lethal get down-weighted even if nominal expression is high.
- Unlike expectation-based objectives on expression alone, the loss-based view lets you encode the **experimental preference**: “A slightly lower expression with robust survival is better than high expression with high probability of culture failure.”

This paper doesn’t address multi-objective robustness or advanced GP acquisition details as fully as Daulton et al., but it gives a more flexible, Taguchi-consistent way to define **what “robust performance” means** via a general loss function, which you could adapt to your biology-specific priorities.

---

## Letham et al. – Constrained Bayesian Optimization with Noisy Experiments (Noisy EI / NEI)

**How it relates**

- Directly addresses **Bayesian optimization with noisy observations and noisy constraints**, where:
  - Outcomes are estimated from randomized experiments or noisy measurements.
  - Constraints (e.g. quality, latency) are also measured with noise.
- Introduces **Noisy Expected Improvement (NEI)**, which integrates over the uncertainty in past observations instead of plugging in noisy best values.
- Derives a **constrained** version of NEI and shows how to handle **batch** evaluations (parallelism).
- Their setting (A/B tests at Facebook) maps well to your situation:
  - Each experiment is expensive,
  - Measurements are noisy,
  - Constraints (e.g., viability) can be violated and are only known with noise,
  - You can run small batches of parallel experiments.

**What you can appropriate**

Handling noisy outcomes and constraints:

- The core conceptual idea is to treat:
  - The underlying objective f and constraint c as latent (unknown true values),
  - Your observed y and ĉ as noisy estimates of these,
  - And integrate acquisition functions over the posterior of f_n and c_n, not over point estimates.
- Specifically for you:
  - Let y_i be measured gene expression (possibly aggregated from multiple replicates).
  - Let viability / success be ĉ_i (proportion of wells surviving, for example).
  - Use NEI to:
    - Integrate over GP uncertainty in f and c,
    - And select new conditions that best balance exploration and exploitation while accounting for noise in **both** expression and viability.

Constrained EI formulation:

- They show a constrained EI where improvement is set to zero when the candidate is infeasible; with noisy constraints, they:
  - Replace unknown “best feasible value” with a distribution, and
  - Integrate EI over the posterior of the latent feasible points.
- For your cell culture:
  - You can target “improvement in expression over best feasible condition so far,” where feasibility is “probability of viability > threshold”.

QMC-based NEI:

- NEI requires integrating EI over GP posterior of f and c at previously evaluated points.
- They propose a **quasi-Monte Carlo (QMC)** approximation that:
  - Dramatically reduces the number of samples needed compared to plain Monte Carlo,
  - Makes NEI practical when you have dozens of past experiments and must evaluate the acquisition many times during optimization.
- This is exactly the regime you’re in: very expensive experiments, relatively few data points, but high noise.

Batch / asynchronous evaluation:

- They extend EI and NEI to **batch selection**, where you pick multiple new candidates before seeing their outcomes.
- That matches your requirement to run up to 4 cultures in parallel each batch.

How this differs from Daulton et al.:

- Letham et al. handle **output noise** and **noisy constraints** robustly, but do not explicitly model **input noise** (perturbations around setpoints) or multi-objective robustness via VAR/MVAR.
- Daulton et al. build on NEI-style ideas but focus explicitly on **robustness to input noise** and on multi-objective settings.
- For you, a natural combination is:
  - Use a **NEI-type acquisition** that integrates over noisy past outcomes and viability,
  - While modeling **input noise** (tolerances on gas / concentration control) and/or using VAR-style objectives if you want strong robustness.

---

## Martinelli et al. (SADCBO) – Learning Relevant Contextual Variables Within Bayesian Optimization

**How it relates**

- Addresses cases where:
  - There are **contextual variables** (environmental conditions, covariates).
  - Only some of them are relevant to the target function.
  - Contextual variables may, in principle, be **optimized**, but at an extra cost.
- Introduces **SADCBO**, which simultaneously:
  - Learns which contextual variables are important (via sensitivity analysis on the GP),
  - Decides whether to optimize them or just observe them,
  - And uses an **early stopping criterion** for when to start actively controlling contextual variables.
- This maps well to your setting if you consider:
  - Different cell lines, passage number, or medium batch as **contextual variables** rather than design parameters.
  - You may be able to systematically vary some of these (e.g. specific cell lines, known medium formulations), but doing so is costly.

**What you can appropriate**

Contextual vs. design variables:

- They distinguish:
  - Design variables x: parameters you always choose (for you: culture conditions).
  - Contextual variables z: environmental/biological factors not fully under control, but sometimes controllable (or at least known for each experiment).
- For your experiments, potential contextual variables:
  - Cell line identity,
  - Passage number / donor batch,
  - Incubator / plate position or other environmental differences,
  - Media lot, reagent batch, etc.

Sensitivity-based variable relevance:

- They use **Feature Collapsing** (Sebenius et al., 2022) to estimate, from the GP posterior, how much each contextual variable matters for high-performing points.
- Applied to your context:
  - You could learn whether the variation across cell lines is a dominant factor or minor compared to culture conditions.
  - You could detect if certain “secondary” contextual variables (e.g. passage number, medium lot) significantly influence robust gene expression.

Trade-off between exploring context and controlling it:

- Their framework explicitly trades off:
  - Cost of controlling a contextual variable,
  - Its inferred relevance to the outcome.
- For your experiments, this could inform decisions such as:
  - When to include additional cell lines in the optimization (treating “cell line” as a contextual factor that you actively sample).
  - When it’s enough to just **observe** which cell line is used in each experiment and adjust conditions accordingly.
- Their phased approach:
  - Early phase: only **observe** context (e.g. run your usual cell lines, record identity).
  - Later phase: once BO has learned the landscape, start **actively choosing** which contexts to include (e.g. deliberately test optimized conditions on the most informative / challenging cell lines).

Early stopping criterion and cost-aware design:

- They adopt an early stopping criterion based on changes in expected regret, which you could borrow conceptually to decide:
  - When to shift from broad exploration to focused testing across cell lines,
  - When additional costly experiments (e.g. on rare or expensive lines) are likely to be informative.

High-dimensional aspects:

- SADCBO is also designed for cases with many potential contextual variables. If you have many covariates (e.g. multiple cell line properties, multiple environmental / media batch variables) and are not sure which matter:
  - Their sensitivity-analysis-driven selection gives a principled way to prioritize which biological covariates to explicitly model in the BO.

---

Below are summaries of how each manuscript relates to your cell‑culture optimization problem, and what aspects you could reuse.

---

## Diessner et al. – ENVBO: Bayesian Optimization with Changing Environmental Conditions

**Relation to your problem**

- ENVBO is designed for optimization when some variables are **controllable** (design variables) and others are **uncontrollable environmental variables** that change from run to run (e.g., wind direction in their example).
- They fit a **single global GP surrogate over both controllable and environmental variables**, but each BO step only optimizes over the controllable ones, **conditioning on the measured environment**.
- This maps well to your situation if you treat things like:
  - Cell line identity, passage number, incubator conditions, lab temperature, etc., as **environmental/contextual variables** that you do not fully control.
  - Gas pressures, media composition, growth factor doses as **controllable design variables**.
- They explicitly analyze performance under **measurement noise**, multiple environmental variables, and different fluctuation/variability patterns—matching your noisy, variable biological setting.

**What you can appropriate**

- Treat your experiments as:
  - Input: (x_c, x_env), where x_c are culture conditions you can choose; x_env are measured but not controlled factors (e.g., line, batch, incubator, timing).
  - Response: gene expression (and viability).
- Use a **global GP over [x_c, x_env]**, but at each batch/iteration:
  - Measure the current environment x_env (which cell line / passage / incubator, etc.),
  - Optimize the acquisition function **conditionally on these observed environmental values**, only in x_c.
- This gives you:
  - A mapping from **environment to optimal culture conditions**, i.e., “for each cell line / passage / ambient condition, what are the best settings?”
  - Efficient sharing of data across environmental conditions: improvements found in one cell line/environment improve the model globally and help predictions in others.
- They show that ENVBO:
  - Can handle **noisy observations** without major degradation,
  - Requires only a **single optimization campaign** to cover a range of environmental conditions, instead of repeating full optimizations separately for each fixed condition—important under your severe time constraints.
- Their use of **EI and LogEI** with global GP, and their handling of noisy experimental outputs, are directly transferable patterns for your implementation.

---

## Chhajer & Roy – PARSEC: Parameter Sensitivity Clustering for Experiment Design

**Relation to your problem**

- PARSEC is a **model-based design of experiments (MBDoE)** framework focused on **parameter estimation** and **informative experiments**, not black-box optimization per se.
- It uses **global sensitivity analysis** (via parameter sensitivity indices) and **clustering of sensitivity profiles** to:
  - Select **which measurements / time points** to take,
  - And with an Approximate Bayesian Computation (ABC-FAR) estimator, identify experiment designs that give **precise and identifiable parameter estimates**.
- They apply it to **kinetic models** in biology (oscillatory repressilator, viral life cycle), which are structurally similar to dynamic models you might have for cell growth / gene expression.

**What you can appropriate**

- If you have or can construct a **mechanistic ODE/kinetic model** of your cell culture system (e.g., growth factor signaling, viability, gene expression dynamics), PARSEC gives a way to:
  - Decide **which conditions and time points** to sample to get maximal information about model parameters.
  - Incorporate **experimental restrictions** (e.g., limited number of time points, measurable variables, simultaneous measurements) directly into design.
- Specifically:
  - Use **global sensitivity indices** over the design variables (e.g., gas, media composition, growth factor concentrations) to identify **which combinations and time windows** are most informative for distinguishing parameter sets that produce different expression dynamics.
  - Cluster sensitivity profiles to choose **non-redundant experimental settings** (time points, conditions) that span distinct dynamical regimes.
- PARSEC is primarily about **informative sampling**, not directly about “max gene expression across cell lines,” but it could be useful if:
  - You want to first identify or calibrate a mechanistic model of cell response under different conditions and cell lines.
  - Then use that model plus BO to search for robust optima.
- The ABC-FAR method they use shows a viable route for **Bayesian parameter estimation in noisy biological systems** with structured experimental design—useful if you need robust inference about underlying parameters before or alongside optimization.

---

## Greif et al. – Structured Sampling Strategies in Bayesian Optimization (DoE + BO)

**Relation to your problem**

- This paper systematically evaluates how different **initial design/sampling strategies** (LHS, fractional factorial designs, etc.) affect the performance of BO in low- to moderate-dimensional problems with **limited numbers of iterations**—exactly your regime (few batches, each very expensive).
- They show that structured designs can substantially improve BO’s early performance, especially when:
  - The evaluation budget is small,
  - The objective is expensive and noisy.
- They combine DoE methods (LHS, fractional factorial) with BO and benchmark across mathematical functions and real-world cases (including 3D printing and material science data).

**What you can appropriate**

- For your very limited number of runs:
  - The **initial batch of culture conditions** is crucial. Replacing naive random/Sobol set-up with a **carefully chosen structured design** (e.g., LHS, fractional factorial, or mixed approach) can:
    - Give better coverage of the culture condition space,
    - Improve the initial GP fit and thus improve BO’s decisions for subsequent batches.
- Specific ideas:
  - Use **LHS or fractional factorial** for the first few experiments to:
    - Sample broad, yet structured combinations of gas pressures, media fractions, and growth factor concentrations.
    - Avoid clustering of initial points and reduce the risk of missing narrow viable regions.
  - Use BO (e.g., EI/UCB with GP) for subsequent batches based on the initial structured design.
- They emphasize:
  - With only ~20 evaluations, structured initialization can meaningfully change convergence behavior.
  - The “best” design strategy depends on problem characteristics (e.g., boundary optima, dimensionality), but structured designs are consistently better than ad hoc random starts.
- For your case (narrow viable regions, many dead zones):
  - A fractional factorial or LHS design with respect to key factors should reduce the chance that the initial batch sits entirely in “dead run” regions and fails to provide the GP any signal.

---

## Borrotti & Poli – Naïve Bayes Ant Colony Optimization (NACO) for Experimental Design

**Relation to your problem**

- NACO is a **combinatorial optimization** strategy for experimental design in very high-dimensional **discrete** spaces (e.g., enzyme engineering, choosing sequences).
- It combines:
  - A **Naïve Bayes classifier** to learn which discrete elements/positions are associated with high response (based on top-performing experiments),
  - With **Ant Colony Optimization (ACO)** to explore the discrete combinatorial space more efficiently.
- Their setting: constructing enzyme sequences from a library of discrete “words” (amino acid fragments), under tight experimental budgets.

**What you can appropriate**

- Most of your variables (gas pressure, concentrations, mixing ratios) are **continuous**, so NACO in its current form is not directly suited.
- However, conceptually:
  - If you discretize your continuous factors into a set of allowed levels (e.g., a menu of media recipes or pre-defined concentration levels), then your problem starts to resemble their **combinatorial design** setting.
  - NACO’s strategy of:
    - Learning which factor levels (or combinations) appear often in successful experiments via a **Naïve Bayes classifier**,
    - And then **biasing search** toward those combinations via an ACO-like mechanism,
    - Could be adapted as an alternative to GP-based BO in a fully discrete design space.
- NACO is particularly designed for situations where:
  - The search space is huge and discrete,
  - A GP surrogate is hard to construct or not justified.
- For your culture optimization, where continuous BO is feasible and you expect strong nonlinearity and local peaks, the papers on GP-based BO and robust design will likely be more applicable. NACO is more of an interesting alternative if you ended up heavily discretizing your design space (e.g., a fixed grid of preformulated media and gas settings).

---

