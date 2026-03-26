1

2

## Learning Relevant Contextual Variables Within Bayesian Optimization

*

Julien Martinelli

1

2

Ayush Bharti

Armi Tiihonen

## Patrick Rinke 3

Samuel Kaski

Louis Filstroff

2,5

1 Inserm Bordeaux Population Health, Vaccine Research Institute, Université de Bordeaux, Inria Bordeaux Sud-ouest, France 2 Department of Computer Science, Aalto University, Helsinki, Finland

3 Department of Applied Physics, Aalto University, Helsinki, Finland

4 Univ. Lille, CNRS, Centrale Lille, UMR 9189 CRIStAL, F-59000 Lille, France

5 Department of Computer Science, University of Manchester, Manchester, United Kingdom

## Abstract

Contextual Bayesian Optimization (CBO) efficiently optimizes black-box functions with respect to design variables, while simultaneously integrating contextual information regarding the environment, such as experimental conditions. However, the relevance of contextual variables is not necessarily known beforehand. Moreover, contextual variables can sometimes be optimized themselves at an additional cost, a setting overlooked by current CBO algorithms. Cost-sensitive CBO would simply include optimizable contextual variables as part of the design variables based on their cost. Instead, we adaptively select a subset of contextual variables to include in the optimization, based on the trade-off between their relevance and the additional cost incurred by optimizing them compared to leaving them to be determined by the environment. We learn the relevance of contextual variables by sensitivity analysis of the posterior surrogate model while minimizing the cost of optimization by leveraging recent developments on early stopping for BO. We empirically evaluate our proposed Sensitivity-Analysis-Driven Contextual BO ( SADCBO ) method against alternatives on both synthetic and real-world experiments, together with extensive ablation studies, and demonstrate a consistent improvement across examples.

## 1 INTRODUCTION

- Bayesian optimization (BO) is a sample-efficient black-box 3
- optimization method, typically used when the objective func4
- tion is too expensive to optimize directly [Garnett, 2023]. 5
- Given an objective function that can be evaluated pointwise 6
- over a set of design variables , BO combines surrogate mod7

eling with a pre-specified policy of evaluation over the design space (the so-called acquisition function) to efficiently locate the global optimum of the function. BO has been especially useful in automatic discovery of materials [Zhang et al., 2020], molecules [Fang et al., 2021], and pharmaceutical compounds [Gómez-Bombarelli et al., 2018, Korovina et al., 2020]-problem domains in which evaluating the performance of a candidate depends on a costly experiment.

Despite the success of BO and its recent algorithmic advancements, open challenges remain for its practical use. A key implicit assumption in vanilla BO is that the objective function only depends on the design variables. This assumption is violated in many practical scenarios, wherein various uncontrolled environmental factors and experimental settings, referred to as contextual variables [Krause and Ong, 2011, Kirschner et al., 2020, Arsenyan et al., 2023], also affect the objective function. For instance, ambient humidity was found to influence the experiments in robotassisted material design [Nega et al., 2021], such that the best compound differed with humidity conditions. Moreover, in practice, the domain experts themselves might not know a priori which contextual variables are relevant, and would observe their confounding effect only during the course of the optimization process. Therefore, it is critical to identify the contextual variables that significantly affect the objective function, not only to achieve the highest optimization results, but also for the practitioners to reliably reproduce experimental results.

To deal with the uncertainty related to the contextual variables, variants of BO have been developed. In particular, Krause and Ong [2011] introduced the Contextual Bayesian optimization (CBO) framework, which uses the uncontrollable contextual information known before the experiment, like current environmental conditions, to enhance the surrogate model. Alternatively, several works have proposed to alter the simple optimization objective to make it robust in some sense, such as by taking the expectation with respect to the contextual variables [Toscano-Palmerin and Frazier, 2022], or considering distributionally-robust scenar-

3

2

S. T. John

4

Sabina J. Sloman

5

8

9

10

11

12

13

14

15

16

17

18

19

20

21

22

23

24

25

26

27

28

29

30

31

32

33

34

35

36

37

38

39

40

41

42

43

44

45

46

- ios [Bogunovic et al., 2018, Kirschner et al., 2020]. However, 47 these works consider a different setup than the original CBO 48 framework, as contextual information is only revealed after 49 the design has been sent for experiment, not before. Besides 50 this distinction, in some applications, contextual variables 51 can be controlled, and therefore set to values they may be 52 unlikely to take during passing observation. Such variables 53 are, for instance, synthesis conditions of material samples, 54 including sintering temperature or the used solvents. Certain 55 environmental conditions like room temperature or ambi56 ent humidity are also 'principally' controllable during the 57 course of an experiment [Higgins et al., 2021, Nega et al., 58 2021]. Nevertheless, whether their inclusion as optimization 59 variables is relevant or not may not be straightforward to 60 predict [Abolhasani and Brown, 2023]. Moreover, optimiz61
- ing over all the potentially relevant contextual variables can 62
- improve BO performance, but this process can be costly, 63
- thus invoking a cost-versus-efficiency trade-off. 64

Contributions. In this paper, we extend the CBO framework to settings in which the relevance of contextual variables is (i) not known beforehand, and (ii) can be optimized, but at some cost. We propose a Sensitivity-Analysis-Driven CBO ( SADCBO ) algorithm for the simultaneous identification and optimization of relevant contextual variables. SADCBO leverages recent advances in sensitivity-analysisdriven variable selection [Sebenius et al., 2022] and early stopping criteria for BO [Ishibashi et al., 2023]. We emphasize that SADCBO combines the contextual observational setting, where the context information is only observed, and the contextual optimization setting, where contextual variables can be optimized (similar to design variables), into a sequential algorithm. In effect, SADCBO provides a way to navigate the following tradeoff: should contextual variables be taken as is at no cost, or should they be steered outside of their observational distribution in order to provide more information about the objective, at a cost? We evaluate the performance of SADCBO , comparing against methods from the CBO and high-dimensional BO literature, on both synthetic and real-world cases.

65

## 2 CONTEXTUAL BAYESIAN OPTIMIZATION (CBO)

