## Robust Multi-Objective Bayesian Optimization Under Input Noise

Samuel Daulton * 1 2 Sait Cakmak * 1 3 Maximilian Balandat 1 Michael A. Osborne 2 Enlu Zhou 3 Eytan Bakshy 1

## Abstract

Bayesian optimization (BO) is a sample-efficient approach for tuning design parameters to optimize expensive-to-evaluate, black-box performance metrics. In many manufacturing processes, the design parameters are subject to random input noise, resulting in a product that is often less performant than expected. Although BO methods have been proposed for optimizing a single objective under input noise, no existing method addresses the practical scenario where there are multiple objectives that are sensitive to input perturbations. In this work, we propose the first multi-objective BO method that is robust to input noise. We formalize our goal as optimizing the multivariate value-at-risk (MVAR), a risk measure of the uncertain objectives. Since directly optimizing MVAR is computationally infeasible in many settings, we propose a scalable, theoretically-grounded approach for optimizing MVAR using random scalarizations. Empirically, we find that our approach significantly outperforms alternative methods and efficiently identifies optimal robust designs that will satisfy specifications across multiple metrics with high probability.

## 1. Introduction

Scientists and engineers frequently face optimization problems where the goal is to tune a set of parameters to optimize multiple competing black-box objective functions. Typically, no single design is best with respect to every objective. Hence, the goal in multi-objective (MO) optimization is to identify the Pareto frontier (PF) of optimal trade-offs between the objectives and the corresponding optimal designs. These problems are ubiquitous in a variety of domains including manufacturing (Liao et al., 2008), materials design (Ashby, 2000), robotics (Calandra and Peters, 2014),

and machine learning (Izquierdo et al., 2021; Eriksson et al., 2021). Obtaining measurements of the objectives often requires resource-intensive simulation or experimentation. Therefore, any practical routine for optimizing such functions must be highly sample-efficient; that is, the method must identify optimal design parameters while only querying the objective functions at a small number of designs. Bayesian optimization (BO) (Shahriari et al., 2016) is a popular technique for addressing this class of problems.

In many real-world applications, the final implementation of the selected design parameters are subject to input noise, resulting from uncontrollable perturbations of the input parameters (Beyer and Sendhoff, 2007). For example, many vaccine production processes involve freeze-drying procedures to stabilize active pharmaceutical ingredients to increase storage lifetime (Mortier et al., 2016; Xie and Schenkendorf, 2019). Tuning the operating parameters, such as shelf temperature and chamber pressure, can significantly improve the efficiency of the drying step with little reduction in product quality. However, shelf temperature and chamber pressure are subject to uncontrollable random input noise around the nominal 1 input parameters. Robustness with respect to this input noise is critical. Higher temperatures lead to greater efficiency, but also make the process more sensitive to perturbations in the temperature because if temperature exceeds the critical collapse threshold, there is irreversible product damage. A more conservative temperature that is robust to input noise may be a better choice than a higher temperature that is worse with respect to the nominal objectives . In such high-stakes and high-throughput production pipelines, decision-makers seek to identify robust design parameters that ensure the manufactured products will have high objective quality with high probability.

Optimization without consideration of input noise can lead to solutions that are catastrophic when subjected to input noise at the implementation stage (Doltsinis and Kang, 2004). The toy problem that is illustrated in left plot of Figure 1 demonstrates a scenario where a non-robust design (depicted as a purple square) is better than the robust design

Figure 1: A toy problem where the goal is to tune a single parameter x to maximize two objectives. Left: The nominal values for a non-robust (purple, x 1 ) and a robust design design (green, x 2 ) are indicated using squares. The plus markers illustrate objective values of each design under zero-mean Gaussian input noise with a standard deviation of 0.1. The non-robust design can lead to low objective values under input perturbations. Center: An illustration of the MVAR sets of the non-robust and robust designs. The triangles represent a discrete approximation of the MVAR set of each design under the input noise distribution. For each point z in the MVAR set of a given design, the objective values for that design subject to input perturbations are ≥ z with probability α ≥ 0 . 9 . In other words, the objectives under input noise for each design will fall in the respective shaded region with probability ≥ α . Under input noise, the non-robust design x 1 may result in poor objective values despite yielding better values (relative to the robust design x 2 ) without perturbations. After accounting for input noise, x 2 is a more robust solution than x 1 . Right: The MVAR set (black stars) across three optimal designs x ∗ 1 , x ∗ 2 , x ∗ 3 is the set of optimal points across the union of the MVAR sets (colored triangles) of each design.

picture-1.png

(marked by a green square) with respect to the nominal values. However, the performance metrics for the non-robust design can be significantly worse when the design is subject to input noise. In contrast, the objectives are far less sensitive to input perturbations around the robust design.

To identify designs that are robust to a noisy performance metric due to input noise or observation noise, the valueat-risk (VAR) is often used as the robust objective because it provides high-probability performance guarantees (Basel Committee on Banking Supervision, 2012). The α VAR is the (1 -α ) -quantile (where 0 ≤ α ≤ 1 ) of the cumulative distribution function (CDF) of the performance metric. Under input noise, variation in the objective is induced via the uncertain inputs. Intuitively, the VAR is the largest value such that the objective value of a given design subject to input perturbations will be greater than that value with probability at least α . Thus, in the context of manufacturing, a design with α VAR exceeding the target specification produces a yield of at least α .

Many recent works have considered BO methods that are robust to input noise in the single objective setting, e.g., by optimizing VAR (Nguyen et al., 2021b) or other risk measures (Frohlich et al., 2020; Cakmak et al., 2020; Nguyen et al., 2021a). However, no previous work has considered sample-efficient, generally-applicable multi-objective Bayesian optimization (MOBO) methods that are robust to input perturbations. To our knowledge, the only existing MO methods that are robust to input noise are evolution-a

y algorithms (EA) that often require tens of thousands of evaluations (e.g., Deb and Gupta (2005)).

In the MO setting, high-probability performance guarantees of a single design can be assessed using the multivariate value-at-risk (MVAR) (Pr'ekopa, 2012). MVAR maps a probability value α to a set of vectors where each element provides a lower bound on the the objectives' possible values under input noise with probability α . As illustrated in the center plot of Figure 1, the MVAR set of a robust design ( x 2 ) often provides significantly better probabilistic lower bounds than the MVAR set of a non-robust design ( x 1 ).

Similar to the PF in the standard MO setting-where the PF is the set of optimal trade-offs between objectives with no input noise-the global MVAR set is the set of optimal trade-offs that can be achieved with probability ≥ α under input noise across all possible designs. The MVAR across a set of 3 designs is illustrated in the right plot of Figure 1.

While MVAR provides a natural measure of robust performance guarantees, it is relatively expensive to compute, making it computationally challenging to directly optimize MVAR in the context of MOBO (see Appendix D.1 for a discussion). In this work, we propose a family of novel, theoretically-grounded methods for optimizing MVAR via random scalarizations that mitigates many challenges associated with optimizing MVAR directly.

## Contributions

1. We introduce robust MO optimization under input

noise, formalize the problem in terms of optimizing global MVAR-a novel, probabilistic form of a robust PF-and discuss computational challenges unique to this setting.

- 2. We derive a novel theoretical connection between the MVAR and the VAR of a particular scalarization of the objectives, which motivates a family of computationally efficient BO methods for identifying MVARoptimal designs using an MVAR Approximation based on Random Scalarizations (MARS).
- 3. We demonstrate that MARS vastly outperforms nonrobust alternatives on a variety of synthetic and realworld robust MO optimization problems, including a pharmaceutical manufacturing application.
- 4. We derive and evaluate extensions of our methods to handle expensive-to-evaluate black-box constraints and parallel candidate generation.

## 2. Background

Multi-Objective Optimization In MO optimization, the goal is to, without loss of generality, maximize a vectorvalued black-box function max x ∈X f ( x ) where f ( x ) := [ f 1 ( x ) , . . . , f M ( x )] , M ≥ 2 , and X ⊂ R d is a compact search space. Often, there may be additional black-box constraints c ( x ) ≥ 0 , where c ( x ) ∈ R V , V > 0 , that must be satisfied. We consider the setting where f and c have no known analytic expressions and no known or observed gradients. The goal is to identify the Pareto frontier (PF) of optimal trade-offs and corresponding Pareto set of optimal designs X ∗ .

Notation For vectors y , y ' ∈ R M . Let ≥ and > denote the component-wise extensions of the order notations ≥ and > , i.e., y ≥ y ' ⇐⇒ y i ≥ y ' i ∀ i and y > y ' ⇐⇒ y i > y ' i ∀ i , where · i denotes the i th element.

Definition 2.1. A vector f ( x ) Pareto dominates f ( x ' ) , denoted by f ( x ) glyph[follows] f ( x ' ) , if f ( x ) ≥ f ( x ' ) and ∃ j ∈ { 1 , . . . , M } such that f ( j ) ( x ) > f ( j ) ( x ' ) .

Definition 2.2. The Pareto frontier over a set of objective vectors F = { f ( x ) | x ∈ X ⊆ X} is PARETO ( F ) = { f ( x ) ∈ F : glyph[notexistential] x ' ∈ X s.t. f ( x ' ) glyph[follows] f ( x ) } . If there are constraints c ( x ) , elements of PARETO are subject to the additional membership assumption that c ( x ) ≥ 0 . We call the corresponding set of optimal designs the Pareto set .

Although the true PF P ∗ = PARETO ( { f ( x ) } x ∈X ) is typically unknown, an MO optimization algorithm can be employed to identify a finite approximation. Hypervolume is a commonly used metric to measure the quality of a PF.

Definition 2.3. The hypervolume (HV) indicator of a set of points Y ⊂ R M is the M -dimensional Lebesgue measure λ M of the region dominated by P := PARETO ( Y ) and

bounded from below by a reference point r ∈ R M , which we write as HV ( Y , r ) .

Definition 2.4. The hypervolume improvement (HVI) of a set of points Y ' with respect to an existing Pareto frontier P and reference point r is defined as HVI ( Y ' |P , r ) = HV ( P ∪ Y ' | r ) -HV ( P| r ) . 2

Bayesian Optimization (BO) is a sample-efficient technique for optimizing black-box functions (Frazier, 2018). BO relies on a probabilistic surrogate model-typically a Gaussian process (GP), which provides well-calibrated uncertainty estimates-and an acquisition function that leverages the surrogate model to quantify the value of evaluating the objective functions for a design x . The acquisition function balances exploring areas with high uncertainty and exploiting regions believed to be optimal. BO selects the next point x to evaluate by maximizing the acquisition function (which is cheap to evaluate relative to the objective functions), observes a (potentially noisy) measurement of the metrics y , adds the new observation to the dataset D = { ( x , y ) }∪{ ( x i , y i ) } n i =1 , updates the surrogate model to incorporate the new observation, and repeats this process for a predetermined budget of evaluations. In the MO setting, a common approach is to optimize a random scalarization of the objectives using a single objective acquisition function, such as expected improvement (Knowles, 2006) or Thompson sampling (Paria et al., 2018). Alternatively, a multi-objective acquisition function can be used to directly optimize the PF. For example, expected hypervolume improvement (EHVI) (Emmerich et al., 2006) aims to maximize the HV of the PF under the surrogate model's posterior distribution.

## 3. Related Work

Many recent works on BO have considered settings where at the time of implementation either (i) the parameters are subject to noise (Bogunovic et al., 2018; Oliveira et al., 2019; Frohlich et al., 2020)-as we consider in this work-or (ii) the system performance depends on auxiliary unknown environmental variables (Kirschner et al., 2020; Iwazaki et al., 2021). While previous work has focused on optimizing the expected (Toscano-Palmerin and Frazier, 2018) or the worst-case performance (ur Rehman et al., 2014; Sessa et al., 2020), a recent body of work has focused on optimizing more sophisticated risk measures (Torossian et al., 2020; Cakmak et al., 2020; Nguyen et al., 2021a;b). However, despite recent significant interest in non-robust MOBO (Lukovic et al., 2020; Suzuki et al., 2020), to our knowledge, no prior work has studied robust MOBO.

Motivated by practical limitations due to manufacturing tolerances, Malkomes et al. (2021) proposed constraint ac-

tive search (CAS), which aims to identify diverse solutions in the region of the search space that exceeds a minimum threshold on the objectives. However, CAS does not model or account for input noise, and CAS alone cannot produce any guarantees on robustness to input noise. Methods such as CAS would require a post-hoc analysis using the data collected during optimization to analyze the sensitivity of the solutions to input noise (Calandra and Peters, 2014).

Approaching robust design by decoupling data collection and sensitivity analysis is central to the Taguchi method (Taguchi, 1989). Data acquisition often revolves around finding designs that balance the mean and variance of the sensitive objective under input noise (Beyer and Sendhoff, 2007). Do et al. (2021) propose an approach in this vein for the two-objective setting where only one objective is subject to input noise. However, the algorithm does not seek to identify trade-offs with high probability robustness guarantees, and the method does not handle multiple sensitive objectives. In contrast with the Taguchi method, we aim to unify data collection and sensitivity analysis by selecting designs that are believed to yield high-probability performance guarantees.

Outside the BO literature, robust MO optimization has been studied using either EAs (Gupta and Deb, 2005; He et al., 2019) or assuming access to the explicit mathematical programming formulation of the problem (Majewski et al., 2017; Roberts et al., 2018). Those works have focused on finding the Pareto frontier of the expectation or the worstcase objectives or on finding the Pareto frontier of the nominal objectives with additional constraints on the deviation from the nominal values (Deb and Gupta, 2005; Avigad and Branke, 2008). Some works have considered conceptual properties of different scalarization methods (Ide and Kobis, 2014), but not in relation to MVAR. EAs that are robust to input noise are not applicable to the small evaluation budget regime that we consider because they typically require a large number of function evaluations (Deb and Gupta, 2005). Even methods that combine EAs with GPs require thousands of evaluations per design (Zhou et al., 2018).

As a final differentiator from prior work, we consider the practical setting where there are additional black-box constraints that are sensitive to input noise (Marzat et al., 2013; Li and Li, 2015), which is a subject addressed by only a few BO methods (Beland and Nair, 2017) even in the single objective case.

## 4. Multi-Objective Optimization with Noisy Inputs

In many practical scenarios, the nominal performance of a design can be evaluated by means of a simulation (e.g., by simulating the pharmaceutical process under nominal operating conditions). We consider the setting where we

can simulate f ( x ) for any given design x ∈ X , but that the design is subject to noise ξ ( x ) from a known noise process ξ ( x ) ∼ P ( ξ ; x ) at implementation time. 3 The realized system performance is given by the random variable f ( x glyph[diamondmath] ξ ) , where x glyph[diamondmath] ξ denotes any known function g ( x , ξ ) (e.g. for additive noise glyph[diamondmath] is simply + ). For an extended problem formulation including black-box constraints, see Appendix A.4.

In robust optimization, the goal is often to optimize a risk measure ρ [ f ( x glyph[diamondmath] ξ )] that maps a random variable to a statistic of its distribution. A common risk measure is the expectation over the input noise distribution (Deb and Gupta, 2005; Toscano-Palmerin and Frazier, 2018; Frohlich et al., 2020), E ξ ∼ P ( ξ ) [ f ( x glyph[diamondmath] ξ )] , which can be used instead of the random variable f ( x glyph[diamondmath] ξ ) and optimized via standard multi-objective optimization methods. We propose the first MOBO methods for optimizing expectation objectives in Appendix F. Despite its widespread use, the expectation risk measure may not always align with the practitioner's true robustness goals. Often, one would prefer solutions with objectives that are better than some performance specification z ∈ R M with high probability (e.g. to maximize production yield) (Sarykalin et al., 2008). Hence, in the single-objective setting, probabilistic risk measures such as value-at-risk (VAR) are frequently used.

Definition 4.1. Given input noise ξ ∼ P ( ξ ) where ξ ∈ R d and a confidence level α ∈ [0 , 1] , the value-at-risk for a given point x is:

VAR α [ f ( x glyph[diamondmath] ξ ) ] = sup { z ∈ R : P [ f ( x glyph[diamondmath] ξ ) ≥ z ] ≥ α } .

Although several BO methods exist for optimizing VAR (Cakmak et al., 2020; Nguyen et al., 2021b), they cannot directly be used in the MO setting because VAR is not defined for multivariate random variables. A naive way to extend VAR to the MO setting would be to consider the VAR of each objective independently. However, this ignores the fact that all M objectives are evaluated at the same realization of x glyph[diamondmath] ξ . Considering the VAR of each objective independently typically leads to overly optimistic risk estimates because objectives under input noise are not typically simultaneously greater than or equal to their respective independent VARs (i.e. the (1 -α ) - quantiles) with probability ≥ α . Thus it is important to use risk measures such as multivariate value-at-risk (MVAR) that account for the joint distribution of the objectives (Pr'ekopa, 2012).

This is illustrated in the center plot in Figure 1. Under input noise, the objective values are correlated, which underscores the importance of accounting for the joint distribution of the objectives in measures of robustness. In addition, the center plot in Figure 1 shows that f ( x glyph[diamondmath] ξ ) has an asymmetric distribution, even for this very simple and well-behaved

toy problem, which highlights how applying VAR to each objective independently or using the expectation risk measure may conceal underlying variation and risk. Indeed, the results in Appendix I show that optimizing an expectation risk measure on this problem results in poor performance.

In contrast with VAR and the expectation risk measure which map a random variable to single scalar or vector, MVAR maps a random variable to a non-dominated set of vectors in the outcome space that are dominated by α -fraction of all possible realizations, where α ∈ [0 , 1] is a hyperparameter set by the practitioner. That is, each vector in the MVAR set corresponds to an objective specification that a design will meet with probability ≥ α . Therefore, α is an interpretable risk level that can be valuable in manufacturing applications where one wishes to find the PF of all objective specifications with a guaranteed yield fraction ( α ).

Definition 4.2. The MVAR of f for a given point x and confidence level α ∈ [0 , 1] is:

MVAR α [ f ( x glyph[diamondmath] ξ ) ] = PARETO ({ z ∈ R M : P [ f ( x glyph[diamondmath] ξ ) ≥ z ] ≥ α }) .

The MVAR set over X specifies objective vectors z such that there exists a known design x ∈ X with corresponding random objectives f ( x glyph[diamondmath] ξ ) under P ( ξ ) that dominate z with probability ≥ α .

Definition 4.3. The MVAR for a set of points X is:

MVAR α [ { f ( x glyph[diamondmath] ξ ) } x ∈ X ] = PARETO ( ⋃ x ∈ X MVAR α [ f ( x glyph[diamondmath] ξ ) ] ) .

The global MVAR across the design space, MVAR α [ { f ( x glyph[diamondmath] ξ ) } x ∈X ] , is a robust analogue of the PF in the standard MO setting. The concept of the MVAR of a set of design points X is a novel contribution of this work.

Optimization Goal In this work, our goal is to identify the MVAR set across the design space: MVAR α [ { f ( x glyph[diamondmath] ξ ) } x ∈X ] . Given an approximate MVAR set across the design space, a decision-maker can pick a design according to their preferences. Similar to the standard MO setting, the HV of the MVAR set across the design space can be used to evaluate optimization performance.

## 5. Optimizing MVAR

A natural approach for optimizing MVAR is to directly maximize the HV dominated by the MVAR set. Although MVAR of a given point typically cannot be evaluated directly, it can be approximated using n ξ MC samples of ξ , provided that independent samples can be draw from the noise process. Thus, evaluating the MVAR set across the previously evaluated designs using the surrogate requires sampling from the posterior of P ( f |D ) evaluated

jointly at x 1 glyph[diamondmath] ξ i , . . . , x n glyph[diamondmath] ξ i for i = 1 , ..., n ξ , where X 1: n := { x 1 , ..., x n } are the previously evaluated designs. Since { f ( x ' glyph[diamondmath] ξ ) } x ' ∈ X 1: n is typically not observed, the corresponding posterior predictions may have large uncertainties. In order to get a reliable estimate of MVAR, we would need to integrate over the posterior distribution of { f ( x ' glyph[diamondmath] ξ ) } x ' ∈ X 1: n . q NEHVI (Daulton et al., 2021a) is a variant of EHVI that integrates over the uncertainty in function values at previously evaluated designs. This makes q NEHVI suitable for optimizing MVAR.

However, several computational issues-including time complexity that is exponential in the number of objectives and exponential in the size of MVAR α [ f ( x glyph[diamondmath] ξ ) ] -make it infeasible to directly optimize MVAR with q NEHVI in many settings. We defer a detailed discussion to Appendix D and present an empirical evaluation in Appendix I.

## 5.1. Relationship between MVAR and Scalarizations

An alternative to direct optimization of the MVAR set is to apply a scalarization to the objectives and use a standard risk measure on the scalarized objective. Unlike the use of independent risk measures on each objective, this approach accounts for the correlation between outcomes induced by the input perturbation. In this section, we present our main theoretical result: under limited assumptions, there exists a bijection, based on VAR, that maps a particular family of scalarizations-Chebyshev scalarizations (Kaisa, 1999)-to points in the MVAR set. In other words, each point in the MVAR set corresponds to a particular set of scalarization weights. This means that we can recover the entire MVAR set using these scalarizations, without any loss. Proofs and additional theoretical results including extensions to the constrained setting are provided in Appendix A.

Definition 5.1. Let w ∈ ∆ M -1 + , where ∆ M -1 + denotes the positive ( M -1) -simplex, and let r ∈ R M . The Chebyshev scalarization s [ y , w , r ] is given by s [ y , w , r ] = min i w i ( y i -r i ) , where · i denotes the i th dimension. 4

The contour in the left plot in Figure 2 shows the Chebyshev scalarization for a fixed w for the two-objective toy problem from Figure 1 and illustrates a connection between Pareto dominance and the Chebyshev scalarization, which we formalize below. The black points are function values under sampled perturbations for a single design x . The center plot in Figure 2 shows the distribution of Chebyshev scalarization values for a given w and the black line indicates the α -level VAR. The right plot in Figure 2 illustrates that using the VAR of a Chebyshev scalarization, we can deduce a point z such that the function values under the

Figure 2: Construction of MVAR sets via random scalarizations for the 1-d example in Figure 1. Left: The function values for a single design with zero-mean Gaussian input perturbations with a standard deviation of 0.1 are marked by black points. The background is a contour showing the value of a Chebyshev scalarization across the objective space. Center: The probability density of a Chebyshev scalarization over the function values under input noise and the value-at-risk v of a Chebyshev scalarization for α = 0 . 9 . The probability mass to the right of the black line is equal to α . Right: Leveraging Theoream 5.1, VAR , w can be mapped to point in the MVAR set that is dominated by the objectives under input perturbations 90% of the time. The green triangles represent a discrete approximation of the MVAR set with 64 samples from the input noise distribution. The green area indicates the region that dominates the identified MVAR point, or, equivalently, the area for which the Chebyshev scalarization defined by w is greater than v .

picture-2.png

input perturbations will dominate z with probability ≥ α .

Lemma 5.1 (VAR of Chebyshev scalarization ⇒ Pareto Dominance) . Let v = VAR α ( s [ f ( x glyph[diamondmath] ξ ) , w ] ) . Then, P [ f ( x glyph[diamondmath] ξ ) ≥ v w ] ≥ α , where v w denotes element-wise division.

The condition in Lemma 5.1 is one criterion for membership in the MVAR set (the other being Pareto efficiency). The shaded region in right plot in Figure 2 illustrates the region that dominates z . With probability ≥ α , f ( x glyph[diamondmath] ξ ) will fall within the shaded region. This result enables translating the VAR of a Chebyshev scalarization into an interpretable, high-probability guarantee on robust performance in terms of Pareto dominance.

Assumption 5.1. f ( x glyph[diamondmath] ξ ) has a continuous, strictly increasing CDF F . I.e., if f ( x ) glyph[follows] f ( x ' ) , then F [ f ( x ) ] > F [ f ( x ' ) ] . 5

If Assumption 5.1 is met, then for any w , v w is not dominated by any other point in the MVAR set, and hence, v w is an element of the MVAR set. Furthermore, we have the following:

Theorem 5.1 (MVAR ⇐⇒ VAR of Chebyshev scalarization) . Given x , f , and P ( ξ ) , let h : MVAR α [ f ( x glyph[diamondmath] ξ ) ] → ∆ M -1 + be the function h ( z ) = w = 1 z || 1 z || . Under

Assumption 5.1, h ( · ) is bijective and h -1 ( w ) = z = 1 w VAR α ( s [ f ( x glyph[diamondmath] ξ ) , w ] ) .

Theorem 5.1 provides a technique for generating points in the MVAR set using the VAR of Chebyshev scalarizations with different weights. Importantly, any design that is globally optimal with respect to MVAR is a maximizer of the VAR of a Chebyshev scalarization. This naturally motivates a methodology for identifying the global MVAR set by optimizing the VAR of random Chebyshev scalarizations.

Corollary 5.1 (Consistent Optimizers) . Suppose z is a point in the global MVAR set, i.e., z ∈ MVAR α [ { f ( x glyph[diamondmath] ξ ) } x ∈X ] . Let X ∗ z be the set of designs such that for all x ∈ X ∗ z , z ∈ MVAR α [ f ( x glyph[diamondmath] ξ ) ] . Then every x ∈ X ∗ z is a maximizer of VAR α ( s [ f ( x glyph[diamondmath] ξ ) , w ] ) for w = ( z || 1 z || ) -1 .

## 5.2. MARS: MVAR Approximation via Random Scalarizations

The connection between the VAR of a Chebyshev scalarization and MVAR can be exploited to optimize the global MVAR by randomly sampling a Chebyshev scalarization at each BO iteration 6 and optimizing the VAR of the Chebyshev scalarization using a single objective BO algorithm, such as Noisy Expected Improvement (NEI) (Letham et al., 2019)-which is required (rather than expected improvement) since we must integrate over the unknown best

unknown incumbent value-or Thompson sampling (TS) (Thompson, 1933; Paria et al., 2018). We refer to this technique as MVAR Approximation via Random Scalarizations (MARS). MARS is a simple, theoretically-grounded technique, and we find, in Section 6, that MARS performs well empirically. In the main text, we focus on MARS with NEI (denoted as MARS-NEI) , but we derive and empirically evaluate upper confidence bound (UCB) (Srinivas et al., 2010) and TS variants in Appendices B and G.

For MARS-NEI, we use the MC formulation of NEI (Balandat et al., 2020) so that the Chebyshev scalarizations can be computed in the NEI as composite objectives (Astudillo and Frazier, 2019). We optimize the the acquisition function using sample-path gradients that leverage well-studied gradient estimators of VAR (see Appendix C.1 for details). See Appendix G.1 for details on optimization.

## 6. Experiments

In this section, we provide an empirical demonstration of robust MOBO on synthetic and real-world problems. We compare three broad classes of methods: (i) Non-robust methods including NSGA-II (Deb et al., 2002), q NPAREGO (Daulton et al., 2020), q NEHVI (Daulton et al., 2021b), (ii) methods for optimizing expectation risk measures, e.g. using q NPAREGO (denoted as EXPq NPAREGO), and (iii) methods for optimizing MVAR via MARS. For readability, we only include one expectation and one MVAR optimization method in the main text, both based on NEI. In Appendix G, we evaluate additional methods including MARS with TS and UCB and methods for direct MVAR optimization based on NEHVI. We consider the expectation risk measure because it is simple and performs well in many scenarios. All robust (expectation and MVAR) methods are our novel contributions. Additionally, we compare against a quasi-random policy, which selects the designs to evaluate according to a scrambled Sobol sequence (Owen, 2003).

For all BO-methods, we begin by evaluating 2( d +1) design points from a scrambled Sobol sequence. We use n ξ = 32 samples because we find that setting n ξ > 32 yields little-to-no improvement in optimization performance (see Figure 12). See Appendix G for details on all methods. Our code is open-sourced at github.com/ facebookresearch/robust\_mobo .

## 6.1. Synthetic Problems

Gaussian Mixture Model (GMM) ( d = 2 , M = 2 , α = 0 . 9 ): This is a variant of the GMM problem from Frohlich et al. (2020) where each objective is an independent GMM. We use a multiplicative noise model, i.e., x glyph[diamondmath] ξ := xξ , where ξ ∼ N ( µ = 1 , Σ = 0 . 07 I 2 ) with I n denoting the n -dimensional identity matrix. In Appendix I.5, we present multiple variations of this problem to demonstrate the consistency of our methods under different noise models.

Constrained Branin Currin ( d = 2 , M = 2 , V = 1 , α = 0 . 7 ): We subject this problem, which originates from Daulton et al. (2020), to a heteroskedastic input noise process given by P ( ξ ; x ) = N ( µ = 0 , Σ = 0 . 05 (1 + σ (1 -2 x 0 )) I 2 ) , where σ ( x ) = 1 1+ e -x . The optimal designs with respect to the nominal objectives are on the boundary of the feasible region with respect to the outcome constraint. Hence, the optimal designs with respect to the nominal metrics often violate the constraint under input perturbations.

## 6.2. Real-World Problems

Disc Brake ( d = 4 , M = 2 , V = 4 , α = 0 . 95 ): In this disc brake manufacturing problem, the goal is to minimize the brake's mass and the stopping time of a vehicle by tuning the inner and outer radii of the disc, the engaging force, and the number of friction surfaces (Ray and Liew, 2002). Following Emch and Parkinson (1994), we use zero-mean uniform input noise with a maximum absolute perturbation value of 5% of the range of each parameter (except for the number of friction surfaces, which is noise-free).

Penicillin Production ( d = 7 , M = 3 , α = 0 . 8 ): This problem considers optimizing the manufacturing process of penicillin (Liang and Lai, 2021). The goal is to maximize the yield while minimizing time-to-ferment and CO 2 output by tuning 7 initial conditions of the chemical reaction. Each parameter is subject to independent zero-mean Gaussian noise, where the standard deviation ranges from 0.5% to 3% of the parameter's domain (see Appendix G for details).

## 6.3. Results

In Figure 9, we evaluate all methods in terms of log HV regret, which is the difference in HV between the true global MVAR set and the MVAR of the set of designs evaluated by each method (see Appendix G.3 for details on estimating true global MVAR). Figure 3 shows that MARS is consistently the best performing method. The non-robust methods consistently perform poorly on all problems. On the GMM problem, EXPq NPAREGO performs worse than MARSNEI because the optimal designs under expectation risk measure are in a disjoint part of the search space from the MVAR-optimal designs. On the Constrained Branin Currin problem, the nominal methods perform no better than quasirandom Sobol search. On the penicillin problem, MARS vastly outperforms all other methods.

Although the log HV regret highlights the performance of MARS, it does not fully capture the necessity of using a robust method in practice. In Figure 4, we analyze the yield (i.e. the probability of the objectives exceeding a performance specification under the input noise distribution) of a robust and a non-robust design on the Disc Brake problem. Using a target performance specification chosen from the

Figure 3: The log MVAR HV regret after the initial space-filling design. For each method, we plot the mean and 2 standard errors of the mean over 20 trials.

picture-3.png

Figure 4: The yield from selecting a robust versus a nonrobust design on the Disc Brake problem. Although the non-robust design is feasible under the nominal objectives, it is located near the boundary of the feasible region in design space and violates some of the black-box constraints (not shown) under a large fraction of input perturbations.

picture-4.png

MVaR

Nominal Values of MVaR Optimal Designs

Target Specification

Nominal Values of the Robust Design

Nominal Values of the Non-robust Design

Dist. of Robust Design, Feasible

MVAR set, we see that if a decision-maker were to select a non-robust design (green pentagon) that is optimal with respect to the nominal objectives and nominal values that meet the target specification, the yield would only be 58 . 2% . In contrast, an MVAR-optimal solution can be chosen such that it meets target specification with high probability. For example, the robust design marked by the orange triangle enjoys a yield of 95 . 3% . As shown in Figure 4, this is because the non-robust design often does not satisfy all of the black-box constraints under input perturbations. In this problem, the objectives are relatively robust to noise (much more so than the toy example in Figure 1), but the feasibility of a design (and therefore the yield) is highly sensitive to input noise when the design is near the boundary of the feasible region in design space.

Table 3 reports the wall times for running a single iteration of BO with each algorithm. Not only is MARS-NEI computationally tractable on all problems, but it achieves wall times that are competitive to alternative algorithms.

PF over Nominal Values

Dist. of Robust Design, Infeasible

Dist. of Non-Robust Design, Feasible

Dist. of Non-Robust Design, Infeasible

Area Meeting Target Specification

Evaluation in additional problem settings and comparisons against additional methods in Appendix I further validate that MARS-NEI is consistently a top performer and yields competitive wall times. We find that MARS-TS performs slightly worse, on average, in terms of log HV regret than MARS-NEI, but that it has shorter wall times. Methods for direct MVAR optimization with q NEHVI perform comparably to MARS-NEI, but the direct MVAR optimization is prohibitively expensive in terms of wall time and memory requirements and was infeasible to run on many problems (including nearly all problems with > 2 objectives). In contrast, Figure 7 shows that MARS-NEI can scale to problems with > 2 objectives and consistently performs best in those settings. Additionally, in Appendix G, we show that MARS-NEI works well under a wide variety of input noise processes, scales well with increasing batch sizes (in the parallel evaluation setting), and is not sensitive to the number of MC samples n ξ used to estimate P ( ξ ) , for n ξ ≥ 32 .

## 7. Discussion

In this work, we formulate the goal of MO robust optimization under input noise as optimizing the global MVAR set-a novel concept that is a robust analogue of the Pareto frontier in the standard MO setting. We derive a correspondence between MVAR and Chebyshev scalarizations based on VAR. This theoretical result naturally motivates a computationally efficient approach (MARS) for using BO to optimize the global MVAR set with high sample-efficiency. Empirically, we find that MARS consistently outperforms alternative approaches and achieves competitive wall times.

Although our focus has been on the small evaluation budget regime and BO, our theoretical results are far more general. The connection between MVAR and Chebyshev scalarizations could be leveraged by gradient-based and evolutionary methods to scale global MVAR optimization to settings with large evaluation budgets. We hope that our contributions serve as a foundation for future advances in methods for robust MO optimization under input noise.

## Acknowledgements

We thank Ben Letham, David Eriksson, James Wilson, Martin Jørgensen, and Raul Astudillo, as well as the members of the Oxford Machine Learning Research Group, for providing insightful feedback. In addition, Sait Cakmak and Enlu Zhou are grateful for support by the Air Force Office of Scientific Research under Grant FA9550-19-1-0283.

## References

M.F. Ashby. Multi-objective optimization in material design and selection. Acta Materialia , 48(1):359-369, 2000. ISSN 1359-6454.

Raul Astudillo and Peter Frazier. Bayesian optimization of composite functions. In Kamalika Chaudhuri and Ruslan Salakhutdinov, editors, Proceedings of the 36th International Conference on Machine Learning , volume 97 of Proceedings of Machine Learning Research , pages 354363. PMLR, 09-15 Jun 2019.

- Gideon Avigad and Jurgen Branke. Embedded evolutionary multi-objective optimization for worst case robustness. In Proceedings of the 10th Annual Conference on Genetic and Evolutionary Computation , GECCO '08, page 617-624, New York, NY, USA, 2008. Association for Computing Machinery. ISBN 9781605581309. doi: 10.1145/1389095.1389221.

Maximilian Balandat, Brian Karrer, Daniel R. Jiang, Samuel Daulton, Benjamin Letham, Andrew Gordon Wilson, and Eytan Bakshy. BoTorch: A Framework for Efficient Monte-Carlo Bayesian Optimization. In Advances in Neural Information Processing Systems 33 , 2020.

Basel Committee on Banking Supervision. Fundamental review of the trading book. Consultative Document , 2012.

- Justin J. Beland and Prasanth B. Nair. Bayesian optimization under uncertainty. 2017.

Hans-Georg Beyer and Bernhard Sendhoff. Robust optimization - a comprehensive survey. Computer Methods in Applied Mechanics and Engineering , 196(33):31903218, 2007. ISSN 0045-7825.

- J. Blank and K. Deb. pymoo: Multi-objective optimization in python. IEEE Access , 8:89497-89509, 2020.

Ilija Bogunovic, Jonathan Scarlett, Stefanie Jegelka, and Volkan Cevher. Adversarially robust optimization with gaussian processes. In S. Bengio, H. Wallach, H. Larochelle, K. Grauman, N. Cesa-Bianchi, and R. Garnett, editors, Advances in Neural Information Processing Systems , volume 31. Curran Associates, Inc., 2018.

Edwin V Bonilla, Kian Chai, and Christopher Williams. Multi-task gaussian process prediction. In J. Platt, D. Koller, Y. Singer, and S. Roweis, editors, Advances in Neural Information Processing Systems , volume 20. Curran Associates, Inc., 2008. URL https://proceedings. neurips.cc/paper/2007/file/ 66368270ffd51418ec58bd793f2d9b1b-Paper. pdf .

- Richard H. Byrd, Peihuang Lu, Jorge Nocedal, and Ciyou Zhu. A limited memory algorithm for bound constrained optimization. SIAM J. Sci. Comput. , 16:1190-1208, 1995.
- Sait Cakmak, Raul Astudillo, Peter Frazier, and Enlu Zhou. Bayesian optimization of risk measures. In Advances in Neural Information Processing Systems 33 , 2020.
- Roberto Calandra and Jan Peters. Pareto front modeling for sensitivity analysis in multi-objective bayesian optimization. 2014.

Daniele Calandriello, Luigi Carratino, Alessandro Lazaric, Michal Valko, and Lorenzo Rosasco. Gaussian process optimization with adaptive sketching: Scalable and no regret, 2019.

- Sayak Ray Chowdhury and Aditya Gopalan. On kernelized multi-armed bandits. In Proceedings of the 34th International Conference on Machine Learning - Volume 70 , ICML'17, page 844-853. JMLR.org, 2017.

Areski Cousin and Elena Di Bernardino. On multivariate extensions of conditional-tail-expectation. Insurance: Mathematics and Economics , 55:272-282, 2014.

Samuel Daulton, Maximilian Balandat, and Eytan Bakshy. Differentiable Expected Hypervolume Improvement for Parallel Multi-Objective Bayesian Optimization. In Advances in Neural Information Processing Systems 33 , 2020.

Samuel Daulton, Maximilian Balandat, and Eytan Bakshy. Parallel Bayesian Optimization of Multiple Noisy Objectives with Expected Hypervolume Improvement. In Advances in Neural Information Processing Systems 34 , 2021a.

- Samuel Daulton, David Eriksson, Maximilian Balandat, and Eytan Bakshy. Multi-objective bayesian optimization over high-dimensional search spaces, 2021b.
- Nando de Freitas, Alex Smola, and Masrour Zoghi. Exponential regret bounds for gaussian process bandits with deterministic observations. In ICML , 2012.
- K. Deb, A. Pratap, S. Agarwal, and T. Meyarivan. A fast and elitist multiobjective genetic algorithm: Nsga-ii. IEEE

Transactions on Evolutionary Computation , 6(2):182197, 2002.

Kalyanmoy Deb and Himanshu Gupta. Searching for robust pareto-optimal solutions in multi-objective optimization. In Carlos A. Coello Coello, Arturo Hern'andez Aguirre, and Eckart Zitzler, editors, Evolutionary Multi-Criterion Optimization , pages 150-164, Berlin, Heidelberg, 2005. Springer Berlin Heidelberg.

- Bach Do, Makoto Ohsaki, and Makoto Yamakawa. Bayesian optimization for robust design of steel frames with joint and individual probabilistic constraints. Engineering Structures , 245:112859, 2021. ISSN 0141-0296.

Ioannis Doltsinis and Zhan Kang. Robust design of structures using optimization methods. Computer Methods in Applied Mechanics and Engineering , 193(23):2221-2237, 2004. ISSN 00457825. doi: https://doi.org/10.1016/j.cma.2003.12. 055. URL https://www.sciencedirect.com/ science/article/pii/S0045782504000787 .

- G. Emch and A. Parkinson. Robust Optimal Design for Worst-Case Tolerances. Journal of Mechanical Design , 116(4):1019-1025, 12 1994. ISSN 1050-0472. doi: 10. 1115/1.2919482.
- M. T. M. Emmerich, K. C. Giannakoglou, and B. Naujoks. Single- and multiobjective evolutionary optimization assisted by gaussian random field metamodels. IEEE Transactions on Evolutionary Computation , 10(4):421-439, 2006.

David Eriksson, Pierce I-Jen Chuang, Samuel Daulton, Peng Xia, Akshat Shrivastava, Arun Babu, Shicong Zhao, Ahmed Aly, Ganesh Venkatesh, and Maximilian Balandat. Latency-aware neural architecture search with multiobjective bayesian optimization, 2021.

Peter I Frazier. A tutorial on bayesian optimization. arXiv preprint arXiv:1807.02811 , 2018.

Lukas Frohlich, Edgar Klenske, Julia Vinogradska, Christian Daniel, and Melanie Zeilinger. Noisy-input entropy search for efficient robust bayesian optimization. In Silvia Chiappa and Roberto Calandra, editors, Proceedings of the Twenty Third International Conference on Artificial Intelligence and Statistics , volume 108 of Proceedings of Machine Learning Research , pages 2262-2272. PMLR, 26-28 Aug 2020.

Himanshu Gupta and Kalyanmoy Deb. Handling constraints in robust multi-objective optimization. In 2005 IEEE Congress on Evolutionary Computation , volume 1, pages 25-32 Vol.1, 2005. doi: 10.1109/CEC.2005.1554663.

Nikolaus Hansen. The CMA Evolution Strategy: A Comparing Review , volume 192, pages 75-102. 06 2007. doi: 10.1007/3-540-32494-1 4.

- Zhenan He, Gary G. Yen, and Zhang Yi. Robust multiobjective optimization via evolutionary algorithms. IEEE Transactions on Evolutionary Computation , 23(2):316330, 2019. doi: 10.1109/TEVC.2018.2859638.
- L. Jeff Hong. Estimating quantile sensitivities. Operations Research , 57(1):118-130, 2009. doi: 10.1287/opre.1080. 0531.
- L Jeff Hong, Zhaolin Hu, and Guangwu Liu. Monte carlo methods for value-at-risk and conditional value-at-risk: a review. ACM Transactions on Modeling and Computer Simulation , 24(4):1-37, 2014.
- Jonas Ide and Elisabeth Kobis. Concepts of efficiency for uncertain multi-objective optimization problems based on set order relations. Mathematical Methods of Operations Research , 80, 08 2014. doi: 10.1007/s00186-014-0471-z.

Hisao Ishibuchi, Naoya Akedo, and Yusuke Nojima. A many-objective test problem for visually examining diversity maintenance behavior in a decision space. In Proceedings of the 13th Annual Conference on Genetic and Evolutionary Computation , GECCO '11, page 649-656, New York, NY, USA, 2011. Association for Computing Machinery. ISBN 9781450305570. doi: 10. 1145/2001576.2001666. URL https://doi.org/ 10.1145/2001576.2001666 .

Hisao Ishibuchi, Ryo Imada, Yu Setoguchi, and Yusuke Nojima. How to specify a reference point in hypervolume calculation for fair performance comparison. Evol. Comput. , 26(3):411-440, September 2018.

Shogo Iwazaki, Yu Inatsu, and Ichiro Takeuchi. Meanvariance analysis in bayesian optimization under uncertainty. In Arindam Banerjee and Kenji Fukumizu, editors, Proceedings of The 24th International Conference on Artificial Intelligence and Statistics , volume 130 of Proceedings of Machine Learning Research , pages 973-981. PMLR, 13-15 Apr 2021.

Sergio Izquierdo, Julia Guerrero-Viu, Sven Hauns, Guilherme Miotto, Simon Schrodi, Andr'e Biedenkapp, Thomas Elsken, Difan Deng, Marius Lindauer, and Frank Hutter. Bag of baselines for multi-objective joint neural architecture search and hyperparameter optimization. In 8th ICML Workshop on Automated Machine Learning (AutoML) , 2021.

Miettinen Kaisa. Nonlinear Multiobjective Optimization , volume 12 of International Series in Operations Research & Management Science . Kluwer Academic Publishers, Boston, USA, 1999.

Diederik P Kingma and Max Welling. Auto-Encoding Variational Bayes. arXiv e-prints , page arXiv:1312.6114, Dec 2013.

- Johannes Kirschner, Ilija Bogunovic, Stefanie Jegelka, and Andreas Krause. Distributionally robust bayesian optimization. In Silvia Chiappa and Roberto Calandra, editors, Proceedings of the Twenty Third International Conference on Artificial Intelligence and Statistics , volume 108 of Proceedings of Machine Learning Research , pages 2174-2184. PMLR, 26-28 Aug 2020.
- J. Knowles. Parego: a hybrid algorithm with on-line landscape approximation for expensive multiobjective optimization problems. IEEE Transactions on Evolutionary Computation , 10(1):50-66, 2006.
- Renaud Lacour, Kathrin Klamroth, and Carlos M. Fonseca. A box decomposition algorithm to compute the hypervolume indicator. Computers & Operations Research , 79: 347 - 360, 2017.
- Benjamin Letham, Brian Karrer, Guilherme Ottoni, and Eytan Bakshy. Constrained bayesian optimization with noisy experiments. Bayesian Analysis , 14(2), 06 2019.
- Zhuangzhi Li and Zukui Li. Optimal robust optimization approximation for chance constrained optimization problem. Computers & Chemical Engineering , 74:89-99, 2015.
- Qiaohao Liang and Lipeng Lai. Scalable bayesian optimization accelerates process optimization of penicillin production. In NeurIPS 2021 AI for Science Workshop , 2021.
- Xingtao Liao, Qing Li, Xujing Yang, Weigang Zhang, and Wei Li. Multiobjective optimization for crash safety design of vehicles using stepwise regression model. Structural and Multidisciplinary Optimization , 35:561-569, 06 2008. doi: 10.1007/s00158-007-0163-x.

Mina Konakovic Lukovic, Yunsheng Tian, and Wojciech Matusik. Diversity-Guided Multi-Objective Bayesian Optimization With Batch Evaluations. In Advances in Neural Information Processing Systems 33 , 2020.

Dinah Elena Majewski, Marco Wirtz, Matthias Lampe, and Andr'e Bardow. Robust multi-objective optimization for sustainable design of distributed energy supply systems. Computers & Chemical Engineering , 102:26-39, 2017. ISSN 0098-1354. Sustainability & Energy Systems.

Gustavo Malkomes, Bolong Cheng, Eric H Lee, and Mike Mccourt. Beyond the pareto efficient frontier: Constraint active search for multiobjective experimental design. In Marina Meila and Tong Zhang, editors, Proceedings of the 38th International Conference on Machine Learning , volume 139 of Proceedings of Machine Learning Research , pages 7423-7434. PMLR, 18-24 Jul 2021.

Julien Marzat, Eric Walter, and H'el'ene Piet-Lahanier. Worstcase global optimization of black-box functions through kriging and relaxation. Journal of Global Optimization , 55:707-727, 04 2013. doi: 10.1007/s10898-012-9899-y.

Merve Meraklı and Simge Kuc¸ ukyavuz. Vectorvalued multivariate conditional value-at-risk. Operations Research Letters , 46(3):300-305, 2018. ISSN 0167-6377. doi: https://doi.org/10.1016/j.orl.2018.02. 006. URL https://www.sciencedirect.com/ science/article/pii/S0167637717304145 .

- S'everine Th'er'ese F.C. Mortier, Pieter-Jan Van Bockstal, Jos Corver, Ingmar Nopens, Krist V. Gernaey, and Thomas De Beer. Uncertainty analysis as essential step in the establishment of the dynamic design space of primary drying during freeze-drying. European Journal of Pharmaceutics and Biopharmaceutics , 103:71-83, 2016. ISSN 0939-6411.

Mojmir Mutny and Andreas Krause. Efficient high dimensional bayesian optimization with additivity and quadrature fourier features. In S. Bengio, H. Wallach, H. Larochelle, K. Grauman, N. Cesa-Bianchi, and R. Garnett, editors, Advances in Neural Information Processing Systems , volume 31. Curran Associates, Inc., 2018.

Quoc Phong Nguyen, Zhongxiang Dai, Bryan Kian Hsiang Low, and Patrick Jaillet. Optimizing conditional value-atrisk of black-box functions. In Thirty-Fifth Conference on Neural Information Processing Systems , 2021a.

Quoc Phong Nguyen, Zhongxiang Dai, Bryan Kian Hsiang Low, and Patrick Jaillet. Value-at-risk optimization with gaussian processes, 2021b.

Rafael Oliveira, Lionel Ott, and Fabio Ramos. Bayesian optimisation under uncertain inputs. In Kamalika Chaudhuri and Masashi Sugiyama, editors, Proceedings of the Twenty-Second International Conference on Artificial Intelligence and Statistics , volume 89 of Proceedings of Machine Learning Research , pages 1177-1184. PMLR, 16-18 Apr 2019.

Michael A. Osborne. Bayesian gaussian processes for sequential prediction, optimisation and quadrature. 2010.

Art B Owen. Quasi-monte carlo sampling. Monte Carlo Ray Tracing: Siggraph , 1:69-88, 2003.

- C.J. Paciorek. Nonstationary Gaussian Processes for Regression and Spatial Modelling . PhD thesis, Carnegie Mellon University, Pittsburgh, Pennsylvania, 2003.

Biswajit Paria, Kirthevasan Kandasamy, and Barnab'as P'oczos. A flexible framework for multi-objective bayesian optimization using random scalarizations, 2018.

Andr'as Pr'ekopa. Multivariate value at risk and related topics. Annals of Operations Research , 193:49-69, 2012.

- Ali Rahimi and Benjamin Recht. Random features for largescale kernel machines. In J. Platt, D. Koller, Y. Singer, and S. Roweis, editors, Advances in Neural Information Processing Systems , volume 20. Curran Associates, Inc., 2008.
- Carl Edward Rasmussen. Gaussian Processes in Machine Learning , pages 63-71. Springer Berlin Heidelberg, Berlin, Heidelberg, 2004.
- Tapabrata Ray and K.M. Liew. A swarm metaphor for multiobjective design optimization. Engineering Optimization , 34(2):141-153, 2002. doi: 10.1080/03052150210915.
- Justo Jos'e Roberts, Agnelo Marotta Cassula, Jos'e Luz Silveira, Edson da Costa Bortoni, and Andr'es Z. Mendiburu. Robust multi-objective optimization of a renewable based hybrid power system. Applied Energy , 223:52-68, 2018. ISSN 0306-2619.
- R.Tyrrell Rockafellar and Stanislav Uryasev. Conditional value-at-risk for general loss distributions. Journal of Banking & Finance , 26(7), 2002. ISSN 0378-4266. doi: 10.1016/S0378-4266(02)00271-6.
- Sergey Sarykalin, Gaia Serraino, and Stan Uryasev. Valueat-risk vs conditional value-at-risk in risk management and optimization. 09 2008. ISBN 978-1-877640-23-0. doi: 10.1287/educ.1080.0052.
- Robert J. Serfling. Approximation Theorems of Mathematical Statistics . Wiley Series in Probability and Statistics. Wiley-Blackwell, 2008. ISBN 9780470316481. doi: 10.1002/9780470316481.
- Pier Giuseppe Sessa, Ilija Bogunovic, Maryam Kamgarpour, and Andreas Krause. Mixed strategies for robust optimization of unknown objectives, 2020.
- Bobak Shahriari, Kevin Swersky, Ziyu Wang, Ryan P. Adams, and Nando de Freitas. Taking the human out of the loop: A review of Bayesian optimization. Proceedings of the IEEE , 104:1-28, 2016.
- Niranjan Srinivas, Andreas Krause, Sham Kakade, and Matthias Seeger. Gaussian process optimization in the bandit setting: No regret and experimental design. In Proceedings of the 27th International Conference on International Conference on Machine Learning , ICML'10, page 1015-1022, Madison, WI, USA, 2010. Omnipress. ISBN 9781605589077.
- Shinya Suzuki, Shion Takeno, Tomoyuki Tamura, Kazuki Shitara, and Masayuki Karasuyama. Multi-objective bayesian optimization using pareto-frontier entropy, 2020.
- G. Taguchi. Einfuhrung in Quality engineering: Minimierung von Verlusten durch Prozeßbeherrschung . American Supplier Inst., 1989. ISBN 9789283310846.
- Ryoji Tanabe and Hisao Ishibuchi. An easy-to-use realworld multi-objective optimization problem suite. Applied Soft Computing , 89:106078, 2020. ISSN 15684946.
- William R. Thompson. On the likelihood that one unknown probability exceeds another in view of the evidence of two samples. Biometrika , 25(3/4):285-294, 1933.
- L'eonard Torossian, Victor Picheny, and Nicolas Durrande. Bayesian quantile and expectile optimisation. arXiv: 2001.04833 , 2020.
- Saul Toscano-Palmerin and Peter I. Frazier. Bayesian optimization with expensive integrands. arXiv: 1803.08661 , 2018.
- Samee ur Rehman, Matthijs Langelaar, and Fred van Keulen. Efficient kriging-based robust optimization of unconstrained problems. Journal of Computational Science , 5 (6):872-881, 2014. ISSN 1877-7503.
- Zi Wang, Clement Gehring, Pushmeet Kohli, and Stefanie Jegelka. Batched large-scale bayesian optimization in high-dimensional spaces, 2018.
- James Wilson, Frank Hutter, and Marc Deisenroth. Maximizing acquisition functions for bayesian optimization. In Advances in Neural Information Processing Systems 31 , pages 9905-9916. 2018.
- James T. Wilson, Viacheslav Borovitskiy, Alexander Terenin, Peter Mostowsky, and Marc Peter Deisenroth. Efficiently sampling functions from gaussian process posteriors. In International Conference on Machine Learning , 2020.
- Xiangzhong Xie and Ren'e Schenkendorf. Robust process design in pharmaceutical manufacturing under batch-tobatch variation. Processes , 7(8), 2019. ISSN 2227-9717. doi: 10.3390/pr7080509.
- Kaifeng Yang, Michael Emmerich, Andr'e Deutz, and Thomas Back. Multi-objective bayesian global optimization using expected hypervolume improvement gradient. Swarm and Evolutionary Computation , 44:945 - 956, 2019. ISSN 2210-6502.
- Qi Zhou, Ping Jiang, Xiang Huang, Feng Zhang, and Taotao Zhou. A multi-objective robust optimization approach based on gaussian process model. Structural and Multidisciplinary Optimization , 57, 01 2018. doi: 10.1007/s00158-017-1746-9.

Ciyou Zhu, Richard H. Byrd, Peihuang Lu, and Jorge Nocedal. Algorithm 778: L-bfgs-b: Fortran subroutines for large-scale bound-constrained optimization. ACM Trans. Math. Softw. , 23(4):550-560, 1997. ISSN 00983500. doi: 10.1145/279232.279236. URL https: //doi.org/10.1145/279232.279236 .

## A. Theory and Proofs

## A.1. Preliminaries

Definition A.1. Let F = { f ( x ) : x ∈ X ⊆ X} . The weakly efficient Pareto frontier is WEAKPARETO ( F ) = { f ( x ) ∈ F : glyph[notexistential] x ' ∈ X s.t. f ( x ' ) > f ( x ) } , and PARETO ( F ) ⊆ WEAKPARETO ( F ) . If there are constraints c ( x ) , elements of WEAKPARETO are subject to the additional membership Assumption that c ( x ) ≥ 0 . We call the corresponding set of optimal designs the weak Pareto set .

Definition A.2. WEAK-MVAR is defined in the same way as MVAR, but only requires that its elements are weakly Pareto optimal.

Definition A.3 (Pr'ekopa (2012)) . If Assumption 5.1 holds, then we can express MVAR with an equality with respect to α :

MVAR α [ f ( x glyph[diamondmath] ξ ) ] = { z ∈ R M : P [ f ( x glyph[diamondmath] ξ ) ≥ z ] = α } . (1)

## A.2. Proofs

Lemma A.1. Let y ∈ R M and v ∈ R . Then s [ y , w ] ≥ v ⇐⇒ y ≥ v w .

Proof. This follows directly from Definition 5.1.

s [ y , w ] ≥ v ⇐⇒ min i w i y i ≥ v ⇐⇒ w i y i ≥ v ∀ i ⇐⇒ y i ≥ v w i ∀ i ⇐⇒ y ≥ v w .

Lemma A.1 states that a lower bound v on the value of a Chebyshev scalarization of an objective vector y can be used to define a point, v w , that y is greater than or equal to. We can extend this to make a similar statement about the VAR of a Chebyshev scalarization.

Lemma 5.1 (VAR of Chebyshev scalarization ⇒ Pareto Dominance) . Let v = VAR α ( s [ f ( x glyph[diamondmath] ξ ) , w ] ) . Then, P [ f ( x glyph[diamondmath] ξ ) ≥ v w ] ≥ α , where v w denotes element-wise division.

Proof. From Definition 4.1, we have that

VAR α [ f ( x glyph[diamondmath] ξ ) ] = sup { z ∈ R : P [ f ( x glyph[diamondmath] ξ ) ≥ z ] ≥ α } .

Hence, P ( s [ f ( x glyph[diamondmath] ξ ) , w ] ≥ v ) ≥ α . From Definition 5.1, we have P (min i [ w i f i ( x glyph[diamondmath] ξ )] ≥ v ) ≥ α. By Lemma A.1, the statement min i [ w i f i ( x glyph[diamondmath] ξ )] ≥ v is equivalent to f ( x glyph[diamondmath] ξ ) ≥ v w . Hence, P ( f ( x glyph[diamondmath] ξ ) ≥ v w ) ≥ α.

Lemma A.2 (VAR of Chebyshev scalarization ⇒ WEAK-MVAR) . Let v = VAR α ( s [ f ( x glyph[diamondmath] ξ ) , w ] ) . Then, v w ∈ WEAK-MVAR α [ f ( x glyph[diamondmath] ξ ) ] .

Proof. Let z = v w . Suppose there exists z ' ∈ WEAK-MVAR α ( f ( x glyph[diamondmath] ξ )) such that z ' > z . Since z ' ∈ WEAK-MVAR α ( f ( x glyph[diamondmath] ξ )) , we have that P ( f ( x glyph[diamondmath] ξ ) ≥ z ' ) ≥ α . Note that f ( x glyph[diamondmath] ξ ) ≥ z ' implies that min i w i f i ( x glyph[diamondmath] ξ ) ≥ min i w i z ' i . Hence, P ( f ( x glyph[diamondmath] ξ ) ≥ z ' ) ≥ α implies that P ( s [ f ( x glyph[diamondmath] ξ ) , w ] ≥ s [ z ' , w ]) ≥ α . Since z ' > z and w ∈ ∆ M -1 + , we have that s [ z ' , w ] > s [ z , w ] = v . But this contradicts Definition 4.1. Since there does not exist z ' ∈ WEAK-MVAR α [ f ( x glyph[diamondmath] ξ ) ] such that z ' > z , we have that z ∈ WEAK-MVAR α [ f ( x glyph[diamondmath] ξ ) ] .

If Assumption 5.1 holds, the inequalities with respect to α in Lemma 5.1 become equalities, and we show that v w is strictly Pareto optimal.

Lemma A.3 (VAR of Chebyshev scalarization ⇒ MVAR) . Let v = VAR α [ s [ f ( x glyph[diamondmath] ξ ) , w ] ] . If Assumption 5.1 holds, then v w ∈ MVAR α [ f ( x glyph[diamondmath] ξ ) ] .

Proof. Let z := v w . Suppose there exists z ' ∈ MVAR α [ f ( x glyph[diamondmath] ξ ) ] such that z ' glyph[follows] z . Because F ( · ) is a strictly increasing CDF, P ( f ( x glyph[diamondmath] ξ ) ≥ z ' ) = F ( z ' ) > F ( z ) = P ( f ( x glyph[diamondmath] ξ ) ≥ z ) . From Lemma 5.1, we have that P ( f ( x glyph[diamondmath] ξ ) ≥ z ) ≥ α . Because z ' ∈ MVAR α [ f ( x glyph[diamondmath] ξ ) ] from Equation (1), we have that P ( f ( x glyph[diamondmath] ξ ) ≥ z ' ) = α . Hence, P ( f ( x glyph[diamondmath] ξ ) ≥ z ) ≥ P ( f ( x glyph[diamondmath] ξ ) ≥ z ' ) . This is a contradiction.

Lemma A.3 provides a technique for generating points in the MVAR set using the VAR of Chebyshev scalarizations with different w .

Now, consider the reverse mapping.

Lemma A.4 (MVAR ⇒ VAR of Chebyshev scalarization) . Suppose that z ∈ MVAR α [ f ( x glyph[diamondmath] ξ ) ] . Then z = v w for w := 1 z || 1 z || -1 1 ∈ ∆ M + and v = VAR α ( s [ f ( x glyph[diamondmath] ξ ) , w ] ) .

Proof. Without loss of generality, assume that z > 0 . 7 Let v = || 1 z || -1 ∈ R + . Then, z = v w . Hence, all we need to show is that v equals VAR α ( s [ f ( x glyph[diamondmath] ξ ) , w ] ) . Let us define v ' := VAR α ( s [ f ( x glyph[diamondmath] ξ ) , w ] ) . By definition,

v ' = sup { v '' ∈ R : P ( s [ f ( x glyph[diamondmath] ξ ) , w ] ≥ v '' ] ≥ α } .

Since z ∈ MVAR α [ f ( x glyph[diamondmath] ξ ) ] , P [ f ( x glyph[diamondmath] ξ ) ≥ z ] ≥ α . Hence, P [ f ( x glyph[diamondmath] ξ ) ≥ v w ] ≥ α . Using Lemma A.1, we have that P ( s [ f ( x glyph[diamondmath] ξ ) , w ] ≥ v ) ≥ α . Since v ' is the supremum, we have that v ≤ v ' . Suppose now that v < v ' . Note that v w = z ∈ MVAR α [ f ( x glyph[diamondmath] ξ ) ] . From Lemma A.2, v ' w ∈ WEAK-MVAR α ( f ( x glyph[diamondmath] ξ ) ) . Since v < v ' , z = v w ≺ v ' w . Hence, z cannot be in MVAR α [ f ( x glyph[diamondmath] ξ ) ] because it is dominated by v ' w and v ' w ∈ WEAK-MVAR α ( f ( x glyph[diamondmath] ξ ) ) . This is a contradiction. Hence v ≥ v ' and therefore it follows that v = v ' = VAR α ( s [ f ( x glyph[diamondmath] ξ ) , w ] ) .

Let h : MVAR α [ f ( x glyph[diamondmath] ξ ) ] → ∆ M -1 + be given by h ( z ) = 1 z || 1 z || . Being the element-wise application of a scalar injective mapping ( z ↦→ 1 /z ) , it is clear that h is injective. However, h ( · ) is not necessarily bijective, since without Assumption 5.1 there may be weight vectors w such that glyph[notexistential] z ∈ MVAR α [ f ( x glyph[diamondmath] ξ ) ] s.t . h ( z ) = w .

Theorem 5.1 (MVAR ⇐⇒ VAR of Chebyshev scalarization) . Given x , f , and P ( ξ ) , let h : MVAR α [ f ( x glyph[diamondmath] ξ ) ] → ∆ M -1 + be the function h ( z ) = w = 1 z || 1 z || . Under Assumption 5.1, h ( · ) is bijective and h -1 ( w ) = z = 1 w VAR α ( s [ f ( x glyph[diamondmath] ξ ) , w ] ) .

Proof. This follows directly from Lemma A.3 and Lemma A.4.

Corollary A.1 (MVAR via Scalarization) . WEAK-MVAR enjoys the following scalarization representation: WEAK-MVAR α ( f ( x glyph[diamondmath] ξ ) ) = { 1 w VAR α ( s [ f ( x glyph[diamondmath] ξ ) , w ] ) : w ∼ ∆ M -1 + } . If Assumption 5.1 holds, then MVAR α [ f ( x glyph[diamondmath] ξ ) ] = { 1 w VAR α ( s [ f ( x glyph[diamondmath] ξ ) , w ] ) : w ∼ ∆ M -1 + } .

Although the MVAR representation in Corollary A.1 depends on Assumption 5.1, Lemma 5.1 recovers all weakly Pareto optimal points even if this assumption is not met because MVAR ⊆ WEAK-MVAR. Hence, with or without Assumption 5.1, Theorem A.1 can be used to approximate the MVAR set.

Result A.1 (MVAR Approximation) . MVAR can be approximated with a finite set of weight vectors: MVAR ∧ ( f ( x glyph[diamondmath] ξ ) ) = { 1 w i VAR α ( s [ f ( x glyph[diamondmath] ξ ) , w i ] ) } N MVAR i =1 .

## A.3. Discussion of the Assumption of Continuous, Strictly-Increasing CDFs with Gaussian Processes

In Bayesian Optimization with Gaussian Process surrogates, it is assumed that the objective function f is sample path from a Gaussian process prior. In this setting, the random variable f ( x glyph[diamondmath] ξ ) for a deterministic sample path f (where the

stochasticity comes solely from ξ ) has a continuous, strictly increasing CDF F ( · ) for many commonly used covariance functions. Hence, the bijective relationship in Theorem 5.1 holds given a suitable choice of covariance function. 8

Lemma A.5. If f is a sample path from a multi-output Gaussian Process prior where the covariance function and the covariance function of the derivative process are strictly positive definite and with sample paths that are differentiable, 9 X is a compact set, and P ( x glyph[diamondmath] ξ ) is a continuous distribution with strictly positive support, then Assumption 5.1 holds.

glyph[negationslash]

Proof. Consider the case of a scalar function f . By Lemma 1 of Cakmak et al. (2020), the density of f ( x glyph[diamondmath] ξ ) is strictly positive. Hence, any interval with positive Lebesgue measure has non-zero density. So the cumulative density function of f ( x glyph[diamondmath] ξ ) is strictly increasing. Consider the case of a multi-output sample path f . Suppose that the joint CDF is not strictly increasing. Then there exist y , y ' such that y ≥ y ' and there exists at least one i ∈ { 1 , ..., M } s.t. y i > y ' i and F ( y ) ≤ F ( y ' ) . Since y ≥ y ' and F is a CDF, F ( y ) ≥ F ( y ' ) . Hence, F ( y ) = F ( y ' ) . So, we have P ( f 1 ( x glyph[diamondmath] ξ ) ≤ y 1 , ..., f M ( x glyph[diamondmath] ξ ) ≤ y M ) = P ( f 1 ( x glyph[diamondmath] ξ ) ≤ y ' 1 , ..., f M ( x glyph[diamondmath] ξ ) ≤ y ' M ) . Suppose y ' i = y i for all i = j . Then, P ( f 1 ( x glyph[diamondmath] ξ ) ≤ y 1 , y ' j < f j ( x glyph[diamondmath] ξ ) ≤ y j , ..., f M ( x glyph[diamondmath] ξ ) ≤ y M ) = 0 . But the hyperrectangle bounded by [ -∞ , ..., y ' j , ..., -∞ ] and y has positive Lebesgue measure. Since the pdf of each of f 1 , ..., f M is strictly positive, the cumulative density over the hyperrectangle is greater than zero, which is a contradiction. The argument is easily extended to the case when there exists 1 ≥ k ≥ M indices i 1 , ..., i k such that y ' i k < y i k .

## A.4. Extension to Black-Box Constraints Under Input Noise

In this section, we consider the setting where in addition to the objective function f there is a vector-valued black-box function c ( x ) ∈ R V specifying the outcome constraint c ( x ) > 0 that is also subject to input noise ξ ∼ P ( ξ ) . To handle black-box constraints under input noise, we weight the objectives f ( x glyph[diamondmath] ξ ) by a feasibility indicator ✶ [ c ( x glyph[diamondmath] ξ ) > 0 ] that is 1 if all constraints are satisfied and 0 otherwise. We define VAR and MVAR for feasibility-weighted objectives and extend the theoretical results from Section 5.1.

The proofs for the results in the constrained setting follow the proofs in Appendix A.2 with slight modifications. 1) Assumption 5.1 does not hold for feasibility-weighted objectives. Hence, the implication is that a random sampled scalarization is only guaranteed to correspond to a point in the WEAK-MVAR set, but importantly any point in the MVAR set does correspond to some scalarization and therefore can be recovered. 2) The proofs handle the special case where some of the constraints are not satisfied and the feasibility-weighted objectives are zero.