The CBO framework [Krause and Ong, 2011] deals with 66 a black-box function f : X × Z → R defined on the joint 67 space of both the design variables X ⊂ R d and contextual 68 variables Z ⊂ R c . We assume that we get noisy evalua69 tions of f , that is, we observe the output y = f ( x , z ) + ε 70 with ε ∼ N (0 , σ 2 noise ) . A Gaussian process (GP) prior 71 [Rasmussen and Williams, 2006] is placed on f ; with 72 the notation v = [ x , z ] , we write f ( v ) ∼ GP (0 , k ( v , v ' )) . 73

A GP is a stochastic process fully characterized by its mean function (taken here to be zero) and its kernel k ( v , v ' ) = cov [ f ( v ) , f ( v ' )] . This implies that for any finite-dimensional collection of inputs [ v 1 , . . . , v t ] , the function values f = [ f ( v 1 ) , . . . , f ( v t )] ⊤ ∈ R t follow a multivariate normal distribution f ∼ N ( 0 , K ) , where K = ( k ( v i , v j )) 1 ≤ i,j ≤ t is the kernel matrix. Given a dataset D t = { ( x i , z i , y i ) } t i =1 = { ( v i , y i ) } t i =1 , the posterior distribution of f ( v ) given D t is Gaussian, with analytical expressions for the mean µ t ( v |D t ) and variance σ 2 t ( v |D t ) .

In the CBO setting, we first observe the context variables, and then choose the design variables accordingly. More precisely, at iteration t +1 , a context vector z t +1 is observed, assumed to have been drawn from an unknown distribution p ( z ) , and the optimal design x ⋆ t +1 is such that

x ⋆ t +1 = arg max x ∈X f ( x , z t +1 ) . (1)

Given z t +1 and the previous t observations D t , the next candidate design x t +1 is selected using the Upper Confidence Bound (UCB) acquisition function α [Srinivas et al., 2012]:

x t +1 = arg max x ∈X α ( x , z t +1 |D t ) = µ t ( x , z t +1 |D t ) + β 1 / 2 t σ t ( x , z t +1 |D t ) , (2)

for a sequence ( β t ) t ≥ 1 . This incurs a design cost λ x .

Extending the CBO problem setup. We extend the problem setting of CBO in two ways. Firstly, we assume that only a subset of the contextual variables truly affect f . Let z = [ z (1) , . . . , z ( c ) ] be the vector of all contextual variables. For any set J belonging to the power set of { 1 , . . . , c } , denote by z ( J ) ∈ R | J | the vector of reduced dimension whose variables are indexed by J . For instance, if J = { 1 , 3 } , then z ( J ) = [ z (1) , z (3) ] . We assume that there exists a set J ⋆ , where | J ⋆ | ≪ c , such that f ( x , z ) = f ( x , z ( J ⋆ ) ) ∀ ( x , z ) . Secondly, we include the possibility of setting the value of any of the contextual variables at some cost over and above the usual design query cost λ x . This means that for all j ∈ { 1 , . . . , c } , the context variable z ( j ) can be optimized at a cost λ j . To be able to control each contextual variable, we must also assume their independence: p ( z ) = ∏ c j =1 p ( z ( j ) ) . With these additional assumptions, we aim to maximize the function f in a cost-efficient manner, while identifying the optimal set J ⋆ . This provides the user with a comprehensive summary of the relevant contextual variables found through optimization, thus ensuring reproducibility and explainability. Unlike CBO, the ability to control contextual variables allows us to judge whether or not one should optimize contextual variables to learn more about the objective (albeit at a cost), or if the current sampled context is already informative enough. Specifically, we aim to maximize the objective

( x ⋆ t +1 , z ⋆ t +1 ) = arg max ( x , z ( J ⋆ ) t +1 ) ∈X× ∏ j ∈ J ⋆ Z j f ( x , z t +1 ) (3)

74

75

76

77

78

79

80

81

82

83

84

85

86

87

88

89

90

91

92

93

94

95

96

97

98

99

100

101

102

103

104

105

106

107

108

109

110

111

112

113

114

115

where, for all j ∈ J ⋆ , we optimize z ( j ) t +1 at cost λ j , and all 116 other elements j ' ∈ { 1 , . . . , c } \ J ⋆ of z t +1 remain at their 117 values sampled from the environment ( z ( j ' ) t +1 ∼ p ( z ( j ' ) ) ). 118

## 3 METHODOLOGY

To solve the extended CBO problem introduced in Section 2, 119 we identify relevant contextual variables, building on a vari120 able selection technique from the GP literature [Sebenius 121 et al., 2022]. Section 3.1 describes our adaptation of this 122 method to the optimization setting, by restricting the dataset 123 to high function values. Section 3.2 then presents our se124 quential algorithm SADCBO , which employs the adapted 125 variable selection method in solving the optimization prob126 lem. A flowchart summarizing the proposed method can be 127 found in Figure S1. 128

## 3.1 VARIABLE SELECTION FOR CBO VIA SENSITIVITY ANALYSIS

To handle the presence of contextual variables that can be 129 optimized, one approach is to include them in the design 130 space. However, such a strategy can be infeasible when their 131 relevance is not known a priori and domain experts can only 132 provide a candidate set of potentially relevant contextual 133 variables. Indeed, this leads to an exponential expansion of 134 the search space, while at the same time increasing the cost 135 of optimization. In such cases, it is crucial to identify the 136 relevant contextual variables, i.e., to find (a good approx137 imation to) the optimal set J ⋆ . This not only allows us to 138 optimize the function more efficiently but also provides ad139 ditional insights about the experiment to the domain experts. 140

To approximate the optimal set J ⋆ , we include those contex141 tual variables that are most relevant for identifying the opti142 mum, which we estimate using sensitivity analysis. Specifi143 cally, we adapt the Feature Collapsing (FC) method [Sebe144 nius et al., 2022]. The FC method perturbs training points 145 (namely, by setting one feature to zero), and measures the 146 induced shift in the posterior predictive distribution in terms 147 of KL divergence. Given a dataset D t = { ( x i , z i , y i ) } t i =1 , 148 the relevance r i,j on the i th sample of the j th contextual 149 variable z ( j ) i is computed as 150

r i,j = KL ( p ( y ⋆ | x i , z i , D t ) || p ( y ⋆ | x i , z i ⊙ ξ [ j ] , D t )) , (4)

̸

where ξ [ j ] = [ ξ (1) , . . . , ξ ( c ) ] is a vector so that ξ ( j ) = 151 0 , and ξ ( j ' ) = 1 for j ' = j , and ⊙ is the element-wise 152 multiplication. The relevance score of the j th contextual 153 variable is then computed as an average over D t : 154

FC D t ( j ) = 1 |D t | |D t | ∑ i =1 ( r i,j ∑ c j ' =1 r i,j ' ) . (5)

The FC scores obtained in this manner reveal the variables 155 that are relevant for predicting the output across D t . As our 156

goal is to maximize f , we are interested in identifying contextual variables that are relevant for high function values. Hence, we adapt Equation (5) to the BO setting by modifying the dataset over which the scores are averaged. We use information about high function values from two different sets: (1) The subset D γ t associated with the highest output values observed so far:

157

158

159

160

161

162

163

D γ t t = { ( x i , z i , y i ) ∈ D t | y i /y best ≥ γ t } , (6)

where y best = max 1 ≤ i ≤ t y i is the current observed maxi164 mum. For example, using γ t = 0 . 8 ∀ t would yield a D γ t t 165 that consists of the highest 20% observations so far. (2) We 166 select a batch of Q points D Q t := { ( x ⋆ q , z t +1 ) } Q q =1 that are 167 promising given the next context z t +1 : 168

{ x ⋆ q } Q q =1 = argmax { x q } Q q =1 ∈X Q α Batch ( { ( x q , z t +1 ) } Q q =1 |D t ) , (7)

where α Batch denotes a batched version of the acquisition function α such as Q -UCB for UCB [Wilson et al., 2017]. We use the union D BO t = D γ t t ∪ D Q t as our dataset for FC. Therefore, we compute FC D BO t based on Equation (5). The importance of working with D BO t instead of D t is illustrated in Figure 1 on a toy example.

169

170

171

172

173

174

Wesuccessively select the indices of the contextual variables 175 with the highest FC scores until their cumulative FC score 176 exceeds some chosen threshold η ∈ [0 , 1] , meaning that the 177 selected variables explain the fraction η of the output sensi178 tivity amongst all contextual variables. Let J η denote the set 179 of indices of the selected contextual variables. We train a GP 180 surrogate based on { ( x i , z ( J η ) i , y i ) } t i =1 and can select a new 181 design through maximization of the acquisition function α : 182

x t +1 = arg max x ∈X α ( x , z ( J η ) t +1 |D t ) . (8)

Note that other measures of variable relevance could have 183 been used, e.g., the method proposed by Spagnol et al. 184 [2019] based on maximum mean discrepancy [Gretton et al., 185 2012]. We found FC to perform better (see Section 5.2). 186

## 3.2 SENSITIVITY-ANALYSIS-DRIVEN CBO ( SADCBO )

Building on top of the variable selection method discussed in 187 Section 3.1, we now present SADCBO , a sequential method 188 for performing BO in the presence of irrelevant contextual 189 variables. SADCBO proceeds in two phases. 190

In the first, observational phase, we choose to only observe 191 the values of the contextual variables without optimizing 192 over them. This ensures that we do not waste budget op193 timizing the contextual variables when their relevance is 194 computed based on a limited amount of data, and hence 195 can be noisy. We select the contextual variables based on 196 their FC relevance and then use vanilla CBO as described 197

Figure 1: Sensitivity analysis on D BO t characterizes variable importance at the optimum faster than D t . Top left : 2D black-box objective together with the queries produced along a BO trajectory. Initial samples are represented by empty dark-colored triangles, newly obtained samples as dots with an increasingly lighter color. Top right : Best value found during the optimization trial. Bottom left : Sensitivity indices for z (1) and z (2) averaged over D BO t . As we converge to the optimum, D BO t mainly involves samples close to the optimum, leading to a different variable relevance ranking (iteration 30 to the end; z (1) is more relevant) compared to the early iterations (10 to 30; z (2) is more relevant). Bottom right : Sensitivity indices computed on the whole dataset D t do not converge as quickly and do not capture the shift in relevance close to the optimum.

picture-1.png

in Section 2 to optimize the design variables. Thus, in this 198 phase, we leverage the available contextual information to 199 guide design selection. 200

In the early stage of the optimization, cheap queries where 201 contextual variables are not optimized still provide a con202 siderable amount of information. The information gained 203 from purely observing contextual variables will, however, 204 saturate at some point, leading to diminishing simple regret 205 differences. At this point, it becomes necessary to pay the 206 higher price to control more dimensions of the input space. 207 This motivates the introduction of a second phase, in which 208 contextual variables can have their values arbitrarily set, 209 through optimization. 210

In the second, contextual optimization phase, we optimize 211 the contextual variables selected at each iteration based on 212 their FC relevance. As optimizing a context variable z ( j ) is 213 associated with a cost λ j , we modify the FC relevance in 214

Equation (5):

˜ FC D t ( j ) = FC D t ( j ) /λ j (9)

Our variable selection criterion can then be interpreted as the 216 degree of sensitivity per unit cost . This allows SADCBO to 217 automatically trade off a variable's potential to greatly affect 218 the optimum with the associated optimization cost. As be219 fore, once the contextual variables z ( J η ) have been selected, 220 we train a GP surrogate using { ( x i , z ( J η ) i , y i ) } t i =1 and select 221 the next design and contextual variables to query as 222

( x t +1 , z ( J η ) t +1 ) = arg max ( x , z ( Jη ) ) ∈X× ∏ j ∈ Jη Z j α ( x , z ( J η ) |D t ) . (10)

In effect, J η represents our approximation for J ⋆ as intro223 duced in Equation (3). Note that our acquisition function is 224 not cost-weighted, as cost-weighted acquisition functions 225 can dramatically underperform [Lee et al., 2021], specifi226 cally for non-continuous cost models. Including the cost at 227 the model selection level avoids this issue. 228

## Switching from observational to optimization phase.

We employ the criterion proposed by Ishibashi et al. [2023] for determining the stopping time in BO. Using this criterion, we detect the point at which the optimization gain based on purely observing the contextual variables diminishes, following which the contextual optimization phase begins. We now briefly describe the details of this switching criterion.

With v = [ x , z ] , let v ⋆ t = argmax v ∈D t f ( v ) be the current best candidate point in the dataset up to time t . Denoting f ⋆ := max v ∈V f ( v ) , let R t = f ⋆ -E ˆ f ∼ p ( f |D t ) [max v ∈V ˆ f ( v )] be the expected minimum simple regret. Then, with probability 1 -δ , ∆ R t = | R t -R t -1 | can be upper bounded by ∆ ˜ R t with

∆ ˜ R t = v ( ϕ ( g ) + g Φ( g )) + | ∆ µ ⋆ t | + κ δ,t -1 √ 1 2 KL ( p ( f |D t ) || p ( f |D t -1 )) , (11)

where ϕ ( · ) and Φ( · ) are the p.d.f. and c.d.f. of a stan237 dard Gaussian distribution, respectively, ∆ µ ⋆ t := µ t ( v ⋆ t ) -238 µ t -1 ( v ⋆ t -1 ) , v := √ σ 2 t ( v ⋆ t ) -2Σ t ( v ⋆ t , v ⋆ t -1 ) + σ 2 t ( v ⋆ t -1 ) , 239 g := ∆ µ ⋆ t /v , and κ δ,t -1 is a sequence indexed by t and de240 pending on δ . Then, we switch from the observational to the 241 optimization phase in SADCBO when ∆ ˜ R t ≤ s t , where 242

s t := ( σ t -1 ( v ⋆ t ) + κ δ,t -1 / 2 ) σ t -1 ( v t ) σ noise √ -2 log δ σ 2 t -1 ( v t ) + σ 2 noise . (12)

Further details about the derivation of s t and the expression of κ δ,t -1 can be found in Appendix B. The entire algorithm is summarized in Algorithm 1.

243

244

245

229

230

231

232

233

234

235

236

- 1: Input : initial dataset D 0 , hyperparameters η and γ , batch size Q , budget Λ , costs λ x , λ 1 , . . . , λ c
- 2: Train initial GP using D 0 and all variables [ x , z ] . phase = observational. t = 1 .
- 3: while Λ ≥ λ x do
- 5: Assemble dataset D BO t (Equations (6) and (7))
- 4: Receive context z t +1 ∼ p ( z )
- 6: Compute FC D BO t ( j ) for all j (Equation (5) or (9) if phase = optimization)
- 7: In descending order, add indices to J η until ∑ j ∈ J η FC D BO t ( j ) > η
- 9: Get x t +1 (Equation (8)) (and z t +1 (Equation (10)) if phase = optimization)
- 8: Train lower-dimensional GP { ( x i , z ( J η ) i , y i ) } t i =1
- 10: Acquire observation y t +1 at [ x t +1 , z t +1 ]
- 11: D t +1 ←D t ∪ { ( x t +1 , z t +1 , y t +1 ) }
- 12: Retrain full GP
- 13: if phase = observational and ∆ ˜ R t ≤ s t [based on p ( f |D t +1 ) ] (Equation (12)) then
- 14: phase = optimization
- 15: end if
- 16: Λ ← Λ -λ x + ∑ j ∈ J η λ j , t ← t +1
- 17: end while

## 4 RELATED WORK

Robust BO. Bogunovic et al. [2018], Kirschner et al. 246 [2020], Husain et al. [2023] and Saday et al. [2023] perform 247 worst-case optimization under fluctuations of the contextual 248 variables. In particular, Distributionally-Robust BO [DRBO, 249 Kirschner et al., 2020] tries to maximize the expected black250 box function value under the worst-case distribution of the 251 contextual variables. This worst-case distribution belongs to 252 an 'uncertainty set', a ball centered around a reference dis253 tribution that is gradually learned [Tulabandhula and Rudin, 254 2014]. However, as in Krause and Ong [2011], these works 255 assume that the relevant contextual variables are known a 256 priori , and can only be observed, after the designs have 257 been selected , and not controlled. 258

High-dimensional BO. Due to the curse of dimensional259 ity, the performance of standard BO is severely degraded 260 when applied in high-dimensional input spaces. To tackle 261 this problem, most approaches either aim at carrying out 262 BO in a lower-dimensional space instead of the original or 263 work with a structured GP surrogate. A lower-dimensional 264 subspace can be found in a data-agnostic manner, for in265 stance by randomly dropping dimensions of the problem [Li 266 et al., 2017] or considering tree-like random decomposi267 tions [Ziomek and Bou-Ammar, 2023]. Data-driven meth268 ods based on various measures of feature relevance have also 269 been proposed [Spagnol et al., 2019, Shen and Kingsford, 270

- 2021]. In contrast, structured surrogate methods encode 271

structural information about the objective, for instance using an additive kernel, yielding an acquisition function that is additive under the provided decomposition [Rolland et al., 2018]. Finally, Eriksson and Jankowiak [2021] and Liu et al. [2023] proposed using a sparsity-enforcing GP surrogate, equipped with a heavy-tailed horseshoe prior on the squared inverse lengthscales.

272

273

274

275

276

277

278

Cost-aware BO. In most methods, the BO budget is given 279 in iterations, implicitly assuming that each evaluation has 280 the same cost. In practice, cost may vary significantly across 281 different regions of the input space [Lee et al., 2020], or 282 depend on the number of variables we optimize over. Cost283 aware BO integrates the cost-constrained nature of the prob284 lem, usually within the acquisition function. Let us also 285 mention more involved strategies like constrained Markov 286 decision processes when the total budget is known before287 hand [Lee et al., 2021]. The recent work by Tay et al. [2023] 288 carries out Robust BO while at the same time involving a 289 notion of controlled variables at a cost. However-unlike 290 our framework-they require the nonselected variables to 291 be sampled from a known distribution at each iteration. 292

## 5 EXPERIMENTAL RESULTS

We evaluate our approach on several real-world examples 293 and synthetic functions, described in Section 5.1. We com294 pare against multiple baselines (Table 1) and present results 295 in Section 5.2. In Section 5.3 we discuss the influence of vari296 ous experimental settings: number of noise variables present, 297 contextual variable query cost, surrogate and method hy298 perparameters. We conclude by presenting several insights 299 regarding the phase-switching criterion. 300

Baselines. We benchmark our approach, coined SADCBO , against baselines referenced in Table 1. In particular, MMDBO operates variable selection in BO through an MMD-based measure of sensitivity [Spagnol et al., 2019] and is detailed in Appendix C, whereas Dropout [Li et al., 2017] randomly selects half of the contextual variable for optimization. Next, CaBO [Lee et al., 2020] performs vanilla BO over [ x , z ] , using a cost-weighted acquisition function. The cost model employed here is a smoothed version of our noncontinuous cost model, using a Gaussian curve. Finally, CBO refers to the Contextual BO framework proposed by Krause and Ong [2011]. As a way to assess the impact of contextual variables and selection mechanisms, we also report CUBO and VBO : Context-Unaware BO over the designs x only and Vanilla BO over both design and contextual variables [ x , z ] .

Implementation details. We fix the hyperparameters of SADCBO to η = 0 . 8 , Q = 10 , γ t = 0 . 8 ∀ t . For the GP surrogate, we use a squared-exponential kernel with independent lengthscales for each variable, learned through marginal likelihood maximization. We use the UCB acquisition strategy, as well as Q -UCB for computing D Q t

316 317 318 319 320 321

301

302

303

304

305

306

307

308

309

310

311

312

313

314

315

Table 1: Methods used in experiments.

|                            | Name                 | Description                                                                                                                                                                                                                  |
|----------------------------|----------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Without variable selection | CUBO VBO CaBO CBO    | Context-Unaware BO over x only Vanilla BO over [ x , z ] Cost-Aware BO over [ x , z ] [Lee et al., 2020] Contextual BO using all contexts z [Krause and Ong, 2011] Randomly drop half of context variables [Li et al., 2017] |
| With variable selection    | Dropout MMDBO SADCBO | Maximum mean discrepancy-driven BO [Spagnol et al., 2019] Sensitivity analysis-driven CBO ( This work )                                                                                                                      |

Table 2: Dimensionality of the experiments. For synthetic experiments, additional dimensions stand for (artificial) noise variables, put on top of the design and contextual variables.

| Experiment   | All dimensions   |   Design variables |   Contextual variables |
|--------------|------------------|--------------------|------------------------|
| Portfolio    | 5                |                  3 |                      2 |
| Yacht        | 6                |                  4 |                      2 |
| Robot        | 14               |                  6 |                      8 |
| Molecule     | 21               |                  3 |                     18 |
| EggHolder    | 2 + 4            |                  1 |                      1 |
| Hartmann4D   | 4 + 3            |                  2 |                      2 |
| Hartmann6D   | 6 + 6            |                  3 |                      3 |
| Ackley       | 5 + 8            |                  2 |                      3 |

(Equation (7)) [Wilson et al., 2017]. In all experiments, we 322 assume that any variable, design or contextual ones, has 323 cost λ j = 1 ∀ j ∈ { 1 , . . . , d + c } , except in a dedicated 324 study in Section 5.3. Our algorithm is implemented using 325 the BoTorch framework [Balandat et al., 2020]. Code can 326 be accessed at https://github.com/julienmartinelli/ SADCBO 327

- .

## 5.1 EXPERIMENTS

We benchmark on 4 real-world and 4 synthetic experiments 328 (Table 2) described in brief here and detailed in Appendix E. 329

Portfolio Optimization 5D. This dataset was first intro330 duced by Cakmak et al. [2020]. The goal is to optimize 331 three design variables, which stand for the hyperparameters 332 of a trading strategy, to maximize return under random en333 vironmental conditions. There are two contextual variables, 334

- namely: bid-ask spread and the borrowing cost. 335

Yacht Hydrodynamics 6D. This dataset comes from the 336 UCI Machine Learning Repository [Gerritsma et al., 2013]. 337 The optimization problem is to maximize the residuary re338 sistance per unit weight of displacement of a yacht by con339 trolling its 5-dimensional hull geometry coefficients. Design 340 variables are the first four dimensions of the hull geome341 try coefficients. The contextual variables are the last hull 342 geometry dimension and the Froude number. 343

Molecule structure optimization 21D. This computational 344 chemistry example consists of optimizing the bond angles 345 in an alanine molecule to determine the lowest energy con346 former, i.e., the structure the molecule will likely take in 347

nature. These problems are complicated by high dimensionality. We consider the Alanine, a molecule with 21 angular variables: 3 key variables based on prior domain knowledge set as design variables, and 18 other angles treated as contextual variables. Molecular energies are calculated with the AMBER forcefield [Case et al., 2023] at each round of BO.

Robot pushing task 14D. We follow Wang et al. [2017] and consider a control parameter tuning problem for robot pushing. This real-world function returns the distance between a designated goal location and two objects being pushed by two robot hands, whose trajectory is determined by 14 parameters specifying the location, rotation, velocity and moving direction, among others. There are 6 design variables and 8 contextual variables.

Synthetic experiments. We also consider four synthetic test functions, (see Table 2 and Appendix E.2 for details). A min-max transformation is performed on the input data, scaling it to the unit cube: X × Z = [0 , 1] d + c . Similarly, the output is scaled between [0 , 1] and a noise term ε ∼ N (0 , σ 2 noise ) is added with σ 2 noise = 0 . 001 . The contextual variable distribution is p ( z ) = U ([0 , 1] c ) .

348

349

350

351

352

353

354

355

356

357

358

359

360

361

362

363

364

365

366

367

368

## 5.2 RESULTS

Real-world experiments. In each plot from Figure 2, we 369 report the best value found by each baseline as a function 370 of the number of iterations. In real-world experiments (Fig371 ure 2 a ), SADCBO (in red with white markers) quickly con372 verges to the optimum. SADCBO consistently outperforms 373 the first baselines VBO and CUBO , even though in the Molec374 ular Shape example, SADCBO and CUBO perform on par 375 due to the good choices of the domain experts on the design 376 variables. Except for the Robot Pushing task, the differ377 ence between SADCBO and CBO (in blue) is marginal in 378 the real-world experiments. The latter enhances the surro379 gate model with information from sampled contexts, while 380 our method may even optimize selected contextual vari381 ables if needed. Given that these baselines perform similarly, 382 combined with the observation that optimizing only design 383 variables ( CUBO , in yellow) produces poor results for the 384 Portfolio and Yacht problems, we can conclude that contex385 tual variables play a significant part in maximizing these 386 two objectives. The cost-aware BO baseline CaBO performs 387 poorly in all tasks. Dropout and MMDBO consistently un388 derperforms, except on the Yacht example for the latter. 389 These baselines perform variable selection in a random man390 ner for Dropout and using Hilbert-Schmidt Independence 391 Criterion for MMDBO [Gretton et al., 2007], two strategies 392 that do not seem to surpass the Feature Collapse method 393 implemented in SADCBO . This observation highlights the 394 need for an informed variable selection strategy. In-depth 395 findings for the Molecule experiment are presented in Ap396 pendix E.1 and provide additional explanations as to why 397 SADCBO clearly outperforms MMDBO and Dropout . 398

Figure 2: Benchmark of the different methods. (a) On real-world datasets, SADCBO (red curve with white markers) performs on par with other baselines and is the top performer for the Robot Pushing task. (b) On synthetic functions, SADCBO outperforms other baselines in three cases out of four. (c) Histograms of phase switching criteriong time for SADCBO computed for the Hartmann6D ( c.1 ) and Hartmann4D problems ( c.2 ). (d) Inclusion probability of each contextual variable for SADCBO computed for the Hartmann6D ( d.1 ) and Hartmann4D problems ( d.2 ). Each panel shows the mean ± 2 standard error across N = 100 trials.

picture-2.png

- Synthetic experiments. Figure 2 b displays the best value 399
- found by each baseline for synthetic functions. SADCBO 400
- ranks first on 3 out of 4 examples, closely followed by the 401
- cost-aware baseline CaBO , which performs much better on 402
- synthetic experiments than on the real-world ones. The con403
- textual BO baseline CBO that obtained second to best results 404
- in real-world experiments, is now less performant, due to 405
- the fact that it does not optimize the context, similarly as 406
- CUBO . This seems to be particularly critical for Ackley5D, 407
- whereas for Hartmann6D/Hartmann4D, simply enhancing 408
- the surrogate with contextual variable observation already 409
- leads to a large performance gap between CUBO and CBO . 410
- Lastly, VBO does a poor job as it optimizes every variable, 411
- thus spending a large fraction of the budget every iteration. 412
- For Hartmann6D and Hartmann4D, Figure 2 c reports the 413
- time at which SADCBO 's switching criterion (Equation (12)) 414
- kicks in, in proportion to the total budget, demonstrating 415 that both phases are leveraged in our approach. 416
- Finally, Figure 2 d 417
- reports the sensitivity indices computed at
- each iteration for each contextual variable, averaged across 418
- whole trajectories of multiple trials. For Hartmann6D, the re419
- sults match the Sobol sensitivity analysis results (Table S1), 420
- even though global sensitivity indices may differ from sensi421
- tivity indices with respect to the function optimum. Similar 422
- findings apply to Hartmann4D (Table S2). Results for other 423
- problems can be found in Figure S2. 424

Main takeaways. Quantitatively, SADCBO achieves the best overall performances, ranking first in 7 out of 8 problems, although other methods obtained comparable performances on 5 out of 8 problems.

The second-best and third-best methods, CBO and MMDBO , both severely underperform in two examples (Ackley and Hartmann6 for CBO , Molecular Shape and EggHolder for MMDBO ). While the improvements provided by SADCBO may seem marginal, they are consistent across the benchmark.

We hypothesize that this consistent behavior stems from our two-stage approach, which allows SADCBO to be versatile. SADCBO can handle both cases where the impact of the contextual variables on the function is limited (hence it is not worth spending budget to control them) and cases where spending budget leads to informative queries are simultaneously well-handled. For instance, SADCBO effectively reverted to a CBO algorithm in the Molecular Shape problem, due to an optimization phase mostly triggered at the end of the run. Meanwhile, for the Ackley function, the optimization phase was triggered in the first quarter of the budget on average, leading to SADCBO outperforming CBO .

## 5.3 SENSITIVITY ANALYSIS

We now report experiments assessing the robustness of 426 SADCBO 's performance to several modifications, either at 427 the hyperparameter level or at the experiment setting level. 428 The latter includes assessing performance when increas429 ing the number of noise variables, varying the contextual 430 variable query cost, or varying the surrogate model. Next, ad431 ditional experiments illustrate the sound behavior of the pro432 posed phase switching criterion implemented in SADCBO . 433

Number of irrelevant contextual variables. We compare the performance reached by SADCBO when adding an increasingly larger number of noise variables and find that even for a large number of irrelevant contextual variables, SADCBO reaches top performance on 3 out of 4 examples (Figure S3). The gap in performance between SADCBO and CaBO , Dropout and MMDBO seems to overall grow with the number of nuisance variables, in favor of SADCBO .

Contextual variables optimization cost. We investigate four different values for the query cost of contextual variables (Figure S4). For extremely cheap contextual variables λ j = 0 . 1 for all j , that is, ten times cheaper than a design variable, VBO performs favorably, as optimizing over all inputs [ x , z ] is cheap. SADCBO remains competitive in this configuration, even though MMDBO and CaBO perform on par. For a moderate cost λ j = 1 (the cost model considered in Figure 2), SADCBO obtains the lowest average rank over all four test functions. For expensive contextual variables, λ j = 3 or λ j = 10 , CaBO seems overall more suitable, although closely followed by SADCBO , and CBO .

434

435

436

437

438

439

440

441

442

443

444

445

446

447

448

449

450

451

452

453

Sparsity-enforcing surrogates with SADCBO . As 454 SADCBO relies on a posterior sensitivity analysis to select 455 the relevant contextual variables, and is hence agnostic to 456 the choice of GP surrogate model, it can be combined with 457 other methods that induce sparsity via the GP surrogate. 458 One such method is by Eriksson and Jankowiak [2021], who 459 introduced a sparsity-enforcing GP surrogate equipped with 460 a horseshoe prior on the square inverse lengthscales, coined 461 SAASBO . In Figure 3, we compare SADCBO with the 462 combined method SAASBO+SADCBO , with both having the 463 same hyperparameters. We observe that SAASBO+SADCBO 464 improves over just SAASBO in all the synthetic examples, 465 and is also better than SADCBO in two out of four examples. 466 Note that the performance of SAASBO+SADCBO may 467 further improve through hyperparameter tuning. 468

SADCBO phase switching criterion. We ensure that the criterion is well-behaved: the more information about the output is contained in the contextual variables, the later the phase switching occurs (Figure S5). Even though the stopping criterion was initially devised for vanilla BO, its application in a CBO setting is fruitful. Figure 4 further illustrates the soundness of the phase switching criterion.

469

470

471

472

473

474

475

Figure 3: Combining SADCBO with sparsity-enforcing surrogate SAASBO . For any variable, the associated query cost is 1. p ( z ) = U ([0 , 1] c ) . The combination is fruitful and improves the performances of SAASBO .

picture-3.png

Figure 4: Assessing SADCBO 's phase switching criterion on the Hartmann6D function. The iteration selected by the adaptive stopping criterion implemented in SADCBO yields one of the best BO trials. Each curve is computed as an average of 10 different random seeds.

picture-4.png

Using the Hartmann6D function under the same setting 476 as described above, the mean switching iteration found by 477 SADCBO over 100 different runs was collected. Then, new 478 BO runs using SADCBO with a fixed phase switching time 479 i ∈ { 1 , . . . , 100 } were performed. This was done 10 times 480 for each switching time, using different random seeds for 481 the initial dataset. The switching time found by SADCBO 482 yields one of the best runs, validating the use of the criterion. 483

SADCBO hyperparameters. We vary the 3 hyperparam484 eters of SADCBO : η, γ, Q . Unsurprisingly, the cumulative 485 sensitivity threshold η stands out as the most relevant pa486 rameter: as its value decreases, fewer variables are included, 487 at which point not all relevant ones are selected, leading to 488 reduced performance (Appendix D). 489

## 6 CONCLUSION

In this paper, we extended Contextual BO [Krause and Ong, 490 2011] to settings in which the contextual variables may be 491 not only observed but also optimized at a cost. We intro492

duced SADCBO , an algorithm designed to select relevant context variables affecting the experimental outcomes by efficiently leveraging information present in both the observational and the interventional data. SADCBO results in more adequate surrogate models, and ensures the reproducibility of experiments by controlling for such relevant variables. In that respect, SADCBO should be used for practical applications where contextual variables can have an influence while being controllable. This includes, e.g., the development of new high-throughput materials or drugs, where machine learning strategies are being increasingly used [Zhang et al., 2020, Gómez-Bombarelli et al., 2018]. SADCBO can also be combined with any GP surrogate. Thus, if a practitioner believes that a specific contextual variable should be included, this can be easily achieved. Conversely, the variable selection procedure could be generalized to discard design variables as well. Lastly, recent work [Branchini et al., 2023] proposed to perform BO under the assumption that the input variables and the output are linked by a causal directed acyclic graph, learning the graph whilst maximizing the objective function. Despite its high computational complexity, applying this technique to our particular problem might be promising.

Limitations and future work. To achieve cost efficiency, SADCBO integrates the query cost at the variable selection level and employs an early stopping criterion. The latter only depends on an upper bound on the instantaneous regret difference and is therefore not cost-aware. Adding a notion of remaining budget to this criterion would certainly benefit our approach. On a similar note, while our algorithm incorporates cost, more effort could be put into specifying the costs. In our experiments, they were set to 1 for all variables to prevent bias in the results, and we carried out an ablation study with different costs in Section 5.3. Yet, it is worth mentioning that our method is compatible with the inference of black-box, input-dependent costs, similarly to CaBO [Lee et al., 2020]. One would simply need to modify Equation (9), replacing λ j by the learned cost. An interesting avenue for future work would be to elicit knowledge of experimental costs from domain experts in real-world situations.

## Acknowledgements

JM acknowledges the support of the Research Council of Finland under the HEALED project (grant 13342077). AB, AT, STJ, and PR were supported by the Research Council of Finland Flagship programme: Finnish Center for Artificial Intelligence FCAI. AT further acknowledges funding from the European Union's Horizon 2020 research and innovation programme under the Marie Skłodowska-Curie grant agreement No. 101059891. SJS and SK were supported by the UKRI Turing AI World-Leading Researcher Fellowship, [EP/W002973/1].

493

494

495

496

497

498

499

500

501

502

503

504

505

506

507

508

509

510

511

512

513

514

515

516

517

518

519

520

521

522

523

524

525

526

527

528

529

530

531

532

533

534

535

536

537

538

539

540

541

542

543

| 544                                                       | References                                                                                                                                                                                                                                                                                                                                                                                    |
|-----------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 545 546                                                   | Milad Abolhasani and Keith A. Brown. Role of AI in exper- imental materials science. MRS Bulletin , 2023.                                                                                                                                                                                                                                                                                     |
| 547 548                                                   | Vahan Arsenyan, Antoine Grosnit, and Haitham Bou- Ammar. Contextual causal Bayesian optimisation. arXiv                                                                                                                                                                                                                                                                                       |
| 549                                                       | preprint arXiv:2301.12412 , 2023. Maximilian Balandat, Brian Karrer, Daniel Jiang, Samuel                                                                                                                                                                                                                                                                                                     |
| 550 551                                                   | Daulton, Ben Letham, Andrew G Wilson, and Eytan Bak- shy. BoTorch: A Framework for Efficient Monte-Carlo Bayesian Optimization. In                                                                                                                                                                                                                                                            |
| 552 553 554                                               | Advances in Neural Informa- tion Processing Systems (NeurIPS) , 2020.                                                                                                                                                                                                                                                                                                                         |
|                                                           | Advances in Neural Information , 2018. Branchini, Virginia Aglietti, Neil Dhir, and Theodoros Damoulas. Causal entropy optimization. In                                                                                                                                                                                                                                                       |
|                                                           | , 2023. Advances in Neural Information Processing Systems , pages 20130-20141, 2020.                                                                                                                                                                                                                                                                                                          |
|                                                           | Intelligence and Statistics (AISTATS) Enlu Zhou. Bayesian Optimization of Risk Measures. In Shalom, Joshua Berryman, Scott Brozell, David Cerutti, Thomas Cheatham, Gerardo Andrés Cisneros, Vinícius Cruzeiro, Tom Darden, Negin Forouzesh, George Gi- ambasu, Timothy Giese, Michael Gilson, Holger Gohlke, Andreas Götz, Julie Harris, Saeed Izadi, and Peter Koll- man. Amber 2023, 2023. |
| 555 556 557 558                                           |                                                                                                                                                                                                                                                                                                                                                                                               |
|                                                           | Ilija Bogunovic, Jonathan Scarlett, Stefanie Jegelka, and Volkan Cevher. Adversarially robust optimization with Gaussian processes. In Processing Systems (NeurIPS)                                                                                                                                                                                                                           |
| 559 560 561 562                                           | Nicola                                                                                                                                                                                                                                                                                                                                                                                        |
| 563                                                       |                                                                                                                                                                                                                                                                                                                                                                                               |
| 564                                                       | Proceedings of the International Conference on Artificial Sait Cakmak, Raul Astudillo Marban, Peter Frazier, and                                                                                                                                                                                                                                                                              |
| 565 566 567 568 569                                       |                                                                                                                                                                                                                                                                                                                                                                                               |
|                                                           | (NeurIPS) David Case, H. Metin Aktulga, Kellon Belfon, Ido Ben-                                                                                                                                                                                                                                                                                                                               |
|                                                           | David Eriksson and Martin Jankowiak. High-dimensional Bayesian optimization with sparse axis-aligned subspaces.                                                                                                                                                                                                                                                                               |
| 570 571 572 573 574                                       | Proceedings of the Thirty-Seventh Conference on Un- certainty in Artificial Intelligence (UAI) , 2021.                                                                                                                                                                                                                                                                                        |
| In Lincan Fang, Esko Makkonen, Milica Todorovi'c, Patrick | Rinke, and Xi Chen. Efficient Amino Acid Conformer Search with Bayesian Optimization. Journal of Chemical , 17, 2021. Bayesian Optimization . Cambridge Uni-                                                                                                                                                                                                                                  |
| 580 581 582 583 584 585 586 587 588 589                   |                                                                                                                                                                                                                                                                                                                                                                                               |
| 575 576 577 578                                           |                                                                                                                                                                                                                                                                                                                                                                                               |
| 579                                                       |                                                                                                                                                                                                                                                                                                                                                                                               |
| 590                                                       | Timothy D. Hirzel, Ryan P. Adams, and Alán Aspuru-                                                                                                                                                                                                                                                                                                                                            |
| 591 592                                                   | Guzik. Automatic Chemical Design Using a Data-Driven ACS Central                                                                                                                                                                                                                                                                                                                              |
|                                                           | , 4(2):268-276, 2018.                                                                                                                                                                                                                                                                                                                                                                         |
|                                                           | Continuous Representation of Molecules. Science                                                                                                                                                                                                                                                                                                                                               |
|                                                           | naud, José Miguel Hernández-Lobato, Benjamín Sánchez- Lengeling, Dennis Sheberla, Jorge Aguilera-Iparraguirre,                                                                                                                                                                                                                                                                                |
|                                                           | Rafael Gómez-Bombarelli, Jennifer N. Wei, David Duve-                                                                                                                                                                                                                                                                                                                                         |
|                                                           | namics. UCI Machine Learning Repository, 2013.                                                                                                                                                                                                                                                                                                                                                |
|                                                           | J. Gerritsma, R. Onnink, and A. Versluis. Yacht Hydrody-                                                                                                                                                                                                                                                                                                                                      |
|                                                           | versity Press, 2023.                                                                                                                                                                                                                                                                                                                                                                          |
|                                                           | Roman Garnett.                                                                                                                                                                                                                                                                                                                                                                                |
|                                                           | Theory and Computation                                                                                                                                                                                                                                                                                                                                                                        |

Arthur Gretton, Kenji Fukumizu, Choon Teo, Le Song, Bern593 hard Schölkopf, and Alex Smola. A kernel statistical test 594 of independence. In Advances in Neural Information 595 Processing Systems , 2007. 596

Arthur Gretton, Karsten M Borgwardt, Malte J Rasch, Bernhard Scholkopf, and Alexander Smola. A kernel twosample test. Journal of Machine Learning Research , 13: 723-773, 2012.

Kate Higgins, Maxim Ziatdinov, Sergei Kalinin, and Mahshid Ahmadi. High-throughput study of antisolvents on the stability of multicomponent metal halide perovskites through robotics-based synthesis and machine learning approaches. Journal of the American Chemical Society , 143(47), 2021.

Hisham Husain, Vu Nguyen, and Anton van den Hengel. Distributionally robust Bayesian optimization with φ -divergences. In Advances in Neural Information Processing Systems (NeurIPS) , 2023.

Hideaki Ishibashi, Masayuki Karasuyama, Ichiro Takeuchi, and Hideitsu Hino. A stopping criterion for Bayesian optimization by the gap of expected minimum simple regrets. In Proceedings of The International Conference on Artificial Intelligence and Statistics (AISTATS) , 2023.

Johannes Kirschner, Ilija Bogunovic, Stefanie Jegelka, and Andreas Krause. Distributionally Robust Bayesian Optimization. In Proceedings of the International Conference on Artificial Intelligence and Statistics (AISTATS) , 2020.

Ksenia Korovina, Sailun Xu, Kirthevasan Kandasamy, Willie Neiswanger, Barnabas Poczos, Jeff Schneider, and Eric Xing. ChemBO: Bayesian Optimization of Small Organic Molecules with Synthesizable Recommendations. In International Conference on Artificial Intelligence and Statistics (AISTATS) , 2020.

Andreas Krause and Cheng Ong. Contextual Gaussian Process Bandit Optimization. In Advances in Neural Information Processing Systems (NeurIPS) , 2011.

Eric Hans Lee, Valerio Perrone, Cedric Archambeau, and Matthias Seeger. Cost-aware Bayesian Optimization. arXiv preprint arXiv:2003.10870 , 2020.

Eric Hans Lee, David Eriksson, Valerio Perrone, and Matthias Seeger. A Nonmyopic Approach to CostConstrained Bayesian Optimization. In Proceedings of the Thirty-Seventh Conference on Uncertainty in Artificial Intelligence (UAI) , 2021.

Cheng Li, Sunil Gupta, Santu Rana, Vu Nguyen, Svetha Venkatesh, and Alistair Shilton. High dimensional Bayesian optimization using dropout. In Proceedings of the International Joint Conference on Artificial Intelligence (IJCAI) , 2017.

597

598

599

600

601

602

603

604

605

606

607

608

609

610

611

612

613

614

615

616

617

618

619

620

621

622

623

624

625

626

627

628

629

630

631

632

633

634

635

636

637

638

639

640

641

Sulin Liu, Qing Feng, David Eriksson, Benjamin Letham, 642 and Eytan Bakshy. Sparse Bayesian Optimization. In 643 Proceedings of the International Conference on Artificial 644 Intelligence and Statistics (AISTATS) , 2023. 645

Anastasia Makarova, Huibin Shen, Valerio Perrone, Aaron 646 Klein, Jean Baptiste Faddoul, Andreas Krause, Matthias 647 Seeger, and Cedric Archambeau. Automatic termination 648 for hyperparameter optimization. In Proceedings of the 649 First International Conference on Automated Machine 650 Learning , 2022. 651

Philip W. Nega, Zhi Li, Victor Ghosh, Janak Thapa, Shi652 jing Sun, Noor Titan Putri Hartono, Mansoor Ani Na653 jeeb Nellikkal, Alexander J. Norquist, Tonio Buonassisi, 654 Emory M. Chan, and Joshua Schrier. Using automated 655 serendipity to discover how trace water promotes and 656 inhibits lead halide perovskite crystal formation. Applied 657 Physics Letters , 119(4), 2021. 658

659

660

- C. Rasmussen and C. Williams. Gaussian Processes for Machine Learning . MIT Press, 2006.

Paul Rolland, Jonathan Scarlett, Ilija Bogunovic, and Volkan 661 Cevher. High-Dimensional Bayesian Optimization via 662 Additive Models with Overlapping Groups. In Proceed663 ings of the International Conference on Artificial Intelli664 gence and Statistics (AISTATS) , 2018. 665

- Artun Saday, Ya¸sar Cahit Yıldırım, and Cem Tekin. Robust 666 Bayesian satisficing. In Advances in Neural Information 667 Processing Systems (NeurIPS) , 2023. 668

669

670

671

672

Romelia Salomon Ferrer, David Case, and Ross Walker. An

overview of the Amber biomolecular simulation package.

Wiley Interdisciplinary Reviews: Computational Molecu-

lar Science

, 3, 2013.

Isaac Sebenius, Topi Paananen, and Aki Vehtari. Feature 673 Collapsing for Gaussian Process Variable Ranking. In 674 Proceedings of The International Conference on Artificial 675 Intelligence and Statistics (AISTATS) , 2022. 676

677

678

679

680

681

682

683

Yihang Shen and Carl Kingsford. Computationally Efficient High-Dimensional Bayesian Optimization via Variable Selection. arXiv preprint arXiv:2109.09264 , 2021.

I.M Sobol. Global sensitivity indices for nonlinear mathematical models and their Monte Carlo estimates. Mathematics and Computers in Simulation , 55(1):271-280, 2001.

Adrien Spagnol, Rodolphe Le Riche, and Sébastien Da 684 Veiga. Bayesian optimization in effective dimensions 685 via kernel-based sensitivity indices. In Proceedings of 686 the International Conference on Applications of Statistics 687 and Probability in Civil Engineering (ICASP) , 2019. 688

Niranjan Srinivas, Andreas Krause, Sham M. Kakade, 689 and Matthias W. Seeger. Information-Theoretic Regret 690 Bounds for Gaussian Process Optimization in the Bandit 691 Setting. IEEE Transactions on Information Theory , 58 692 (5):3250-3265, 2012. 693

Sebastian Shenghong Tay, Chuan Sheng Foo, Daisuke Urano, Richalynn Chiu Xian Leong, and Bryan Kian Hsiang Low. Bayesian optimization with costvarying variable subsets. In Advances in Neural Information Processing Systems (NeurIPS) , 2023.

Saul Toscano-Palmerin and Peter I. Frazier. Bayesian optimization with expensive integrands. SIAM Journal on Optimization , 32(2):417-444, 2022.

Theja Tulabandhula and Cynthia Rudin. Robust optimization using machine learning for uncertainty sets. arXiv preprint arXiv:1407.1097 , 2014.

Zi Wang, Chengtao Li, Stefanie Jegelka, and Pushmeet Kohli. Batched high-dimensional Bayesian optimization via structural kernel learning. In Proceedings of the 34th International Conference on Machine Learning (ICML) , 2017.

James T. Wilson, Riccardo Moriconi, Frank Hutter, and Marc Peter Deisenroth. The reparameterization trick for acquisition functions. arXiv preprint arXiv:1712.00424 , 2017.

Yichi Zhang, Daniel W Apley, and Wei Chen. Bayesian optimization for materials design with mixed quantitative and qualitative variables. Scientific Reports , 10(1):1-13, 2020.

Juliusz Ziomek and Haitham Bou-Ammar. Are Random Decompositions all we need in High Dimensional Bayesian Optimisation? In Proceedings of the International Conference on Machine Learning (ICML) , 2023.

694

695

696

697

698

699

700

701

702

703

704

705

706

707

708

709

710

711

712

713

714

715

716

717

718

719

720

721

## Learning Relevant Contextual Variables Within Bayesian Optimization (Supplementary Material)

*

Julien Martinelli

1

2

Ayush Bharti

Patrick Rinke

Armi Tiihonen

3

2

Louis Filstroff

2,5

Samuel Kaski

1 Inserm Bordeaux Population Health, Vaccine Research Institute, Université de Bordeaux, Inria Bordeaux Sud-ouest, France 2 Department of Computer Science, Aalto University, Helsinki, Finland

3 Department of Applied Physics, Aalto University, Helsinki, Finland

- 4 Univ. Lille, CNRS, Centrale Lille, UMR 9189 CRIStAL, F-59000 Lille, France

5 Department of Computer Science, University of Manchester, Manchester, United Kingdom

## APPENDIX

Outline. The Appendix is organized as follows. In Appendix A, we provide a flowchart summarizing the proposed method 722 SADCBO . In Appendix B, we provide further details about the phase switching criterion introduced in Section 3.2. In 723 Appendix C, we provide more details about one of the baselines used in the main text, based on maximum mean discrepancy. 724 Appendix D contains further experimental results regarding: 725

726

727

728

729

730

731

- · Phase switching time and sensitivity-based inclusion probabilities of contextual variables found by SADCBO for additional test functions (Figure S2).
- · Varying the number of irrelevant contextual variables (Figure S3).
- · Varying contextual variables query cost (Figure S4).
- · The distribution of phase switching times for SADCBO (Figure S5).
- · Varying SADCBO hyperparameters (Appendix D.1 and Figure S6).

Finally, Appendix E contains a description of the real-world experiments performed throughout the paper, along with the 732 analytical expressions of the synthetic examples used. 733

## A FLOWCHART OF THE ALGORITHM

Figure S1: Flowchart of the proposed method SADCBO .

picture-5.png

3

S. T. John

4

Sabina J. Sloman

5

## B PHASE SWITCHING CRITERION

The phase switching criterion we employ is derived from the stopping criterion from Ishibashi et al. [2023]. The absolute difference of expected minimum simple regrets ∆ R t := | R t -R t -1 | can be upper bounded with probability 1 -δ by ∆ ˜ R t , a quantity defined in Equation (11). Directly quoting the work of Ishibashi et al. [2023], the rationale behind this criterion reads as follows: 'By evaluating the difference between the expected minimum simple regrets, we can stop BO without knowing f ∗ , because it indicates that the search efficiency is low and there is almost no improvement in the objective value. However, it is generally difficult to calculate ∆ R t analytically'. Next, any stopping criterion involves the computation of some sort of threshold. Ishibashi et al. [2023] exploit the fact that their upper bound ∆ ˜ R t can itself be upper bounded by a quantity (introduced in [Ishibashi et al., 2023, Equation 10]), whose convergence speed to zero is limited by a specific term, s t (Equation 12). s t can be computed analytically and therefore yields an adaptive threshold.

Finally, Equation (11) involves a sequence κ δ,t -1 :

κ δ,t -1 = max v ∈D t -1 UCB δ ( v ) -max v ∈V LCB δ ( v ) , (S1)

where UCB δ ( v ) = µ t ( v |D t ) + β 1 / 2 t σ t ( v |D t ) and LCB δ ( v ) = µ t ( v |D t ) -β 1 / 2 t σ t ( v |D t ) . β 1 / 2 t is a trade-off parameter between exploration and exploitation that depends on δ [Srinivas et al., 2012]. κ δ,t -1 is a quantity that was first introduced by Makarova et al. [2022, Section 3.2] as an upper bound for the simple regret of the surrogate, which directly flows from the bounds provided by Srinivas et al. [2012] for well-calibrated surrogates.

734

735

736

737

738

739

740

741

742

743

744

745

746

747

Heuristically, one can think of our setting as applying the stopping criterion to x ↦→ f ( x , z ) , a stochastic black-box function 748 with z ∼ p ( z ) . Upon satisfaction of this criterion, we switch to the optimization of ( x , z ) ↦→ f ( x , z ) where some contextual 749 variables are optimized, and some others are still sampled from p ( z ( j ) ) . 750

## C MAXIMUMMEANDISCREPANCY-BASED VARIABLE SELECTION

Spagnol et al. [2019] introduced a BO algorithm with a variable selection procedure based on the Hilbert Schmidt Independence Criterion (HSIC). This measure can be used in our setting as well. We now briefly describe how it is defined.

As introduced in the main text, let Z ⊂ R c be the space of contextual variables, and H be a Hilbert space of R -valued functions on Z . Assume that k : Z × Z → R is the unique positive definite kernel associated with the Reproducing Kernel Hilbert Space H . Let µ P Z be the kernel mean embedding of the distribution P Z , µ P Z := E Z [ k ( Z, · )] = ∫ Z k ( z , · )d P Z . Kernel embeddings of probability measures provide a distance between distributions between their embeddings in the Hilbert Space H , named Maximum Mean Discrepancy (MMD, [Gretton et al., 2012]):

MMD ( P Z , P Y ) = ∥ µ P Z -µ P Y ∥ 2 H . (S2)

For two random variables Z ∼ P Z on H and Y ∼ P Y on G , the HSIC is the squared MMD between the product distribution P ZY and the product of its marginals P Z P Y ,

HSIC ( Z, Y ) = MMD 2 ( P ZY , P Z P Y ) (S3) = ∥ µ P ZY -µ P Z P Y ∥ 2 H⊗G (S4) = E Z,Y E Z ' ,Y ' [ k ( Z, Z ' ) l ( Y, Y ' )] (S5) + E Z E Y E Z ' E Y ' [ k ( Z, Z ' ) l ( Y, Y ' )] 2 E Z,Y E Z E Y [ k ( Z, Z ' ) l ( Y, Y ' )] .

-' '

To determine the relevance of a variable Z ( i ) , Spagnol et al. [2019] introduce

S HSIC ( Z ( i ) ) = HSIC ( Z ( i ) , I ( Z ∈ L γ )) , (S6)

with L γ a region of interest: the locations where the objective function value is above a threshold γ . This measure reflects how important Z ( i ) is to reach L γ .

We implemented this measure, substituting expectations for empirical means over the dataset D . We use γ = 0 . 8 , a threshold identical to the one used for SADCBO in Equation (6). The kernel k is chosen to be a RBF kernel, and l is a linear kernel l ( y, y ' ) = yy ' , a common choice for binary data.

751

752

753

754

755

756

757

758

759

760

761

762

763

## D ADDITIONAL EXPERIMENTAL RESULTS

Molecular

Figure S2: Each row deals with a specific problem. The left panel shows the BO trial results for each baseline. The middle and right panel show statistics related to SADCBO : 1) the phase switching time after which phase II begins and 2) the inclusion probabilities for each contextual variable. Statistics are computed across N = 100 BO trials with different seeds.

picture-6.png

found

alue

v

Best

Figure S3: Varying the number of irrelevant contextual variables. For any variable, the associated query cost is 1. p ( z ) = U ([0 , 1] c ) . On the three test functions Ackley5D, Hartmann6D and Hartmann4D, our approach outperforms other baselines even in high dimensions.

picture-7.png

found

alue

v

Best

Figure S4: Ablation study on contextual variable query cost. Design variables have cost 1. p ( z ) = U ([0 , 1] c ) .

picture-8.png

Figure S5: Distribution of phase switching criterion triggering times for SADCBO across N = 100 different BO trials. We consider the Ackley5D function with an increasingly larger ratio of relevant contextual variables over design variables, and 8 irrelevant contextual variables. p ( z ) = U ([0 , 1] c ) . For any variable, the associated query cost is 1. As the impact of contextual variables on the output function grows, the number of iterations spent in the observational phase grows as well.

picture-9.png

## D.1 ADDITIONAL DETAILS ON HYPERPARAMETER VARIATIONS.

Figure S6: Varying hyperparameters for SADCBO . For any variable, the associated query cost is 1. p ( z ) = U ([0 , 1] c ) . Top: varying η , the contextual variable inclusion threshold over the cumulative sum of sensitivity indices. Middle: varying γ , the threshold used in the creation of the truncated dataset D γ from Equation (6). Bottom: varying Q , the size of the dataset D Q from Equation (7). η is the most sensitive hyperparameter here.

picture-10.png

- We vary the 3 hyperparameters of SADCBO : η [0 , 1] the threshold based over the cumulative sum of sensitivity indices, 764
- which in turn regulates how many variables are selected every iteration; γ [0 , 1] , a threshold upon which a value is 765
- considered high enough to have its input added to dataset (Equation (6)), used for sensitivity analysis; and 766 Q (Equation (7)).
- ∈ ∈ D γ Q the size of
- the dataset D 767
- Figure S6 reports the performances. Unsurprisingly, η stands out as the most stringent parameter: as its value decreases, 768
- fewer variables are included, at which point not all relevant ones are selected, leading to reduced performances. Note that in 769
- a setting where there are no relevant contextual variables, lower values of η will actually lead to better performances.Then, 770
- varying γ ∈ [0 , 1] slightly affects the results: γ increasing means that more samples are collected for sensitivity analysis, but 771
- these are less relevant for producing a reliable set of variables accounting for the fluctuations at the optimum. Finally, for the 772
- examples considered, Q has only a limited effect, close to that of varying γ . This might stem from the fact that batched 773
- acquisition functions are notoriously difficult to optimize and may sometimes struggle to enforce diversity. 774

## E EXPERIMENT DETAILS

## E.1 REAL-WORLD DATASETS

Portfolio optimization dataset. This dataset was first introduced in [Cakmak et al., 2020]. The goal is to tune the hyper-parameters of a trading strategy so as to maximize return under risk-aversion to random environmental conditions. A software is used to simulate and optimize the evolution of a portfolio over a period of four years using open-source market data. Each evaluation of this simulator returns the average daily return over this period of time under the given combination of hyper-parameters and environmental conditions. Since the simulator is expensive to evaluate, we do not use it directly but perform pool-based Bayesian Optimization using a pool of 3000 points generated according to a Sobol sampling design.

The hyper-parameters to be optimized are the risk and trade aversion parameters and the holding cost multiplier. These variables constitute the design variables. The contextual variables are the bid-ask spread and the borrowing cost.

Yacht hydrodynamics dataset. This dataset comes from the UCI Machine Learning Repository [Gerritsma et al., 2013]. The optimization problem is to maximize the residuary resistance per unit weight of displacement of a yacht by controlling its 5-dimensional hull geometry coefficients. Another optimization variable is the 1-dimensional Froude number. We chose as design variables the first four dimensions of the hull geometry coefficients. The contextual variables are the last hull geometry dimension and the Froude number. Like the Portfolio optimization dataset, we have access to a limited number of samples ( ≈ 300) and thus perform pool-based Bayesian optimization.

775

776

777

778

779

780

781

782

783

784

785

786

787

788

Molecular structure optimization. This case is a computational chemistry challenge. Molecules can adopt different 789 structures that preserve the topology (bonds and bonding types), but have different internal angles. Finding such conformers 790 is a global optimization problem. Here, we are searching for the conformers of alanine - a molecule with structure 791 C 3 H 7 NO 2 -whose energy is calculated at each round of BO with the AMBER force field [Salomon Ferrer et al., 2013, 792 Case et al., 2023]. Alanine provides 33 structural variables to optimize: ten dihedral angles, eleven bond angles, and twelve 793 bond lengths. Conformer search in the full 33-dimensional space is very challenging, but progress has been made with 794 Bayesian optimization recently by reducing the problem to the four most important dihedral angles [Fang et al., 2021]. For 795 the example in this work, three of these four dihedral angles were chosen as the design variables (indices 3, 17, and 21 796 in the dataset; which denotes dihedral angles d4, d11, and d13 in AMBER notation; d4 is the bond leading to the amino 797 group, d13 the one leading to the hydroxyl group, and d11 is the bond between these two), the rest of the dihedral and bond 798 angles (18 angles) are chosen as the contextual variables, and the bond lengths are kept fixed to facilitate faster simulations. 799 The search space is selected by utilizing molecule domain knowledge in a conservative manner that allows 10-20 degree 800 variations for the bond angles and is free for the dihedral angles. To outline the alanine optimization results, the structure 801 optimization performed here as a test case is a high-dimensional problem, thus, the VBO method that tries to optimize all the 802 variables x and z converges slowly. Due to the same reason, methods MMDBO and Dropout also underperform in terms of 803 convergence for the alanine problem. Similarly to SADCBO , these two baselines operate variable selection, although using a 804 different selection criterion. However, controlling the selected variable comes at a cost. On the opposite, it turns out that 805 from Figure S2 (fifth row, middle panel), SADCBO virtually never switches to phase II for the Molecular Shape example. 806 Therefore, SADCBO does perform contextual variable selection, but does not control them, it only chooses which of these 807 variables will be included in the surrogate, hence behaving like CBO , but with a variable selection step. This explains why 1) 808 CBO closely follows SADCBO for this example and 2) why other variable selection baselines like MMDBO and Dropout 809 end up far from SADCBO . Interestingly, in this case, the simplified case of optimizing only the design variables x ( CUBO ) 810 also performs well. This is because our domain experts made good initial choices on the relevant design variables x and the 811 search spaces of context variables z . This type of pre-analysis is time-consuming and more challenging for larger molecules. 812 Hence, a future line of work is to test context-aware BO more comprehensively in molecule structure optimization tasks. 813

Robot Pushing Task This task was first introduced in Wang et al. [2017], and consists of a control parameter tuning problem for robot pushing. This real-world function returns the distance between a designated goal location and two objects being pushed by two robot hands, whose trajectory is determined by 14 parameters specifying the location, rotation, velocity and moving direction, among others. The function is implemented with a physics engine, the Box2D simulator. There are 6 design variables and 8 contextual variables.

814

815

816

817

818

## E.2 SYNTHETIC TEST FUNCTIONS

## Hartmann-6D function:

f

( v ) = -4 ∑ i =1 α i exp   -6 ∑ j =1 A ij ( v ( j ) -P ij )   α = (1 . 0 , 1 . 2 , 3 . 0 , 3 . 2) T A =     10 3 17 3 . 5 1 . 7 8 0 . 05 10 17 0 . 1 8 14 3 3 . 5 1 . 7 10 17 8 17 8 0 . 05 10 0 . 1 14     P = 10 -4     1312 1696 5569 124 8283 5886 2329 4135 8307 3736 1004 9991 2348 1451 3522 2883 3047 6650 4047 8828 8732 5743 1091 381    

defined over V = [0 , 1] 6 . The second, fifth, and sixth variables were considered as design variables, while the first, third, 819 and fourth variables were considered as contextual variables. 6 noise variables were added. Table S1 provides the results 820 of a Sobol global sensitivity analysis performed using evaluations of the function collected over a grid of N = 917504 821 samples [Sobol, 2001]. Adding up the first order indices for design and contextual variables separately leads to S x ≈ 0 . 124 822 and S z ≈ 0 . 196 . This means that with respect to first-order interactions, contextual variables have more impact than design 823 variables, in this synthetic example. One should notice however that these indices are computed across the whole search 824 space and not specifically at the optimum. 825