Definition A.4. The value-at-risk of the feasibility-weighted objective for a given point x is:

VAR α ( f ( x glyph[diamondmath] ξ ) ✶ [ c ( x glyph[diamondmath] ξ ) > 0 ] ) = sup { z ∈ R : P ( f ( x glyph[diamondmath] ξ ) ✶ [ c ( x glyph[diamondmath] ξ ) > 0 ] ≥ z ) ≥ α } .

Definition A.5. The MVAR of the feasibility-weighted objectives f for a given point x is:

MVAR α ( f ( x glyph[diamondmath] ξ ) , c ( x glyph[diamondmath] ξ ) ) = { z ∈ R M : P ( f ( x glyph[diamondmath] ξ ) ✶ [ c ( x glyph[diamondmath] ξ ) > 0 ] ≥ z ) ≥ α, glyph[notexistential] z ' ∈ R M , z ' glyph[follows] z , P ( f ( x glyph[diamondmath] ξ ) ✶ [ c glyph[follows] 0 ] ≥ z ' ) ≥ α } .

Let M c ξ ,X := ⋃ x ∈ X MVAR α [ f ( x glyph[diamondmath] ξ ) , c ( x glyph[diamondmath] ξ ) ] . The global MVAR of the feasibility weighted objectives for a set of points X is defined as

MVAR P ( ξ ) ,α ( { f ( x glyph[diamondmath] ξ ) , c ( x glyph[diamondmath] ξ ) } x ∈ X ) = { z ∈ M c ξ ,X : glyph[notexistential] z ' ∈ M c ξ ,X s.t. z ' glyph[follows] z } .

WEAK-MVAR of the feasibility-weighted objectives is defined in the same way, but only requires that its elements are weakly Pareto optimal

Lemma A.6. Given a weight vector w ∈ ∆ M -1 + , y ∈ R M , y c ∈ R M ' and v ∈ R ,

s [ y , w ] ✶ [ y c glyph[follows] 0 ] ≥ v ⇐⇒ y ✶ [ y c glyph[follows] 0 ] ≥ v w .

Proof. This follows directly from Definition 5.1.

s [ y , w ] ✶ [ y c glyph[follows] 0 ] ≥ v ⇐⇒ ✶ [ y c glyph[follows] 0 ] min i w i y i ≥ v ⇐⇒ w i y i ✶ [ y c glyph[follows] 0 ] ≥ v ∀ i ⇐⇒ y i ✶ [ y c glyph[follows] 0 ] ≥ v w i ∀ i ⇐⇒ y ✶ [ y c glyph[follows] 0 ] ≥ v w .

Theorem A.1 (VAR of Feasibility-Weighted Chebyshev scalarization ⇒ Pareto Dominance) . Given a weight vector w ∈ ∆ M -1 + , let v = VAR α ( s [ f ( x glyph[diamondmath] ξ ) , w ] ✶ [ c ( x glyph[diamondmath] ξ ) > 0 ] ) . Then,

P ( f ( x glyph[diamondmath] ξ ) ✶ [ c ( x glyph[diamondmath] ξ ) > 0 ] ≥ v w ) ≥ α.

Proof. From Definition A.4, we have that

VAR α ( s [ f ( x glyph[diamondmath] ξ ) , w ] ✶ [ c ( x glyph[diamondmath] ξ ) > 0 ] ) = sup { z ∈ R : s [ f ( x glyph[diamondmath] ξ ) , w ] ✶ [ c ( x glyph[diamondmath] ξ ) > 0 ] ≥ z ) ≥ α } .

Hence,

P ( s [ f ( x glyph[diamondmath] ξ ) , w ] ✶ [ c ( x glyph[diamondmath] ξ ) > 0 ] ≥ v ) ≥ α.

From Definition 5.1, we have

P ( ✶ [ c ( x glyph[diamondmath] ξ ) > 0 ] min i w i f ( i ) ( x glyph[diamondmath] ξ ) ≥ v ) ≥ α.

By Lemma A.6, the following expressions are equivalent

✶ [ c ( x glyph[diamondmath] ξ ) > 0 ] min i w i f ( i ) ( x glyph[diamondmath] ξ ) ≥ v ⇐⇒ f ( x glyph[diamondmath] ξ ) ✶ [ c ( x glyph[diamondmath] ξ ) > 0 ] ≥ v w .

Hence, P ( f ( x glyph[diamondmath] ξ ) ✶ [ c ( x glyph[diamondmath] ξ ) > 0 ] ≥ v w ) ≥ α.

Lemma A.7. Let z ∈ MVAR α [ f ( x glyph[diamondmath] ξ ) , c ( x glyph[diamondmath] ξ ) ] . If f ( x ) > 0 , then z glyph[notfollows] 0 if and only if z = 0 .

Proof. The following shows that, since f ( x ) > 0 , z glyph[notfollows] 0 if and only if P ( c ( x glyph[diamondmath] ξ ) > 0 ) < α . Thus, z glyph[notfollows] 0 if and only if z = 0 .

Since f ( x ) > 0 , we have that z i ≥ 0 for all i = 1 , ..., M and there exists j ∈ { 1 , ..., M } such that z j = 0 . Note that since f ( x ) > 0 , the i th element f ( i ) ( x ) ✶ [ c ( x ) > 0 ] = 0 if and only if ✶ [ c ( x ) > 0 ] = 0 . Hence, either f ( x ) ✶ [ c ( x ) > 0 ] > 0 or f ( x ) ✶ [ c ( x ) > 0 ] = 0 .

From Definition A.5, we have that P ( f ( x glyph[diamondmath] ξ ) ✶ [ c ( x glyph[diamondmath] ξ ) > 0 ] ≥ z ) ≥ α. Since f ( x glyph[diamondmath] ξ ) ✶ [ c ( x glyph[diamondmath] ξ ) > 0 ] > 0 or f ( x glyph[diamondmath] ξ ) ✶ [ c ( x glyph[diamondmath] ξ ) > 0 ] = 0 ,

P ( f ( x glyph[diamondmath] ξ ) ✶ [ c ( x glyph[diamondmath] ξ ) > 0 ] > 0 ) + P ( f ( x glyph[diamondmath] ξ ) ✶ [ c ( x glyph[diamondmath] ξ ) > 0 ] = 0 ) = 1 .

Suppose z glyph[notfollows] 0 . Since z is not dominated by any other point in the MVAR set, P ( f ( x glyph[diamondmath] ξ ) ✶ [ c ( x glyph[diamondmath] ξ ) > 0 ] glyph[follows] 0 ) < α. Hence, z must be 0 .

Lemma A.8 (VAR of Feasibility-Weighted Chebyshev scalarization ⇒ Feasibility-Weighted WEAK-MVAR) . Let v = VAR α ( s [ f ( x glyph[diamondmath] ξ ) ✶ [ c ( x glyph[diamondmath] ξ ) > 0 ] , w ] ) . Then, v w ∈ WEAK-MVAR α ( f ( x glyph[diamondmath] ξ ) , c ( x glyph[diamondmath] ξ ) ) .

Proof. Without loss of generality, assume that the objectives f ( x ) > 0 . 7 Let z = v w . Suppose there exists z ' ∈ WEAK-MVAR α ( f ( x glyph[diamondmath] ξ ) , c ( x glyph[diamondmath] ξ )) such that z ' > z . Since z ' ∈ WEAK-MVAR α ( f ( x glyph[diamondmath] ξ ) , c ( x glyph[diamondmath] ξ )) ,

P ( f ( x glyph[diamondmath] ξ ) ✶ [ c ( x glyph[diamondmath] ξ ) > 0 ] ≥ z ' ) ≥ α.

Note that f ( x glyph[diamondmath] ξ ) ✶ [ c ( x glyph[diamondmath] ξ ) > 0 ] ≥ z ' implies that

✶ [ c ( x glyph[diamondmath] ξ ) > 0 ] min i w i f ( i ) ( x glyph[diamondmath] ξ ) ≥ min i w i z i .

Hence, P ( f ( x glyph[diamondmath] ξ ) ✶ [ c ( x glyph[diamondmath] ξ ) > 0 ] ≥ z ' ) ≥ α implies that

P ( s [ f ( x glyph[diamondmath] ξ ) , w ] ✶ [ c ( x glyph[diamondmath] ξ ) > 0 ] ≥ s [ z ' , w ]) ≥ α.

Note that s [ z ' , w ] > s [ z , w ] = v . But by the Definition A.4, v is the maximum value such that

P ( s [ f ( x glyph[diamondmath] ξ ) , w ] ✶ [ c ( x glyph[diamondmath] ξ ) > 0 ] ≥ v ) ≥ α.

This is a contradiction.

Theorem A.2 (Feasibility-Weighted MVAR ⇒ VAR of Feasibility-Weighted Chebyshev scalarization) . For any z ∈ MVAR α [ f ( x glyph[diamondmath] ξ ) , c ( x glyph[diamondmath] ξ ) ] , there exists w ∈ ∆ M -1 + such that z = v w where v = VAR α ( s [ f ( x glyph[diamondmath] ξ ) , w ] ✶ [ c ( x glyph[diamondmath] ξ ) > 0 ] ) .

Proof. Without loss of generality, assume that the objectives f ( x ) > 0 . 7

Case 1 : z glyph[notfollows] 0 . From Lemma A.7, we have that z = 0 . Note that since the MVAR set contains only non-dominated points and 0 is a lower bound on f ( x ) ✶ [ c ( x ) > 0 ] , MVAR α [ f ( x glyph[diamondmath] ξ ) , c ( x glyph[diamondmath] ξ ) ] = { 0 } . Let v := VAR α ( s [ f ( x glyph[diamondmath] ξ ) , w ] ✶ [ c ( x glyph[diamondmath] ξ ) > 0 ] ) .

Suppose v = 0 . Then, for any w ∈ ∆ M -1 + , v w = 0 ∈ MVAR α [ f ( x glyph[diamondmath] ξ ) , c ( x glyph[diamondmath] ξ ) ] .

Suppose v > 0 . Then for any w ∈ ∆ M -1 + , v w > 0 . By Lemma A.8, v w ∈ WEAK-MVAR α ( f ( x glyph[diamondmath] ξ ) , c ( x glyph[diamondmath] ξ ) ) . But MVAR α [ f ( x glyph[diamondmath] ξ ) , c ( x glyph[diamondmath] ξ ) ] = { 0 } , and any point in the MVAR set is non-dominated. So v w glyph[notfollows] 0 . This is a contradiction.

Hence, v = 0 . Therefore, Theorem A.2 holds when z glyph[notfollows] 0 .

## Case 2 : z glyph[follows] 0 .

Consider the vector 1 z . If we divide 1 z by its L1-norm, we obtain a vector w := 1 z || 1 z || -1 1 ∈ ∆ M + , where || · || 1 denotes the L1norm. Let v = 1 || 1 z || 1 ∈ R + . Then, z = v w . Hence, all we need to show is that v = VAR α ( s [ f ( x glyph[diamondmath] ξ ) , w ] ✶ [ c ( x glyph[diamondmath] ξ ) > 0 ] ) . By definition,

VAR α ( s [ f ( x glyph[diamondmath] ξ ) , w ] ✶ [ c ( x glyph[diamondmath] ξ ) > 0 ] ) = sup { v '' ∈ R : P ( s [ f ( x glyph[diamondmath] ξ ) , w ] ✶ [ c ( x glyph[diamondmath] ξ ) > 0 ] ≥ v '' ) ≥ α } .

Let us define v ' := VAR α ( s [ f ( x glyph[diamondmath] ξ ) , w ] ✶ [ c ( x glyph[diamondmath] ξ ) > 0 ] ) . Suppose that v < v ' . By Lemma A.8, v ' w ∈ WEAK-MVAR α ( f ( x glyph[diamondmath] ξ ) , c ( x glyph[diamondmath] ξ ) ) . Since v < v ' , z = v w ≺ v ' w . But z is in MVAR α [ f ( x glyph[diamondmath] ξ ) ] . So by Definition 4.2, z is not dominated by any other vector in WEAK-MVAR α ( f ( x glyph[diamondmath] ξ ) , c ( x glyph[diamondmath] ξ ) ) . This is a contradiction. Hence, v = VAR α ( s [ f ( x glyph[diamondmath] ξ ) , w ] ✶ [ c ( x glyph[diamondmath] ξ ) > 0 ] ) . Thus, Theorem A.2 holds when z glyph[follows] 0 .

Corollary A.2. There is a injective function g : MVAR α [ f ( x glyph[diamondmath] ξ ) , c ( x glyph[diamondmath] ξ ) ] → ∆ M -1 + such that g ( z ) = 1 z || 1 z || = w and z = v w where v = VAR α ( s [ f ( x glyph[diamondmath] ξ ) , w ] ) = 1 || 1 || .

z

Corollary A.3. Suppose z is a point in the global MVAR set z ∈ MVAR α [ { f ( x glyph[diamondmath] ξ ) , c ( x glyph[diamondmath] ξ ) } x ∈X ] . Let X ∗ z be the set of designs such that for all x ∈ X ∗ z , z ∈ MVAR α [ f ( x glyph[diamondmath] ξ ) , c ( x glyph[diamondmath] ξ ) ] . Then every x ∈ X ∗ z is a maximizer of VAR α ( s [ f ( x glyph[diamondmath] ξ ) , w ] ✶ [ c ( x glyph[diamondmath] ξ ) > 0 ] ) .

glyph[negationslash]

When MVAR α [ f ( x glyph[diamondmath] ξ ) , c ( x glyph[diamondmath] ξ ) ] = { 0 } , it follows directly from the injective mapping from z to w that VAR α ( s [ f ( x glyph[diamondmath] ξ ) , w ] ✶ [ c ( x glyph[diamondmath] ξ ) > 0 ] ) is the same for all x ∈ X ∗ z . When MVAR α [ f ( x glyph[diamondmath] ξ ) , c ( x glyph[diamondmath] ξ ) ] = { 0 } , then all x are infeasible designs and X ∗ z = X .

## B. MARS with Alternative Acquisition Functions

In this section, we discuss using MARS with two alternative acquisition functions: Thompson Sampling (TS) and Upper Confidence Bound (UCB).

## B.1. MARS with Thompson Sampling

As discussed in Section 5, direct MVAR optimization with q NEHVI requires evaluating the joint posterior over n ξ ( n +1) designs. The same is true when using MARS-NEI. Although low-rank Cholesky updates can significantly reduce the complexity (Osborne, 2010), further computational improvements can be obtained by using TS with random Fourier features (RFFs) (Rahimi and Recht, 2008). However, RFFs are approximate GP samples and introduce approximation error (see Appendix D.3 for further discussion). 10 We refer to this method as MARS-TS. MARS-TS naturally supports (i) parallel candidate generation by drawing a new posterior sample and new scalarization weights for each candidate; (ii) constraints, by evaluating the feasibility-weighted objectives under the posterior sample, and (iii) noisy observations.

## B.2. MARS with Upper Confidence Bound

Another computationally efficient approach is to use Upper Confidence Bound (UCB), which does not require the expensive integration over f ( x glyph[diamondmath] ξ ) where x ∈ X 1: n . We refer to this method as MARS-UCB. In what follows, we show how to extend the V-UCB algorithm of Nguyen et al. (2021b) to optimize the VAR of Chebyshev scalarizations. The result builds on the following lemma by Chowdhury and Gopalan (2017), which holds under the assumption that the function f ( i ) ( · ) belongs to a reproducing kernel Hilbert space (RKHS) F k i ( B i ) , whose RKHS norm is bounded by ‖ f ( i ) ‖ k i ≤ B i , where f = [ f (1) , ..., f ( M ) ] . We use µ ( i ) n ( x ) , Σ ( i ) n ( x, x ) to denote the posterior, conditional on observations up to iteration n , mean and variance of the GP surrogate corresponding to i th objective, and use σ 2 i to denote the observation noise for the i th objective.

Lemma B.1. (Chowdhury and Gopalan, 2017). For δ ∈ (0 , 1) , ζ ( i ) n +1 = B i + σ 2 i √ 2( γ n +1+log(1 /δ )) , the following holds for all x ∈ X with probability ≥ 1 -δ :

l ( i ) n ( x ) ≤ f ( i ) ( x ) ≤ u ( i ) n ( x ) , (2)

where l ( i ) n ( x ) := µ ( i ) n ( x ) -ζ ( i ) n +1 (Σ ( i ) n ( x , x )) 1 / 2 , u ( i ) n ( x ) := µ ( i ) n ( x ) + ζ ( i ) n +1 (Σ ( i ) n ( x , x )) 1 / 2 , and γ n denotes the maximum information gain.

Assuming that each objective is modeled using an independent GP surrogate and considering all objectives jointly, we see that (2) holds jointly for all i = 1 , . . . , m with probability at least (1 -δ ' ) := (1 -δ ) m . Applying the Chebyshev scalarization,

w i l ( i ) n ( x ) ≤ w i f ( i ) ( x ) ≤ w i u ( i ) n ( x ) , ∀ i = 1 , . . . , M min i w i l ( i ) n ( x ) ≤ min i w i f ( i ) ( x ) ≤ min i w i u ( i ) n ( x ) s [ l n ( x ) , w ] ≤ s [ f ( x ) , w ] ≤ s [ u n ( x ) , w ]

holds with probability ≥ (1 -δ ' ) for all x ∈ X . Following Lemma 2 of (Nguyen et al., 2021b), we get that

VAR α ( s [ l n ( x glyph[diamondmath] ξ ) , w ]) ≤ VAR α ( s [ f ( x glyph[diamondmath] ξ ) , w ]) ≤ VAR α ( s [ u n ( x glyph[diamondmath] ξ ) , w ])

holds with probability at least (1 -δ ' ) . Thus, the UCB policy for VAR of Chebyshev scalarization is defined as the policy that samples x n +1 = arg max x VAR α ( s [ u n ( x glyph[diamondmath] ξ ) , w ]) .

In practice, computing the ζ ( i ) n +1 given in Lemma B.1 is impractical, and typically leads to an acquisition function that is overly conservative. Thus, we follow Nguyen et al. (2021b) and use ζ ( i ) n +1 = 2log( n 2 π 2 / 0 . 6) in the experiments.

MARS-UCB can be extended to support parallel candidate generation by sampling a new scalarization for each candidate and noisy observations. The UCB policy derived above does not hold for feasibility-weighted objectives because Lemma B.1 requires that f ( i ) ( · ) belongs to a RKHS, and this is not the case when f ( i ) ( · ) is weighted by a feasibility indicator because it is no longer continuous (de Freitas et al., 2012).

glyph[negationslash]

## C. Gradient-based Acquisition Function Optimization

## C.1. Approximate Gradients of VAR

One of the earliest and simplest-to-use gradient estimators for VAR was presented by Hong (2009). Under mild regularity assumptions on the distribution of the random variable, they establish the consistency of the VAR gradient estimator, which can be seen as the sample-path gradient of the well-known estimator of VAR. For this discussion, let g ( · ) be a deterministic function of its argument (e.g., a sample path of the GP), let us fix x , and let ξ ∼ P ( ξ ) be a continuous random variable. Let ξ 1 , . . . , ξ k denote i.i.d. samples from P ( ξ ) . Define the following ordering of the samples, where the subscript ( · ) denote the order statistic:

g ( x glyph[diamondmath] ξ (1) ) ≤ g ( x glyph[diamondmath] ξ (2) ) ≤ . . . ≤ g ( x glyph[diamondmath] ξ ( k ) ) .

The VAR at risk level α can be estimated by

VAR ξ ∼ P ( ξ ) ( g ( x glyph[diamondmath] ξ )) ≈ g ( x glyph[diamondmath] ξ ( glyph[floorleft] (1 -α ) k glyph[floorright] ) ) ,

where glyph[floorleft]·glyph[floorright] denotes the largest integer less than or equal to · . It is well known (cf. Serfling (2008)) that this estimator is consistent as k →∞ . Hong (2009) extend this result to show that the corresponding gradient estimator

∇ x VAR ξ ∼ P ( ξ ) ( g ( x glyph[diamondmath] ξ )) ≈ ∇ x g ( x glyph[diamondmath] ξ ( glyph[floorleft] (1 -α ) k glyph[floorright] ) )

is an asymptotically (as k → ∞ ) unbiased estimator of the gradient of VAR. The estimator is also consistent as long as ∇ x g ( x glyph[diamondmath] ξ ( glyph[floorleft] (1 -α ) k glyph[floorright] ) ) is not a function of ξ , otherwise, averaging of multiple sample gradients is required to obtain a consistent estimator of the gradient of VAR.

In addition to the sample-path gradient estimator discussed above, there are other estimators of gradients of VAR that are based on, e.g., the likelihood ratio gradient estimation or on the kernel density estimators. A detailed discussion of these can be found in Hong et al. (2014).

## C.2. Approximate Gradients of MVAR

Differentiability of MVAR, more precisely the differentiability of the elements of the MVAR set, is a subject that has not been explored in the literature. Since the computation of the MVAR set corresponding to a set of posterior samples is expensive enough to be the bottleneck during acquisition function optimization, it is highly desirable to avoid the finite-difference gradient estimation, which requires multiple evaluations of the objective and is in general less efficient than the sample-path gradients. Instead, it is preferable to establish a direct connection between the MVAR set and the gradients of the samples on which the MVAR is computed. The method we discuss below is inspired by the gradients of VAR, which correspond to the gradients of the sample that is equal to VAR.

glyph[negationslash]

The correspondence between ∇ x VAR ξ ∼ P ( ξ ) ( g ( x glyph[diamondmath] ξ )) and ∇ x g ( x glyph[diamondmath] ξ ( glyph[floorleft] (1 -α ) k glyph[floorright] ) ) follows from the observation that, since g ( · ) is a continuous function, shifting x by a sufficiently small glyph[epsilon1] should not change the ordering of ξ 's. We should still have VAR ξ ∼ P ( ξ ) ( g ( x + glyph[epsilon1] + ξ )) ≈ g ( x + glyph[epsilon1] + ξ ( glyph[floorleft] (1 -α ) k glyph[floorright] ) ) with the same ordering, as long as g ( x glyph[diamondmath] ξ ( i ) ) = g ( x glyph[diamondmath] ξ ( j ) ) for i = j . The same idea extends to the MVAR. Using a finite set of samples to approximate the MVAR set, with m ( x glyph[diamondmath] ξ ) denoting an arbitrary element of the MVAR set, we have that m ( j ) ( x glyph[diamondmath] ξ ) = f ( j ) ( x glyph[diamondmath] ξ i ( j ) ) for some i ( j ) ∈ { 1 , . . . , k } , where f = [ f (1) , ..., f ( M ) ] and the i ( j ) is dependent on the outcome j and the particular element of the MVAR set. This can be interpreted as saying that the elements of the MVAR set are constructed by piecing together outcomes from the samples of the random variable.

glyph[negationslash]

Similar to what was discussed for VAR, if we perturb x by a small glyph[epsilon1] , under the assumption that f ( j ) ( x glyph[diamondmath] ξ i ) = f ( j ) ( x glyph[diamondmath] ξ k ) for i = k , we should get that m ( j ) ( x + glyph[epsilon1] + ξ ) = f ( j ) ( x + glyph[epsilon1] + ξ i ( j ) ) with the same i ( j ) as before the perturbation. This, in essence, says that we can calculate the gradients of the elements of the MVAR set as ∇ x m ( j ) ( x glyph[diamondmath] ξ ) = ∇ x f ( j ) ( x glyph[diamondmath] ξ i ( j ) ) . Putting all outcomes together, we get

glyph[negationslash]

∇ x m ( x glyph[diamondmath] ξ ) = [ ∇ x f (1) ( x glyph[diamondmath] ξ i (1) ) , . . . , ∇ x f ( M ) ( x glyph[diamondmath] ξ i ( M ) )] .

A theoretical consistency analysis of these MVAR gradient estimators is beyond the scope of this paper. However, we observe that they do work well in practice, enabling efficient optimization of MVAR-NEHVI-RFF (see Appendix D&I).

## D. Direct MVAR Optimization using q NEHVI

In this section, we discuss direct optimization of MVAR using NEHVI, highlight the computational challenges that come with this approach, and introduce an approximation that mitigates some of these challenges for some problems where the MVAR set for a design is relatively small (e.g. where the number of objectives is small and α is large). However, we find that these approaches are typically infeasible when M ≥ 3 due to GPU memory limits (see for example Table 4).

## D.1. Direct Optimization of MVAR with NEHVI

As described in Section 5, the extension of q NEHVI to optimize MVAR is conceptually simple. q NEHVI selects the next point to evaluate by maximizing the expected HVI under the GP posterior. We replace the standard HVI of a new point with respect to the PF with the joint HVI of the MVAR set of a new point with respect to the MVAR set over the previously evaluated designs:

α MVAR-NEHVI ( x ) = E f ∼ P ( f |D ) [ HVI ( MVAR α [ f ( x glyph[diamondmath] ξ )] | MVAR α [ { f ( x ' glyph[diamondmath] ξ ) } x ' ∈ X 1: n ] ] .

However, there are several computational issues that make this approach prohibitively expensive, except for when the objective evaluation takes multiple hours or days. In our experiments, we observe long runtimes, even when using very reasonable parameter values of n ξ = 32 and α = 0 . 9 . We discuss the factors contributing to this in the following subsection. Note that all the issues discussed are compounded by the fact that even when using a gradient-based approach to optimize the acquisition function, the acquisition function needs to be evaluated many times.

## D.2. Complexity and Challenges

There are three primary computational bottlenecks, corresponding to three stages of computing α MVAR-NEHVI ( x ) . We discuss each stage and their complexity below.

Figure 5: The maximum size of MVAR α [ f ( x glyph[diamondmath] ξ )] across the design space x ∈ X , which is an estimator of the maximum MVAR set size that will encountered during numerical optimization, for different α with n ξ = 32 on the GMM problem. The size of the MVAR set significantly increases as α decreases and as M increases.

picture-5.png

- 1. Posterior Sampling : Computing α MVAR-NEHVI ( x ) requires drawing joint posterior samples at the baseline points (points that are already evaluated) and the current candidate(s) x under all n ξ perturbations. For all methods using GPs, posterior sampling at n points and n ξ perturbations scales as O ( n 3 ξ n 3 M ) . Hence, using GPs is only feasible for modest n ξ .
- 2. Computing MVAR : The next stage is computing the MVAR corresponding to each x from posterior samples of f ( x glyph[diamondmath] ξ ) . Computing MVAR involves computing the distribution function of f ( x glyph[diamondmath] ξ ) , 11 which has a time and space complexity of O ((1 -α ) M n M +1 ξ M ) . The MVAR has to be computed for each x and each posterior sample, further

inflating the computational effort required. The size of the resulting MVAR set is O ((1 -α ) M n M ξ M ) . Figure 5 empirically demonstrates that the size of the MVAR set significantly increases as α decreases, particularly for larger M .

- 3. Computing joint hypervolume improvement : Given the samples of MVAR corresponding to the baseline points and the candidate(s), the final step of MVAR-NEHVI is to compute the joint HVI of the MVAR set of the candidates over the global MVAR set corresponding to the baseline points. To our knowledge, the only existing differentiable approach for joint HVI computation relies on the inclusion-exclusion principle (IEP, Daulton et al. (2020)). The time and space complexity of computing the joint HVI of a set of q ' points using the IEP is exponential with respect to q ' . To compute the HVI of the MVAR for a set of q candidates, we must compute the joint HVI of a set of size q ' = q | MVAR α [ { f ( x i + ξ ) } q i =1 )] | . Since the size of the MVAR set of a single candidate scales as O ((1 -α ) M n M ξ M ) , using IEP quickly becomes infeasible, except for very moderate M,n ξ , and α . As shown in Figure 5, even for q = 1 , n ξ = 32 , and M = 3 , the size of the MVAR set can be quite large for smaller values of α , which precludes the use of IEP. Although IEP is necessary to make the joint hypervolume improvement computation differentiable, there are non-differentiable approaches (e.g. Lacour et al. (2017)) that could be used instead to compute the joint hypervolume improvement. However, optimizing α MVAR-NEHVI without gradients would be very slow given that Daulton et al. (2020) showed that simply optimizing analytic EHVI (without MVAR) with CMA-ES (Hansen, 2007) or L-BFGS-B (Byrd et al., 1995) with approximate gradients estimated via finite differences is over an order of magnitude slower than when using exact gradients.

A final challenge in using MVAR-NEHVI is that the calculation of MVAR is not differentiable, and there are no known theoretically-grounded gradient estimators of MVAR. Therefore, we use the heuristic approach described in Appendix C.2 for estimating the gradients of MVAR.

## D.3. Approximating qNEHVI with RFF Draws

Random Fourier Features (RFF, Rahimi and Recht (2008)) offer an inexpensive and differentiable approximation of GP sample paths. Daulton et al. (2021a) propose to combine a single RFF draw with NEHVI to obtain a cheap approximation, q NEHVI-1. They find that q NEHVI-1 is competitive with qNEHVI in small dimensional search spaces, though its performance degrades as the dimensionality increases.

We follow their approach and extend q NEHVI-1 to optimize MVAR, and name this method MVAR-NEHVI-RFF. Using RFFs significantly reduces the computational cost of the evaluating the acquisition function because it avoids computationally expensive exact posterior sampling. In addition, since we use a single RFF draw, the MVAR set and its HVI only have to be computed once per acquisition function evaluation rather than for each posterior sample. In the end, for small problem instances, MVAR-NEHVI-RFF ends up with a per-iteration runtime measured in seconds, which is an immense reduction from the time it takes for MVAR-NEHVI using an exact GP model.

Note that MVAR-NEHVI-RFF still requires the use of IEP for differentiable HVI computations, making this approach infeasible in many settings as discussed above. In addition, the performance of RFFs based acquisition functions are known to degrade when the underlying function is difficult to model and variance starvation is a known issue (Wang et al., 2018; Wilson et al., 2020; Mutny and Krause, 2018; Calandriello et al., 2019). Thus, for an acquisition function that works well in all settings, we recommend using MARS.

## E. Pruning for Efficient Joint Posterior Sampling

NEI and q NEHVI both require sampling from the joint posterior over function values at the new design x and previously evaluated designs X 1: n : P ( { f ( x ) } x ∈ X 1: n ∪{ x 1 ,..., x q } |D ) . To reduce the cost of posterior sampling, we prune X 1: n to only include the subset of points X pruned ⊆ X 1: n that have nonzero probability of being optimal. We estimate the probability of being optimal using MC estimation with N prune samples from the joint posterior. For NEI, optimality is with respect to a scalar objective and often means | X pruned | << n . For q NEHVI, any design that has nonzero probability of being Pareto optimal is retained; typically, this results in a much larger X pruned than we using NEI with a scalar objective. The typically larger size of X pruned under q NEHVI-based methods has a significant effect when using MVARq NEHVI, where sampling from the joint posterior scales as O ( Mnn ξ ) and has a very significant effect on runtime. Pruning strategies that leverage techniques from pre-screening and population selection in EAs may further improve computational efficiency.

## F. Optimization of Multi-Objective Expectation Objectives

As noted in the main text, the optimization of expectation of objectives can be achieved via rather straightforward extensions of the existing multi-objective acquisition functions. Here, we discuss the main idea, and show how to extend q NPAREGO and q NEHVI (Daulton et al., 2021a).

## F.1. Optimization Expectation Objectives with q NPAREGO

The acquisition function PAREGO is an extension of the well known Expected Improvement acquisition function to the multi-objective setting via augmented Chebyshev scalarizations. q NPAREGO is an MC-based variant that uses composite objectives with the NEI acquisition function (Daulton et al., 2021a). Given a weight vector w ∈ ∆ M -1 + , it selects the next point to evaluate as follows:

x n +1 = arg max x ∈X E f ∼ P ( f |D ) [ s a [ f ( x ) , w ] -max x ' ∈ X 1: n s a [ f ( x ' ) , w ] ] + , (3)

where X 1: n denotes the points evaluated so far, and [ · ] + denotes max( · , 0) , s a [ y , w ] = min w i y i + β ∑ i w i y i , and β is a small positive constant. The expectation in (3) is not available in closed form, and is typically replaced by a (Q)MC approximation obtained by drawing samples of { f ( x ' ) } x ' ∈ X 1: n ∪{ x } from the joint GP posterior. A batch of q candidates can be selected in a sequential greedy fashion where each point is selected using a different scalarization weight vector and the improvement from the batch of q points replaces the improvement from a single point in (3).

To extend qNPAREGO to the expectation objectives, we replace each occurrence of s a [ f ( x ) , w ] in (3) with s a [ E ξ ∼ P ( ξ ) [ f ( x glyph[diamondmath] ξ )] , w ] . For implementation, we follow the same MC idea, and draw samples from the joint posterior of { f ( x glyph[diamondmath] ξ ) } x ∈ X 1: n ∪{ x 1 ,..., x q } , ξ ∈ Ξ where Ξ is a set of n ξ input noise samples, and approximate (3) using these samples. To improve the computational efficiency, one can also calculate the posterior distribution of { E ξ ∼ P ( ξ ) [ f ( x glyph[diamondmath] ξ )] } x ∈ X 1: n ∪{ x 1 ,..., x q } from the posterior of [ f ( x glyph[diamondmath] ξ )] x ∈ X 1: n ∪{ x 1 ,..., x q } , ξ ∈ Ξ via a simple matrix-matrix product, and use that to draw the posterior samples. This avoids inverting m ( n + q ) n ξ × ( n + q ) | Ξ | matrices, and reduces the cost of posterior sampling from O ( m ( n + q ) 3 n 3 ξ ) to O ( m ( n + q ) 3 ) . However, this computational technique cannot be used if there are black-box constraints. We refer to this method as EXPq NPAREGO.

## F.2. Optimization Expectation Objectives with q NEHVI

The other acquisition function we consider is the q NEHVI:

x n +1 = arg max x ∈X E f ∼ P ( f |D ) [ HVI ( f ( x ) |P n )] , (4)

where P n is the PF over f ( x ) x ' ∈ X 1: n . The extension of qNEHVI to the expectation objectives follows a similar path to that of q NPAREGO. We replace the hypervolume improvement of f ( x ) in (4) with the hypervolume improvement of E ξ ∼ P ( ξ ) [ f ( x )] with respect to the PF over { E ξ ∼ P ( ξ ) [ f ( x ' )] } x ' ∈ X 1: n . To do so, we replace the posterior samples of f ( x ) and { f ( x ' )] } x ' ∈ X 1: n that are used in q NEHVI calculations with the posterior samples of the expectation over P ( ξ ) , which can be obtained in the same manner described above for q NPAREGO. However, q NEHVI with expectation objective is often prohibitively slow because typically relatively few points from X 1: n can be pruned as discussed in Appendix E. Hence, we only evaluate a single sample approximation using RFFs, analogous to the RFF approximation of MVAR-NEHVI, which we refer to as EXP-NEHVI-RFF.

## F.3. Challenges of Using Expectation with Feasibility-Weighted Objectives

Independently computing the expectation of the objectives and the feasibility and taking the product of the expectations, would ignore the fact that the objective functions and constraint functions are evaluated on the same perturbed designs. To account for the perturbed inputs jointly across in the objectives and constraints, we use feasibility weighted objectives. Feasibility weighting requires penalizing designs that are infeasible such that the feasibility-weighted objectives for an infeasible design are worse than the objectives for any feasible design.

Feasibility weighting can make the expectation sensitive to the range of the objectives. When evaluating a solution near the border of the feasible domain, we end up with a subset of the perturbed solutions evaluating to zero due to infeasibility and others evaluating to their respective objective values. To see how this can affect the performance, consider the following

examples. Suppose that half of the perturbed solutions are infeasible and the objective values are bounded in [0 , 1] . In this case, the feasibility weighted objective take values in [0 , 0 . 5] , where it will be inferior to some other solutions due to the potential for it to be infeasible. Now, suppose that the objective values are bounded in [100 , 101] . The infeasibility in this case will bring the feasibility weighted objective to the range of [50 , 50 . 5] , which is strictly worse than any solution that is more feasible, even if by only a small fraction. If we instead set the infeasible solutions to 100 rather than zero, this would lead to the feasibility weighted objective value to [100 , 100 . 5] . Note that setting the infeasible solutions to 100 is equivalent to normalizing the objectives to [0 , 1] before applying the feasibility and using zero for the infeasible objectives, which in theory should have no effect in the optimization performance. In practice, we typically do not know precise bounds on the objectives, and instead standardize / normalize the objectives during optimization using bounds derived on the go.

The example above highlights the effect of the range of each objective. Often, an infeasibility cost λ is used to penalize for infeasible designs to ensure that infeasible points are worse than any feasible point (for example if the objectives can take negative values) by setting the feasibility-weighted objectives to ( f ( x ) + λ ) ✶ [ c > 0 ] -λ for some λ ≥ 0 . However, the feasibility weighted expectation is typically sensitive to the infeasibility cost, and feasibility weighted objectives give higher value to conservative solutions if the Pareto front lies near the border of the feasible domain. This makes it difficult to determine apriori how conservatively expectation methods will act when using feasibility weighted objectives. In contrast, MVAR avoids this issue by providing high probability guarantees on the value of the feasibility-weighted objectives under input noise. For any that design is feasible with probability α , the infeasibility cost is in the tail of the multivariate CDF and has no effect on the elements of the MVAR set.

## G. Experiment Details

## G.1. Method Details

We evaluate the following BO methods:

Methods that optimize the nominal objectives (see Daulton et al. (2021a)) for details): q NPAREGO, q NEHVI, and NEHVI-RFF (referred to as q NEHVI-1 in Daulton et al. (2021a)), which approximates the expectation in q NEHVI with a single approximate GP sample using RFFs.

Methods that optimize the expectation objectives (Appendix F): EXPq NPAREGO, and EXP-NEHVI-RFF.

Methods that optimize MVAR : MARS-NEI (Section 5.2), MARS-TS (Appendix B), MARS-UCB (Appendix B), MVAR-NEHVI (Appendix D), and MVAR-NEHVI-RFF (Appendix D).

We implemented all methods using the BoTorch library (Balandat et al., 2020) (except for NSGA-II), leveraging the existing implementations of NEI and q NEHVI available at https://github.com/pytorch/botorch . We used the implementation of NSGA-II in the PyMOO library (Blank and Deb, 2020), which is available at https://github. com/anyoptimization/pymoo .

For all model-based methods, we model each objective and constraint with an independent GP with a Mat'ern5 2 ARD kernel (Rasmussen, 2004). 12 For methods that use scalarizations, we use composite objectives (Astudillo and Frazier, 2019). We use maximum a posteriori estimates of the GP hyperparameters using the default priors in BoTorch. For all MC-based acquisition functions, we use N MC = 256 QMCsamples from the GP posterior. We use sample-average approximation (Balandat et al., 2020) by using fixed Quasi-MC samples from P ( ξ ) (for robust methods) and fixed Quasi-MC base samples for all methods to approximate the expectation over the GP posterior. 13,14 This results in an approximation of the acquisition function that is a deterministic function of the input x . For RFF-based methods, the approximate GP sample (using 512 random features) is also a deterministic function, which, coupled with fixed samples from P ( ξ ) , results in a deterministic approximation of the acquisition functions. The deterministic approximations of the acquisition functions enable the use of quasi-Newton methods for optimization. We optimize all acquisition functions using multi-start optimization with L-BFGS-B (Zhu et al., 1997).

For MARS and other MVAR-based methods, we use the known MVAR reference point, which would typically be supplied

by the decision maker. For q NEHVI and EXP-NEHVI-RFF, we use the heuristic from Daulton et al. (2020) to adaptively infer the the reference point during the optimization (the MVAR reference point is not suitable for the nominal and expectation objectives).

For methods involving scalarizations, the objectives are normalized before applying the scalarizations. For MARS methods, the reference point is used as the lower bound and the ideal point (i.e. the component-wise maximum of each objective, Ishibuchi et al. (2018)) across the MVAR set over the previously evaluated designs (estimated using the posterior mean) is used as the upper bound for normalization. For q NPAREGO, we use the ideal and nadir points (i.e. the component-wise minimum objective values across the PF, Ishibuchi et al. (2018)) across the PF over the previously evaluated designs. Similarly, for EXPq NPAREGO, we use the ideal and nadir points over the PF expectation objectives (estimated using the posterior mean) over the previously evaluated designs.

For feasibility weighting the objectives, we use a sigmoid function as a differentiable approximation of the indicator function as in Balandat et al. (2020). The infeasibility cost set to be the minimum posterior mean minus six standard deviations.

For methods that use NEI and q NEHVI with exact posterior sampling, we prune the previously evaluated designs using N prune = 2048 samples to estimate the probability that a previously evaluated design is optimal. Additionally, we cache the Cholesky decomposition of the posterior covariance matrix over { f ( x ' glyph[diamondmath] ξ ) } x ' ∈ X pruned and use low-rank updates to draw joint samples over { f ( x ' glyph[diamondmath] ξ ) } x ' ∈ X pruned ∪{ x } (Osborne, 2010).

For batch (or asynchronous) candidate generation, we use a sequential greedy approach (Wilson et al., 2018), where one new candidate is optimized at time and the joint acquisition value of all candidates x 1 , ..., x i is optimize to select x i . For methods relying on scalarizations, a new scalarization is sampled for each new candidate.

For NSGA-II, we used the same initial sobol starting points as for the other methods. We used a population size of 10 and adjusted the number of iterations for NSGA-II according to the evaluation budget. The PyMOO default configuration was used for all other settings. In addition to the objectives, observations of the constraints are provided to NSGA-II.

## G.2. Problem Details

In this section, we provide descriptions of the test problems. The reference points used for all problems are provided in Table 2. For each problem, we set the reference point to be slightly worse than the nadir point (using the heuristic from Ishibuchi et al. (2011)) of the MVAR set evaluated over a large grid of design points. For the Penicillin problem, we set the reference point to exclude the region of the objective space with low values of the time objective (following (Liang and Lai, 2021)) as those objective trade-offs are less appealing to decision makers due to providing negligible Penicillin yield.

Toy Problem ( d = 1 , M = 2 , α = 0 . 9 ): This is the toy problem that was used to highlight the concepts in Figures 1&2. The noise model is given by P ( ξ ) = N ( µ = 0 , Σ = 0 . 1 I 2 ) . The first objective is a mixture linear-sinusoidal function, and the second objective is modified from the well-known Levy test function. The exact expressions are given as follows. The function is evaluated on x ∈ [0 , 0 . 7] .

```
f (1) ( x ) = 30 -30 ∗ ( p 1 ( x ) p 4 ( x ) + p 2 ( x )(1 -p 4 ( x )) + p 3 ( x )) p 1 ( x ) = 2 . 4 -10 x -0 . 1 x 2 p 2 ( x ) = 2 x -0 . 1 x 2 p 3 ( x ) = ( x -0 . 5) 2 +0 . 1 sin(30 x ) p 4 ( x ) = 1 / (1 + exp (( x -0 . 2) / 0 . 005)) f (2) ( x ) = p 5 (( x ∗ 0 . 95 + 0 . 03) ∗ 20 -10) p 5 ( x ) = p 6 (1 + ( x -1) / 4) -0 . 75 ∗ x 2 +9 . 0955 p 6 ( x ) = (sin( π ∗ x )) 2 +( x -1) 2 (1 + 10 (sin( π ∗ x )) 2 )
```

GMM ( d = 2 , M ∈ { 2 , 3 , 4 } , α ∈ { 0 . 7 , 0 . 8 , 0 . 9 } ): In addition to the version presented in the main text, we consider several variations of the GMM problem using different number of objectives, different noise models, and different risk levels to analyze the effects of these factors on the optimization performance of the algorithms. For all GMM problems considered, each objective is a mixture of the probability density function of three Gaussian distributions, modified from the single objective version presented in (Frohlich et al., 2020). We present the canonical formula of the objectives and

the parameters corresponding to each objective below. In the formula, φ ( x ; µ, Σ) is used to denote the probability density function of the multivariate Gaussian distribution with mean µ and covariance matrix Σ . The search space for all GMM problems is X = [0 , 1] 2 .

The GMM problem used in the main text involves 2 objectives and uses α = 0 . 9 . Additional experiments in Appendix I.1 use additional independent GMMs to increase the number of objectives to 3 and 4 , and evaluate performance with different settings of α ∈ { 0 . 7 , 0 . 8 , 0 . 9 } . In all experiments with 3 and 4 objective GMM, we use additive noise, where P ( ξ ) = N ( µ = 0 , Σ = 0 . 05 I M ) . Many additional noise processes are discussed and evaluated in Appendix I.5, using the same 2 objective GMM problem from the main text and α = 0 . 9 .

f ( i ) ( x ) = 2 π 3 ∑ j =1 var ( i ) j cons ( i ) j φ ( x ; µ = pos ( i ) j , Σ = var ( i ) j I 2 ) pos ( i ) j =            j = 1 j = 2 j = 3 [0 . 2 , 0 . 2] [0 . 8 , 0 . 2] [0 . 5 , 0 . 7] if i = 1 [0 . 07 , 0 . 2] [0 . 4 , 0 . 8] [0 . 85 , 0 . 1] if i = 2 [0 . 08 , 0 . 21] [0 . 45 , 0 . 75] [0 . 86 , 0 . 1] if i = 3 [0 . 09 , 0 . 19] [0 . 44 , 0 . 72] [0 . 89 , 0 . 13] if i = 4 var ( i ) j =            j = 1 j = 2 j = 3 0 . 04 0 . 01 0 . 01 if i = 1 0 . 04 0 . 01 0 . 0025 if i = 2 0 . 04 0 . 01 0 . 0049 if i = 3 0 . 0225 0 . 0049 0 . 0081 if i = 4 cons ( i ) j =            j = 1 j = 2 j = 3 0 . 5 0 . 7 0 . 7 if i = 1 0 . 5 0 . 7 0 . 7 if i = 2 0 . 5 0 . 7 0 . 9 if i = 3 0 . 5 0 . 7 0 . 9 if i = 4

Constrained Branin Currin We use the open source implementation available at https://github.com/pytorch/ botorch . See Daulton et al. (2020) for details.

Disc Brake We use the open source implementation available at https://github.com/ryojitanabe/ reproblems . See Tanabe and Ishibuchi (2020) for details.

Penicillin Manufacturing Problem We use the open-source implementation available at https://github.com/ HarryQL/TuRBO-Penicillin . See Liang and Lai (2021) for details. We adapt the problem by adding independent zero-mean Gaussian input noise to each parameter. The standard deviation of the input noise distribution for each parameter is listed in Table 1.

Table 1: Standard deviation for independent zero-mean Gaussian input noise for each parameter in the Penicillin Problem (reported as a percentage of the range of each parameter).

| Parameter                    | Noise Level   |
|------------------------------|---------------|
| Culture Volume               | 3%            |
| Biomass Concentration        | 3%            |
| Temperature                  | 0.5%          |
| Glucose Concentration        | 2%            |
| Substrate Feed Rate          | 1%            |
| Substrate Feed Concentration | 1%            |
| H + Concentration            | 1%            |

Table 2: Reference points for negative versions (i.e. multiplying the objectives by -1 to make the goal maximization of all objectives) of all problems (except the GMM and Toy problems, which are designed for maximization).

| Problem                                                    | Reference Point                    |
|------------------------------------------------------------|------------------------------------|
| Toy Problem                                                | [-14.1951, -3.1887]                |
| Disc Brake                                                 | [-5.89, -3.27]                     |
| Constrained Branin Currin (heteroskedastic noise)          | [-194.9376, -12.2969]              |
| Constrained Branin Currin (homoskedastic noise)            | [-195.4667, -12.4984]              |
| Penicillin                                                 | [5.657, -64.1, -340.0]             |
| GMM( M = 2 , α = 0 . 9 , multiplicative noise)             | [0.3752, 0.3548]                   |
| GMM( M = 2 , α = 0 . 9 , correlated noise)                 | [0.2727, 0.2583]                   |
| GMM( M = 2 , α = 0 . 8 , heteroskedastic noise)            | [0.3465, 0.3036]                   |
| GMM( M = 2 , α = 0 . 9 , homoskedastic noise, σ = 0 . 05 ) | [0.2756, 0.2368]                   |
| GMM( M = 2 , α = 0 . 9 , homoskedastic noise, σ = 0 . 1 )  | [0.1047, 0.1112]                   |
| GMM( M = 2 , α = 0 . 9 , homoskedastic noise, σ = 0 . 2 )  | [0.0160, 0.0131]                   |
| GMM( M = 3 , α = 0 . 9 , homoskedastic noise, σ = 0 . 05 ) | [0.2733, 0.0051, 0.1538]           |
| GMM( M = 3 , α = 0 . 9 , homoskedastic noise, σ = 0 . 05 ) | [0.2733, 0.0051, 0.1538]           |
| GMM( M = 3 , α = 0 . 8 , homoskedastic noise, σ = 0 . 05 ) | [0.0420, 0.0180, 0.1952]           |
| GMM( M = 3 , α = 0 . 7 , homoskedastic noise, σ = 0 . 05 ) | [0.0537, -0.0517, -0.0021]         |
| GMM( M = 4 , α = 0 . 9 , homoskedastic noise, σ = 0 . 05 ) | [0.0264, -0.0396, 0.0619, 0.1689]  |
| GMM( M = 4 , α = 0 . 8 , homoskedastic noise, σ = 0 . 05 ) | [0.0322, -0.0398, 0.1168, -0.0023] |

## G.3. Evaluation Details

The global MVAR set is unknown and is approximated by taking the union of the MVAR sets of all designs evaluated across all methods and all replications. We take this approach because even using an evolutionary algorithm to optimize MVAR is nontrivial, since MVAR maps a single design to a set of points and is relatively computationally intensive to evaluate. To evaluate the performance of a given method, we use n ξ = 512 (except for 4 objective GMM, where we use n ξ = 256 ) to compute a high-fidelity estimate of the MVAR set across the designs selected during optimization by the method. We similarly use the same n ξ = 512 samples to estimate the true MVAR set (by considering all designs evaluated across all methods and all replications).

## H. Wall Times

In Table 3, we present the time it takes to run a single BO iteration using all algorithms we considered in this paper. We include the runtimes for the four problems from the main text. Wall times for additional problems including several problems with 3 and 4 objectives are provided in Table 4 (these problems are described in Appendix I.1). As we discuss in Appendix D, MVAR-NEHVI, when not computationally infeasible, is quite expensive to run, making it an impractical method for most problems. Although MVAR-NEHVI-RFF provides a much cheaper and highly performant approximation, we see that it also runs into computational limitations as the size of the MVAR set grows (e.g. for Penicillin with 3 objectives); this is also pronounced in Table 4 on the problems with 3 and 4 objectives. For the MARS family of methods, we see overall quite reasonable runtimes, with the most expensive one, MARS-NEI, taking on average 41 . 4 seconds on the most expensive problem instance we considered. MARS-TS offers a cheaper alternative to MARS-NEI, with its average runtime remaining below 10 seconds on all experiments. As we show later in the Appendix, the performance of MARS-TS typically trails closely behind MARS-NEI, making it a strong alternative when the algorithm runtime is of the essence.

Table 3: The wall time (in seconds) per BO iteration. The experiments were timed on a shared cluster using 4 CPU cores, 1 GPU, and 16 GB of RAM. We report the mean and 2 standard errors over 20 trials. An N/A entry denotes that we did not attempt to run a particular experiment (e.g., because the method does not support the problem setting), whereas an OOM entry denotes that we attempted but the experiment did not run due to scalability limitations. The three top-performing algorithms, with respect to the final average MVAR HV regret in each experiment are highlighted using best , second , third , respectively.

| Algorithm ( d, M, V, α )   | GMM ( 2 , 2 , 0 , 0 . 9 )   | Constrained BC ( 2 , 2 , 1 , 0 . 7 )   | Disc Brake ( 4 , 2 , 4 , 0 . 95 )   | Penicillin ( 7 , 3 , 0 , 0 . 8 )   |
|----------------------------|-----------------------------|----------------------------------------|-------------------------------------|------------------------------------|
| Sobol                      | 0 . 4 ( ± 0 . 6)            | 0 . 9 ( ± 1 . 0)                       | 0 . 9 ( ± 1 . 0)                    | 2 . 9 ( ± 1 . 5)                   |
| q NPAREGO                  | 2 . 5 ( ± 2 . 3)            | 5 . 6 ( ± 4 . 4)                       | 7 . 9 ( ± 15 . 1)                   | 21 . 2 ( ± 25 . 6)                 |
| q NEHVI                    | 2 . 5 ( ± 2 . 0)            | 9 . 8 ( ± 8 . 9)                       | 16 . 9 ( ± 10 . 3)                  | 23 . 6 ( ± 41 . 0)                 |
| q NEHVI-RFF                | 0 . 8 ( ± 0 . 7)            | 2 . 1 ( ± 3 . 2)                       | 2 . 7 ( ± 5 . 4)                    | 5 . 0 ( ± 3 . 9)                   |
| Exp- q NPAREGO             | 3 . 3 ( ± 2 . 4)            | 10 . 7 ( ± 11 . 1)                     | 23 . 3 ( ± 166 . 3)                 | 121 . 9 ( ± 119 . 0)               |
| EXP-NEHVI-RFF              | 0 . 9 ( ± 1 . 0)            | 7 . 2 ( ± 10 . 8)                      | 3 . 3 ( ± 7 . 3)                    | 5 . 1 ( ± 3 . 3)                   |
| MARS-NEI                   | 3 . 9 ( ± 3 . 0)            | 8 . 4 ( ± 5 . 3)                       | 10 . 3 ( ± 45 . 4)                  | 41 . 4 ( ± 60 . 1)                 |
| MARS-TS                    | 3 . 3 ( ± 3 . 6)            | 3 . 3 ( ± 3 . 7)                       | 3 . 1 ( ± 6 . 5)                    | 6 . 7 ( ± 5 . 8)                   |
| MARS-UCB                   | 8 . 2 ( ± 11 . 0)           | N/A                                    | N/A                                 | 12 . 3 ( ± 14 . 1)                 |
| MVAR-NEHVI                 | 145 . 6 ( ± 130 . 7)        | N/A                                    | 243 . 2 ( ± 154 . )                 | N/A                                |
| MVAR-NEHVI-RFF             | 1 . 8 ( ± 1 . 7)            | 10 . 0 ( ± 11 . 9)                     | 2 . 9 ( ± 3 . 1)                    | OOM                                |

## I. Additional Experiments

## I.1. Additional Test Problems

In addition to the problems presented in the main text, we studied the performance of the algorithms on the Toy problem used for illustrations in Figures 1 and 2, and the 3 and 4 objective variations of the GMM problem. These problems are described in detail in Appendix G.2. In addition to the acquisition functions presented in the main text, we ran EXP-NEHVI-RFF, MVAR-NEHVI-RFF, MARS-UCB, and MARS-TS. The results of these experiments are presented in Figure 6 for the Toy problem and Figure 7 for the GMM problems, and the runtimes of the algorithms are reported in Table 4. We see that MARS-NEI is overall the best performing method, with MARS-TS typically following closely. MARS-UCB appears to be less reliable, demonstrating significantly worse performance in most experiments. In addition, the MVAR-NEHVI-RFF is missing from all but two of the experiments, which is due to the method running into the scalability limitations discussed in Appendix D.

1D Toy Problem

Figure 6: The log MVAR hypervolume regret on the Toy problem. We plot means and 2 standard errors across 20 trials.

picture-6.png

Log MVAR HV Regret

GMM, M=3,  =0.7

GMM, M=4,  =0.8

Figure 7: The log MVAR hypervolume regret on 3 and 4-Objective GMM problems. We plot means and 2 standard errors across 20 trials.

picture-7.png

Table 4: The wall time (in seconds) per BO iteration for the additional problems. The experiments were timed on a shared cluster using 4 CPU cores, 1 GPU, and 16 GB of RAM. We report the mean and 2 standard errors over 20 trials. An OOM entry denotes that the experiment did not run due to scalability limitations. The three top-performing algorithms, with respect to the final average MVAR HV regret in each experiment are highlighted using best , second , third , respectively.

| Algorithm      | Toy Problem        |                    | GMM, M=3 ( 2 , 3 , 0 . 8 )   |                    | GMM, M=4            | GMM, M=4              |
|----------------|--------------------|--------------------|------------------------------|--------------------|---------------------|-----------------------|
| ( d, M, α )    | ( 1 , 2 , 0 . 9 )  | ( 2 , 3 , 0 . 7 )  |                              | ( 2 , 3 , 0 . 9 )  | ( 2 , 4 , 0 . 8 )   | ( 2 , 4 , 0 . 9 )     |
| Sobol          | 0 . 4 ( ± 0 . 3)   | 1 . 9 ( ± 2 . 2)   | 1 . 2 ( ± 1 . 4)             | 0 . 5 ( ± 0 . 3)   | 3 . 3 ( ± 3 . 3)    | 0 . 8 ( ± 0 . 2)      |
| q NPAREGO      | 2 . 2 ( ± 2 . 0)   | 4 . 5 ( ± 2 . 6)   | 3 . 4 ( ± 2 . 6)             | 1 . 4 ( ± 1 . 5)   | 8 . 9 ( ± 10 . 2)   | 4 . 5 ( ± 12 . 9)     |
| q NEHVI        | 3 . 0 ( ± 2 . 5)   | 23 . 3 ( ± 26 . 4) | 20 . 2 ( ± 22 . 8)           | 23 . 8 ( ± 43 . 8) | 57 . 3 ( ± 75 . 4)  | 48 . 0 ( ± 55 . 2)    |
| q NEHVI-RFF    | 0 . 4 ( ± 0 . 3)   | 3 . 1 ( ± 5 . 8)   | 1 . 7 ( ± 1 . 0)             | 0 . 7 ( ± 0 . 5)   | 10 . 0 ( ± 91 . 8)  | 2 . 1 ( ± 4 . 0)      |
| Exp- q NPAREGO | 13 . 1 ( ± 23 . 5) | 8 . 0 ( ± 12 . 8)  | 6 . 2 ( ± 6 . 4)             | 13 . 4 ( ± 54 . 2) | 10 . 6 ( ± 10 . 6)  | 8 . 1 ( ± 18 . 3)     |
| EXP-NEHVI-RFF  | 0 . 4 ( ± 0 . 3)   | 4 . 0 ( ± 10 . 3)  | 2 . 1 ( ± 6 . 3)             | 0 . 8 ( ± 0 . 8)   | 5 . 3 ( ± 4 . 0)    | 1 . 9 ( ± 1 . 5)      |
| MARS-NEI       | 10 . 4 ( ± 21 .    | 1) 11 . 1 ( ± 14 . | 4) 10 . 7 ( ± 23 . 0) 8      | . 8 ( ± 13 .       | 2) 27 . 5 ( ± 221 . | 1) 13 . 5 ( ± 15 . 9) |
| MARS-TS        | 0 . 9 ( ± 0 . 7)   | 5 . 7 ( ± 8 . 9)   | 3 . 8 ( ± 2 . 9)             | 1 . 8 ( ± 2 . 5)   | 9 . 9 ( ± 27 . 1)   | 4 . 2 ( ± 4 . 2)      |
| MARS-UCB       | 6 . 7 ( ± 7 . 4)   | 14 . 4 ( ± 22 . 5) | 10 . 4 ( ± 11 . 1)           | 10 . 3 ( ± 12 . 3) | 17 . 6 ( ± 19 . 1)  | 12 . 1 ( ± 11 . 9)    |
| MVAR-NEHVI-RFF | 2 . 6 ( ± 2 . 2)   | OOM                | OOM                          | 3 . 0 ( ± 2 . 1)   | OOM                 | OOM                   |

## I.2. Comparison with q NEHVI Based Methods

Table 5: The final MVAR HV regret obtained using each method. We report the mean and 2 standard errors over 20 trials. An N/A entry denotes that we did not attempt to run a particular experiment (e.g., because the method does not support the problem setting), whereas an OOM entry denotes that we attempted but the experiment did not run due to scalability limitations. The three top-performing algorithms, with respect to the final average MVAR HV regret in each experiment are highlighted using best , second , third , respectively.

| Algorithm d, M, V, α   | GMM                              | Constrained BC                 | Disc Brake                      | Penicillin ( 7 , 3 , 0 , 0 . 8 ) 1 × 10 4   | Toy Problem ( 1 , 2 , 0 , 0 . 9 ) 1 × 10 0   | GMM, M=4 ( 2 , 4 , 0 , 0 . 8 ) 1 × 10 - 3   |
|------------------------|----------------------------------|--------------------------------|---------------------------------|---------------------------------------------|----------------------------------------------|---------------------------------------------|
| ( ) Scale              | ( 2 , 2 , 0 , 0 . 9 ) 1 × 10 - 4 | ( 2 , 2 , 1 , 0 . 7 ) 1 × 10 1 | ( 4 , 2 , 4 , 0 . 95 ) 1 × 10 0 |                                             |                                              |                                             |
| Sobol                  | 65 . 37 ( ± 10 . 23)             | 3 . 12 ( ± 0 . 36)             | 19 . 39 ( ± 1 . 35)             | 1 . 68 ( ± 0 . 05)                          | 2 . 38 ( ± 0 . 27)                           | 7 . 39 ( ± 1 . 35)                          |
| q NPAREGO              | 21 . 54 ( ± 7 . 68)              | 3 . 28 ( ± 0 . 41)             | 19 . 10 ( ± 1 . 83)             | 1 . 73 ( ± 0 . 09)                          | 4 . 69 ( ± 0 . 78)                           | 3 . 04 ( ± 1 . 24)                          |
| q NEHVI                | 31 . 21 ( ± 14 . 34)             | 5 . 16 ( ± 0 . 76)             | 12 . 33 ( ± 1 . 77)             | 1 . 65 ( ± 0 . 12)                          | 8 . 95 ( ± 1 . 87)                           | 3 . 66 ( ± 1 . 47)                          |
| q NEHVI-RFF            | 30 . 54 ( ± 12 . 13)             | 5 . 60 ( ± 0 . 81)             | 17 . 11 ( ± 1 . 97)             | 1 . 38 ( ± 0 . 09)                          | 5 . 97 ( ± 0 . 77)                           | 2 . 18 ( ± 0 . 62)                          |
| Exp- q NPAREGO         | 10 . 29 ( ± 2 . 92)              | 2 . 73 ( ± 0 . 77)             | 3 . 43 ( ± 0 . 57)              | 1 . 52 ( ± 0 . 10)                          | 8 . 65 ( ± 4 . 53)                           | 2 . 12 ( ± 0 . 47)                          |
| EXP-NEHVI-RFF          | 13 . 83 ( ± 5 . 95)              | 1 . 23 ( ± 0 . 05)             | 1 . 02 ( ± 0 . 05) 1            | . 46 ( ± 0 . 06)                            | 2 . 09 ( ± 0 . 13)                           | 1 . 90 ( ± 0 . 50)                          |
| MARS-NEI               | 1 . 45 ( ± 0 . 12)               | 0 . 70 ( ± 0 . 04)             | 2 . 96 ( ± 0 . 13)              | 1 . 06 ( ± 0 . 09) 0                        | . 86 ( ± 0 . 05) 0                           | . 88 ( ± 0 . 05)                            |
| MARS-TS                | 0 . 92 ( ± 0 . 07)               | 0 . 85 ( ± 0 . 05)             | 3 . 20 ( ± 0 . 21)              | 1 . 49 ( ± 0 . 05)                          | 1 . 02 ( ± 0 . 08) 1                         | . 00 ( ± 0 . 06)                            |
| MARS-UCB               | 26 . 69 ( ± 3 . 28)              | N/A                            | N/A                             | 1 . 77 ( ± 0 . 01)                          | 1 . 24 ( ± 0 . 10)                           | 2 . 61 ( ± 0 . 64)                          |
| MVAR-NEHVI             | 0 . 57 ( ± 0 . 01)               | N/A                            | 1 . 02 ( ± 0 . 04)              | N/A                                         | 1 . 76 ( ± 0 . 15)                           | N/A                                         |
| MVAR-NEHVI-RFF         | 0 . 64 ( ± 0 . 03)               | 0 . 57 ( ± 0 . 02) 1           | . 92 ( ± 0 . 06)                | OOM                                         | 0 . 87 ( ± 0 . 07)                           | OOM                                         |

As discussed in Section 5 and detailed in Appendix D, MVAR can be optimized using q NEHVI based methods. However, this comes with serious computational challenges, some of which are eased if we use q NEHVI with RFF draws. In this section, we present results comparing MARS-NEI with EXP-NEHVI-RFF and MVAR-NEHVI-RFF using the test problems from the main text. MVAR-NEHVI was also ran on two of the problems to provide a point of reference for its performance. In addition, we present both the standard q NEHVI, which uses GPs, as well as its RFF counterpart as a reference point on how the use of RFFs affects the performance of the methods on a given problem.

Figure 8 and Table 5 show that although there are instances where q NEHVI methods outperform MARS-NEI, MARS-NEI remains competitive throughout. We see that q NEHVI and its RFF counterpart are competitive, without a clear winner across problems. The expectation q NEHVI outperforms the expectation q NPAREGO in all but one problem, highlighting the benefit of using a method that aims to directly maximize the hypervolume in an MO setting. Lastly, it is worth highlighting that the MVAR-NEHVI-RFF is not reported for the Penicillin problem, which is due to the scalability limitations of IEP (discussed in Appendix D), preventing the hypervolume improvement computations from running with the available GPU memory. See Appendix I.1 for additional experiments comparing q NEHVI based methods.

Figure 8: A comparison of the methods from evaluated in the main text with additional q NEHVI-based methods: NEHVIRFF, EXP-NEHVI-RFF, MVAR-NEHVI, and MVAR-NEHVI-RFF.

picture-8.png

## I.3. Comparison of Methods Optimizing MVAR

In Figure 9 and Table 5, we present results from the test problems from the main text showing the performance of all acquisition functions that we proposed for optimizing MVAR. Results for additional problems comparing are presented in Appendix I.1. We observe that MVAR-NEHVI-RFF is a reasonably cheap method (see Table 3 for runtimes) that performs quite well on smaller problem instances. However, in larger problem instances where the size of the MVAR set is large ( > 10 ) it starts running into scalability limitations and no-longer works. Among the class of MARS methods, we find that MARS-NEI consistently performs quite well, with MARS-TS being a close second and a slightly cheaper alternative. On the other hand, MARS-UCB proves to be not as reliable, performing worse than non-robust methods in some problems. We attribute this to its fundamental reliance on the parameter ζ ( i ) n +1 , which would have to be tuned on a problem by problem basis to optimize its performance. In light of all the results, we recommend MARS-NEI as a broadly applicable and high performing method for optimizing MVAR, and recommend MVAR-NEHVI-RFF as an alternative when the size of the MVAR set is small.

Figure 9: A comparison of the methods from evaluated in the main text with additional methods for optimizing MVAR: MARS-TS, MARS-UCB, MVAR-NEHVI-RFF, and MVAR-NEHVI.

picture-9.png

## I.4. Parallel Evaluations

In Table 6, we present results demonstrating the effect of varying batch size on the optimization performance of MARS-NEI and MVAR-NEHVI-RFF. As expected, the results show that the performance of both algorithms degrade as the batch size increases. Interestingly, the degradation is rather minimal for MARS-NEI, while the effects of increasing batch size seem to be rather significant for MVAR-NEHVI-RFF. We see that MVAR-NEHVI-RFF underperforms all versions of MARS-NEI even with a batch size of 2 . In addition, there are fewer results presented for MVAR-NEHVI-RFF, which is due to the hypervolume improvement computations using IEP running into scalability limits (recall that IEP scales exponentially in the size of the joint MVAR set of the current batch of candidates). These results make a strong case for using MARS-NEI whenever one wished to evaluate candidates in parallel.

## I.5. Effect of Noise Level

The location of robust designs on a problem depends on many factors, including the magnitude of the input noise. In the edge case where there is no input noise present, the robust designs will be the same as the nominal Pareto optimal designs. As the magnitude of the input noise increases, the robust designs may start to deviate from the nominally optimal designs, with the exact behavior typically being unpredictable and heavily dependent on the problem and noise structure. In Figure 10, we present results on the GMM problem from the main text under various noise models, demonstrating how the

Table 6: Effect of the batch size on optimization performance. We report the final MVAR HV regret and 2 standard errors from 20 trials. OOM denotes that the method ran into scalability issues and did not run.

| Algorithm               | 1D Toy Problem ( d = 1 , M = 2 )   | Constrained Branin-Currin ( d = 2 , M = 2 , V = 1 )   |
|-------------------------|------------------------------------|-------------------------------------------------------|
| MARS-NEI, q = 1         | 0 . 86 ( ± 0 . 05)                 | 5 . 43 ( ± 0 . 27)                                    |
| MARS-NEI, q = 2         | 0 . 92 ( ± 0 . 04)                 | 5 . 20 ( ± 0 . 21)                                    |
| MARS-NEI, q = 4         | 0 . 92 ( ± 0 . 07)                 | 5 . 58 ( ± 0 . 38)                                    |
| MARS-NEI, q = 8         | 0 . 93 ( ± 0 . 08)                 | 5 . 59 ( ± 0 . 41)                                    |
| MVAR q NEHVI RFF, q = 1 | 0 . 87 ( ± 0 . 07)                 | 4 . 36 ( ± 0 . 13)                                    |
| MVAR q NEHVI RFF, q = 2 | 1 . 03 ( ± 0 . 05)                 | 6 . 16 ( ± 0 . 36)                                    |
| MVAR q NEHVI RFF, q = 4 | 1 . 21 ( ± 0 . 07)                 | OOM                                                   |
| MVAR q NEHVI RFF, q = 8 | OOM                                | OOM                                                   |

performance of the algorithms change in response to changes in the noise model. The list of noise models considered in this study are as follows:

- · Homoscedastic normal noise, std = 0.05: P ( ξ ) = N ( µ = 0 , Σ = 0 . 05 I 2 ) .
- · Homoscedastic normal noise, std = 0.10: P ( ξ ) = N ( µ = 0 , Σ = 0 . 10 I 2 ) .
- · Homoscedastic normal noise, std = 0.20: P ( ξ ) = N ( µ = 0 , Σ = 0 . 20 I 2 ) .
- · Heteroscedastic normal noise, std = 0.2X: P ( ξ ; x ) = N ( µ = 0 , Σ = 0 . 2 S ) with S = [ x 1 , 0; 0 , x 2 ] is the 2 × 2 matrix with the given entries.
- · Correlated normal noise: P ( ξ ) = N ( µ = 0 , Σ = 0 . 001 S ) with S = [2 . 5 , -2; -2 , 2 . 5] .
- · Multiplicative noise model from the main text: x glyph[diamondmath] ξ := xξ ' , where ξ ' ∼ N ( µ = 1 , Σ = 0 . 07 I 2 ) .

We see that MARS-NEI consistently outperforms the alternatives, except for the under homoskedastic noise with a large standard deviation of 0 . 2 . In this setting, the EXPq NPAREGO is slightly ahead, which is not too surprising since the expectation and MVAR optimal designs happen to be in the same part of the solution space under this noise model.

In addition to the GMM problem, the Constrained Branin Currin problem was also ran under multiple noise models. Along with the heteroscedastic noise model used in the main text, we also studied it using a simple homoscedastic noise model: P ( ξ ) = N ( µ = 0 , Σ = 0 . 05 I 2 ) . The plots for these are shown in Figure 11, demonstrating that MARS performs consistently under both noise models.

## I.6. Effect of n ξ on Optimization Performance

In a final side study, we analyze the effect of varying n ξ on the optimization performance of the acquisition functions optimizing expectation and MVAR. We attempted to run the Constrained Branin Currin and Disc Brake experiments with n ξ ∈ { 8 , 16 , 32 , 64 , 96 , 128 } and included the results of the algorithms that successfully completed without running into scalability issues, such as getting an out-of-memory error.

The results are shown in Figure 12. We see that the methods using GPs (EXPq NPAREGO and MARS-NEI), do not scale beyond n ξ = 64 due to the cubic complexity (with respect to the number of points and n ξ ) of posterior sampling (see Appendix D.2, the remaining methods avoid this issue since the RFF draws are deterministic functions). Overall, the results for MVAR methods show that too small of an n ξ leads to poor performance, while increasing it much beyond our default value of n ξ = 32 does not yield any a significant benefit, at least in these problems. On Constrained Branin-Currin, we observe that the performance of the expectation methods degrade as n ξ increases, which we attribute to these methods becoming more proficient at differentiating the expectation and MVAR optimal regions (which are not co-located), thus focusing their sampling away from the MVAR optimal region.

## J. Efficient Methods for Computing MVAR

Before going into the discussion, we note that the this section presents the MVAR computation for a random variable to be minimized. This simplifies the discussion by enabling the use of common terms such as CDF and quantile, since the

MVAR, as originally defined in Pr'ekopa (2012), corresponds to the α quantile of a random variable to be minimized. In the maximization setting studied in this paper, the MVAR, as defined in Definition 4.2, can be computed by first computing the MVAR of the negative of the random variable, as discussed here, then negating the result.

The existing algorithms for computing the MVAR (e.g., those presented by Pr'ekopa (2012)) presume the availability of a cheap to evaluate CDF of the random variable of interest. In the general setting we consider, with f being an arbitrary function, such as a sample path of the GP, the random variable f ( x glyph[diamondmath] ξ ) (induced by ξ ∼ P ( ξ ) ) does not admit a known CDF. Thus, to compute a QMC estimate of MVAR, we first need to compute the empirical CDF corresponding to the QMC samples of f ( x glyph[diamondmath] ξ ) .

Computing the empirical CDF of a random variable is a conceptually simple operation. All we need to do is to count the number of samples that are dominated by a given point and divide that by the total number of samples. Keeping all other objectives constant, the empirical CDF can be seen as a step function over the domain of the given objective that changes its value only at the points that correspond to one of the sample values of that objective. Since there are n ξ samples, ignoring the possibility that some samples may have equal value for some objectives, this defines an M -dimensional grid with n ξ points on each dimension, on which the empirical CDF can change its value. Thus, to fully compute the empirical CDF, we need to compute the number of samples that dominate a grid of n M ξ points, at a cost of O ( M ) comparison per point, leading to a total O ( n M ξ M ) cost for computing the empirical CDF.

Once the empirical CDF is computed, the MVAR set can easily be computed by taking the Pareto set of the points on the grid with a CDF greater than or equal to α . This part of the computation, fortunately, has a lower complexity than the CDF computations.

A careful reader might have noticed that the MVAR (in the minimization setting) is bounded from below by the independent VAR of each objective and bounded from above by the maximum value observed for that given objective. We can leverage this fact to lower the cost of computing MVAR significantly. Instead of considering the full grid of n M ξ points, we can only compute the empirical CDF for the grid formed by the objective values that exceed the independent VAR of each objective, of which there are (1 -α ) n ξ , reducing the complexity of MVAR computations to O ((1 -α ) M n M ξ M ) .

Within our code base, we provide two implementations for computing MVAR. One implementation is geared towards batched calculations with small to moderate n ξ , and the other is geared towards less memory intensive calculations with large n ξ . Both utilize efficient vectorized computations to exploit modern computing hardware. However, even with these highly optimized implementations, computing MVAR can easily become a bottleneck in a BO method, since MVAR has to be computed many times during acquisition optimization. Thus, the methods for direct optimization of MVAR that we present can still be prohibitively expensive unless the objective evaluations are significantly expensive.

## K. Multivariate Extensions of CVAR

CVAR is another popular risk measure that is commonly used with univariate random variables (Rockafellar and Uryasev, 2002). Similar to VAR, it has also been extended to the multi-variate case by various authors (e.g., Cousin and Di Bernardino (2014) and Meraklı and Kuc¸ ukyavuz (2018)). However, these extensions in general do not admit a natural interpretation, whereas MVAR provides interpretable objective specifications that the objectives (under input noise) for a given design will meet with high-probability.

Figure 10: The effect of different noise models on the GMM problem.

picture-10.png

Constrained BC, Homoscedastic

Constrained BC, Heteroscedastic

Figure 11: The effect of different noise models on the Constrained Branin Currin problem.

picture-11.png

Figure 12: Final MVAR hypervolume regret obtained using different n ξ on Constrained Branin-Currin and Disc Brake.

picture-12.png