Table S1: Sobol global sensitivity analysis for the Hartmann-6D function using N = 917504 samples.

| Variable   |   First order sensitivity indices |   Total order sensitivity indices |
|------------|-----------------------------------|-----------------------------------|
| z (1)      |                             0.107 |                             0.343 |
| x (2)      |                             0.006 |                             0.399 |
| z (3)      |                             0.007 |                             0.052 |
| z (4)      |                             0.082 |                             0.379 |
| x (5)      |                             0.106 |                             0.297 |
| x (6)      |                             0.012 |                             0.482 |

## Hartmann-4D function:

f

( v ) = 1 0 . 839   1 . 1 -4 ∑ i =1 α i exp   -4 ∑ j =1 A ij ( v ( j ) -P ij )     α = (1 . 0 , 1 . 2 , 3 . 0 , 3 . 2) T A =     10 3 17 3 . 5 0 . 05 10 17 0 . 1 3 3 . 5 1 . 7 10 17 8 0 . 05 10     P = 10 -4     1312 1696 5569 124 2329 4135 8307 3736 2348 1451 3522 2883 4047 8828 8732 5743    

defined over V = [0 , 1] 4 . The first and fourth variables were considered as design variables, while the second and third 826 variables were considered as contextual variables. 3 noise variables were added. Table S2 provides the results of a Sobol 827

global sensitivity analysis performed using evaluations of the function collected over a grid of N = 300000 samples. Adding up the first order indices for design and contextual variables separately leads to S x ≈ 0 . 579 and S z ≈ 0 . 091 . This means that with respect to first-order interactions, design variables have much more impact on the output than contextual variables. The gap slightly reduces when considering total order sensitivity indices. However, it is worth remembering that these indices are computed across the whole search space and not specifically at the optimum.

Table S2: Sobol global sensitivity analysis for the Hartmann-4D function using N = 300000 samples.

| Variable   |   First order sensitivity indices |   Total order sensitivity indices |
|------------|-----------------------------------|-----------------------------------|
| x (1)      |                             0.307 |                             0.477 |
| z (2)      |                             0.037 |                             0.279 |
| z (3)      |                             0.054 |                             0.103 |
| x (4)      |                             0.272 |                             0.526 |

## Ackley 5D function:

f ( v ) = -20 exp   -0 . 2 √ √ √ √ 1 5 5 ∑ j =1 ( v ( j ) ) 2   -exp   1 5 5 ∑ j =1 cos(2 πv ( j ) )   +20 + e 1

defined over V = [ -5 , 5] 5 . 8 noise variables were added.

## EggHolder 2D function:

f ( v ) = -( v (2) +47)sin ( √ ∣ ∣ ∣ ∣ v (2) + v (1) 2 +47 ∣ ∣ ∣ ∣ ) -v (1) sin ( √ | v (1) -( v (2) +47) | )

defined over V = [ -512 , 512] 2 . The first variable was considered as a design variable, and the second one as a contextual 834 variable. 4 noise variables were added. A Sobol global sensitivity analysis performed using evaluations of the function 835 collected over a grid of N = 3000000 samples shows that both variables have a similar contribution to the output (Table S3). 836

Table S3: Sobol global sensitivity analysis for the EggHolder-2D function using N = 3000000 samples.

| Variable   |   First order sensitivity indices |   Total order sensitivity indices |
|------------|-----------------------------------|-----------------------------------|
| x (1)      |                            0.001  |                             0.998 |
| z (2)      |                            0.0004 |                             0.999 |

828

829

830

831

832

833