picture-1.png

picture-2.png

International Journal of Approximate Reasoning 31 (2002) 291-311

www.elsevier.com/locate/ijar

## Ant colony optimization for learning Bayesian networks

Luis M. de Campos a, * , Juan M. Fern /C19 a andez-Luna b , Jos /C19 e eA.G /C19 a amez c , Jos /C19 e e M. Puerta c

- a Departamento de Ciencias de la Computaci /C19 o o n e I.A., E.T.S. de Ingenieria Informatica, Universidad de Granada, Avenida Andalucia 38, 18071 Granada, Spain b Departamento de Inform /C19 a a tica, Universidad de Ja /C19 e e n, 23071 Ja /C19 e e n, Spain c Departamento de Inform /C19 a a tica, Universidad de Castilla-La Mancha, 02071 Albacete, Spain

Received 1 January 2002; accepted 1 July 2002

## Abstract

One important approach to learning Bayesian networks (BNs) from data uses a scoring metric to evaluate the fitness of any given candidate network for the data base, and applies a search procedure to explore the set of candidate networks. The most usual search methods are greedy hill climbing, either deterministic or stochastic, although other techniques have also been used. In this paper we propose a new algorithm for learning BNs based on a recently introduced metaheuristic, which has been successfully applied to solve a variety of combinatorial optimization problems: ant colony optimization (ACO). We describe all the elements necessary to tackle our learning problem using this metaheuristic, and experimentally compare the performance of our ACObased algorithm with other algorithms used in the literature. The experimental work is carried out using three different domains: ALARM, INSURANCE and BOBLO. /C211 2002 Elsevier Science Inc. All rights reserved.

Keywords: Bayesian networks; Learning; Ant colony optimization

E-mail addresses:

lci@decsai.ugr.es (L.M. de Campos), jmfluna@ujaen.es (J.M. Fern

/C19 a andez-

Luna), jgamez@info-ab.uclm.es (J.A. G

/C19

a

amez), jpuerta@info-ab.uclm.es (J.M. Puerta).

## 1. Introduction

Bayesian networks (BNs), also known as probabilistic belief networks or causal networks, are knowledge representation tools capable of efficiently manage the dependence/independence relationships among the random variables that compose the problem domain we wish to model. This representation has two components: (a) a graphical structure, or more precisely a directed acyclic graph (dag), and (b) a set of parameters, which together specify a joint probability distribution over the random variables [28,37]. In BNs, the graphical structure represents dependence and independence relationships. The parameters are a collection of conditional probability measures, which shape the relationships.

Once the BN has been specified, it constitutes an efficient device for the performance of inference tasks. However, there still remains the previous problem of building such a network. It is an important task, therefore, to develop automatic methods capable of learning the network directly from data, as an alternative or a complement to the method of eliciting conditional (in)dependence assertions from experts.

Nowadays, the problem of learning or estimating a BN from data is receiving increasing attention within the community of researchers into uncertainty in artificial intelligence. Algorithms for learning (the structure of) BNs have been studied, basically from two points of view: methods based on conditional independence tests [15,17,38,41] and methods based on a scoring metric optimization [13,26,30]. This classification is not exhaustive and/or strict, since there are also some algorithms that use a combination of these two methods [1,2,14,40]. In this paper we focus on learning methods based on a scoring metric.

Because learning BNs is, in general, a NP-hard problem [12] and exact methods become unfeasible, we have to solve the problem with heuristic methods. Most existing scoring-based learning algorithms apply standard heuristic search techniques, such as greedy hill climbing (HC), iterated local search (ILS), simulated annealing, etc. In this paper we propose a new scoringbased learning method that uses a recently introduced metaheuristic for combinatorial optimization: ant colony optimization (ACO) [23]. The algorithms based on ACO were initially used to solve specific problems: the ant system, for example, was successfully applied to the traveling salesman problem (TSP) [22], whose search space is the set of permutations of the nodes in a graph. Applications to shortest path problems in graphs were developed in order to study the behavior of these algorithms on simple problems, but they later gave rise to an optimization metaheuristic that can be applied to combinatorial optimization problems which may be represented in the form of a graph [20].

The paper is structured as follows: we begin in Section 2 with the preliminaries, where we briefly describe the concepts and methods related to BNs and ACO which we require for our discussion in the latter part of the paper. In Section 3 we develop an algorithm for learning BNs based on ACO. In Section 4 we present the experimental evaluation of our algorithm. Finally, Section 5 contains the concluding remarks.

## 2. Preliminaries

In this section we briefly review some basic concepts related to BNs and how to learn them, as well as other concepts related to ACO.

## 2.1. Learning Bayesian networks

A BN is a directed acyclic graph G ¼ð V ; E Þ , where the set of nodes V ¼ f x 1 ; x 2 ; ... ; xn g represents the system variables and E , a set of arcs, represents the direct dependence relationships among the variables. A set of parameters is also stored for each variable in V , which are usually conditional probability distributions. For each variable xi 2 V we have a family of conditional distributions P ð xi j Pa ð xi ÞÞ , where Pa ð xi Þ represents the parent set of the variable xi . From these conditional distributions we can recover the joint distribution over V :

Y

P ð x 1 ; x 2 ; ... ; xn Þ¼ n i ¼ 1 P ð xi j Pa ð xi ÞÞ ð 1 Þ

This expression represents a decomposition of the joint distribution. The dependence/independence relationships which make this decomposition possible are graphically encoded (through the d-separation criterion [37]) by means of the presence or absence of direct connections between pairs of variables.

The problem of learning a BN can be stated as follows: given a training set D ¼f v 1 ; ... ; v m g of instances of V , find the BN that best matches D . The common approach to this problem is to introduce a scoring function, f , that evaluates each network with respect to the training data, and then to search for the best network according to this score. Different Bayesian and non-Bayesian scoring metrics can be used [2,10,13,26,30].

A desirable and important property of a metric is its decomposability in the presence of full data, i.e., the scoring function can be decomposed in the following way:

X

f ð G : D Þ¼ n i ¼ 1 f ð xi ; Pa ð xi Þ : Nx i ; Pa ð xi ÞÞ ð 2 Þ

where Nx i ; Pa ð xi Þ are the statistics of the variable xi and Pa ð xi Þ in D , i.e., the number of instances in D that match each possible instantiation of xi and Pa ð xi Þ . The decomposition of the metric is very important for the learning task: a local search procedure that changes one arc at each move can efficiently evaluate the improvement obtained by this change, because it can reuse most of the computations made in previous stages (only the statistics corresponding to the variables whose parent sets have been modified need to be recomputed). One example is a greedy HC method that at each step performs the local change yielding the maximal gain, until it reaches a local maximum of the scoring function. The usual choices for local changes in the space of dags are arc addition, arc deletion and arc reversal [26]. As this procedure is trapped in the first local maximum it reaches, several methods for avoiding this situation have been used, such as stochastic HC (with random restart [26]), variable neighborhood search [18], genetic algorithms (GAs) [31,34], simulated annealing [11], Tabu search [8], etc.

We will now review the algorithm B and the K2 metric, because the former will be used in our ACO based learning algorithm and the latter will be the scoring function used in our experiments.

## 2.1.1. The algorithm B

Algorithm B [9] is a greedy construction heuristic. It starts with an empty dag (arc-less structure) and at each step it adds the arc with the maximum increase in the (decomposable) scoring metric f , but avoiding the inclusion of directed cycles in the graph. The algorithm stops when adding any valid arc does not increase the value of the metric. An outline of algorithm B is given in Fig. 1.

In this algorithm A ½ i ; j /C138 is an adjacency matrix that stores the difference f ð xi ; Pa ð xi Þ[f xj gÞ /C0 f ð xi ; Pa ð xi ÞÞ , i.e., the gain obtained by inserting the arc xj ! xi in the graph. The arcs whose inclusion in the graph would generate a directed cycle are identified by assigning to A ½ i ; j /C138 the value /C01 . At each step, after inserting a valid arc xj ! xi , the algorithm identifies the new forbidden arcs by searching for the ancestors and descendants of xi . Then, as the value f ð xi ; Pa ð xi ÞÞ has been modified, the algorithm recomputes the new values of A ½ i ; k /C138 for any valid arc xk ! xi . The computational complexity for this update is O ð n 2 Þ .

## 2.1.2. The K2 metric

The K2 algorithm [13] is perhaps the best known of the algorithms for learning BNs, and has been the basis of much subsequent work. This algorithm uses a Bayesian scoring metric, which measures the joint probability of a BN G and a database D . The metric has adopted the name of the algorithm, so that it is referred to as the K2 metric, whose expression is:

Fig. 1. Structure of algorithm B.

Y

Y

Y

P ð G ; D Þ¼ P ð G Þ n i ¼ 1 qi j ¼ 1 ð ri /C0 1 Þ ! ð Nij þ ri /C0 1 Þ ! ri k ¼ 1 Nijk !

where ri is the number of possible values of the variable xi , qi is the number of possible configurations (instantiations) for the variables in Pa ð xi Þ , Nijk is the number of cases in D in which variable xi has its k th value and Pa ð xi Þ is instantiated to its j th value, and Nij ¼ P ri k ¼ 1 Nijk .

Assuming an uniform prior for P ð G Þ and using log ð P ð G ; D ÞÞ instead of P ð G ; D Þ , we get a decomposable metric:

X

f K2 ð G : D Þ¼ n i ¼ 1 f K2 ð xi ; Pa ð xi Þ : Nx i ; Pa ð xi ÞÞ ð 3 Þ !

/C18

/C19

f K2 ð xi ; Pa ð xi Þ : Nx i ; Pa ð xi ÞÞ¼ X qi j ¼ 1 log ð ri /C0 1 Þ ! ð Nij þ ri /C0 1 Þ ! þ X ri k ¼ 1 log ð Nijk ! Þ ð 4 Þ

## 2.2. Optimization based on ant colonies

Ant algorithms [20-23] are based on the cooperative behavior of real ant colonies, which are able to find the shortest path from a food source to their

nest. While walking, real ants deposit a chemical substance called pheromone on the ground. Ants can smell pheromone and, when choosing their way, they tend to choose, in a probabilistic way, paths marked by strong pheromone concentrations. In the absence of pheromone, ants choose randomly, but after a transitory period shortest paths will be more frequently visited and pheromone will accumulate faster on them, which in turn causes more ants to use these paths. This positive feedback effect means that all the ants will eventually use the shortest path. So, although a single ant is capable of building a solution (i.e., a path), the optimal solution comes about solely as a result of the cooperative behavior of the ant colony (which is based on a simple form of indirect communication through the pheromone, called stigmergy ). Although the first ACO algorithm, called Ant System, was applied to solve the TSP problem, a large number of applications to other problems were proposed after the introduction of ant system. Recently, the ACO metaheuristic was proposed as a common framework for existing applications [20,21].

Each ant builds a possible solution to the problem by moving through a finite sequence of neighbor states (nodes). Moves are selected by applying a stochastic local search directed by the ant internal state, problem-specific local information and the shared information about the pheromone.

Pheromone is modeled by means of a matrix s , where s ij contains the level of pheromone deposited in the arc from node i to node j . In the first ant systems, an ant k in node i will select the next node j to visit with probability:

8

pk ð i ; j Þ¼ ½ s ij /C138 a ½ g ij /C138 b P u 2 J k ð i Þ ½ s iu /C138 a ½ g iu /C138 b if j 2 Jk ð i Þ 0 otherwise < : ð 5 Þ

where g ij represents heuristic information about the problem; Jk ð i Þ is the set of neighbor nodes of node i that have not yet been visited by the ant k ; a and b are two parameters that determine the relative importance of the pheromone with respect to the heuristic information. For example, in the TSP, g ij ¼ 1 = dij , dij being the distance between cities i and j .

A different transition rule (for complex models) [22] introduces another parameter, as a trade-off between exploitation and exploration . The next node j to visit is chosen as

/C26

j ¼ arg max u 2 Jk ð i Þf½ s iu /C138½ g iu /C138 b g if q 6 q 0 J if q > q 0 ð 6 Þ

where q is a random number uniformly distributed in ½ 0 ; 1 /C138 ; q 0 is the parameter that determines the relative importance of exploitation versus exploration (0 6 q 0 < 1); J 2 Jk ð i Þ is a node randomly selected according to the probabilities in eq. (5), with a ¼ 1.

At each iteration of the algorithm each ant, using the previous transition rule, progressively builds a solution (path). The matrix of pheromone is updated in the following way:

- · Global updating: Only the ant which constructed the best solution reinforces the level of pheromone in the arcs that are part of the best solution, S þ ,obtained so far. This directs the search in the neighborhood of the best solution. The global updating rule is:

s ij ð 1 /C0 q Þ s ij þ q D s ij ð 7 Þ

where

/C26

D s ij ¼ 1 C ð S þ Þ if f ij g2 S þ s ij if f ij g 62 S þ

q (0 < q 6 1) is a parameter that controls the pheromone evaporation (decay) and C ð S þ Þ is the cost associated the best solution. Note with that the expression above implies that only the arcs belonging to the current best solution are reinforced.

- · Local updating: While building a solution, if an ant carries out the transition from node i to node j , then the pheromone level of the corresponding arc is changed in the following way:

s ij ð 1 /C0 w Þ s ij þ ws 0 ð 8 Þ

where s 0 is the initial pheromone level and 0 < w 6 1. Every time an ant uses an arc it looses some of its pheromone, making the arc less desirable. This rule favors the exploration of other arcs, thus avoiding premature convergence; without local updating all the ants would search in the neighborhood of the best solution found so far.

Another improvement included in the Ant colony system algorithm with respect to previous ant systems is the use of a local optimizer: some or all of the solutions obtained by the ants are locally optimized by using a local search method. This technique is particularly useful for many combinatorial optimization problems, where in practice best results are obtained when coupling ACO algorithms with local optimizers.

## 3. Learning Bayesian networks using ant colony optimization

In this section we develop a scoring-based learning algorithm for BNs that uses any given decomposable metric, the search method being based on ACO.

To apply the ACO metaheuristic to our problem, we have to define the following components:

- · An appropriate representation of the problem, which allows the incremental construction of possible solutions, using a probabilistic transition rule to move from one state i to a neighboring state j .
- · The heuristic information that will represent the problem-specific knowledge used by the search process to move from state i to state j , g ij .
- · The rule(s) to update the pheromone matrix s .
- · The probabilistic transition rule that uses the heuristic g and the pheromone s .
- ·
- The local optimizer.
- Now, let us define all these components for our learning problem:
- · Representation of the problem : In our case, the representation of the problem is a graph where the states of the problem are dags with n nodes. Thus, a state Gh will be a graph with the nodes xi 2 V and exactly h arcs and no directed cycle. The ant incremental construction of the solution starts from the empty graph G 0 (arcs-less dag) and proceeds by adding an arc xj ! xi to the current state Gh , i.e., Gh þ 1 ¼ Gh [f xj ! xi g . The final solution will be the state Gh in which the ant decides to stop the construction phase. Fig. 2 illustrates this process.
- · Heuristic information : The selected heuristic is to include in the graph the arc producing the greatest increase in the selected decomposable metric f . Therefore, we define

g ij ¼ f ð xi ; Pa ð xi Þ[f xj gÞ /C0 f ð xi ; Pa ð xi ÞÞ ð 9 Þ

Note that this is the way in which the algorithm B proceeds. Note also that this heuristic information is not static, i.e., it is not the same for all the ants (as happens, e.g., in the TSP, where the distance between cities is fixed). In our case the heuristic also depends on the ant /C213 s internal state, i.e., the current graph representing the partial solution of the problem, which determines the identity of the sets Pa ð xi Þ .

- · Pheromone updating rules : The global and local updating rules considered are the same as previously described, Eqs. (7) and (8), where s ij is the level

Fig. 2. Transition graph for Ant B.

picture-3.png

of pheromone in the arc xj ! xi , G þ is the best graph found so far and D s ij ¼ 1 = j f ð G þ : D Þj ,if xj ! xi 2 G þ and D s ij ¼ s ij otherwise; the initial level of pheromone is s 0 ¼ 1 = n j f ð G K2SN : D Þj , where n is the number of variables and G K2SN is the network obtained by the K2SN heuristic [18]. Observe that we are using the absolute value, j/C1j , in the expressions for D s ij and s 0. The reason is that the values of f ð/C1Þ are negative, because we always use the logarithmic version of the metric.

- · Probabilistic transition rule : The next arc to be included in the current graph, G , by an ant is selected in a way similar to that used by algorithm B, but using a stochastic decision rule (instead of a deterministic rule) that also takes into account the pheromone deposited at each arc (thus obtaining expressions similar to Eqs. (6) and (5)):

(

n

o

Select xl ! xr such that r ; l ¼ arg max i ; j 2 FG ½ s ij /C138½ g ij /C138 b if q 6 q 0 I ; J if q > q 0 ð 10 Þ

where I , J are two nodes randomly selected according to the following probabilities:

Fig. 3. Structure of Ant B.

300

pk ð i ; j Þ¼ ½ s ij /C138½ g ij /C138 b P u ; v 2 F G ½ s uv /C138 a ½ g uv /C138 b if i ; j 2 FG 0 otherwise < : ð 11 Þ

The set FG contains all the arcs which are still candidates for insertion in G (i.e., they do not belong to G , their inclusion in G does not create a directed cycle and g ij > 0).

- · Local optimizer : In this case we use a HC algorithm with the standard operators of arc addition, arc deletion and arc reversal (HCST) [26]. This algorithm is a greedy best-improvement where, at each step, the best move according to the metric and operators used is selected. The complexity for these moves is O ð n 2 Þ ( n is the number of variables). We should note that if we use a decomposable metric, a large number of computations can be reused from the previous stages of the algorithm. We should also note that the transition operators chosen for HCST contain the one chosen for Ant B.

## 1 Initialization:

- Obtain GK2SN
- To
- for all
- G+ GK2SN
- @rc (i,j) do: To Tij
- Set number of iterations for doing local search */ tstep
- 2. Loop: /* iterations; m ants tmaz
- (a) For t = 1 to do: tmaz
- for k = 1 to m do:
- A. Gk Ant B() /* see Figure 3
- B. If (t mod 0) then Gk = HillClimbing(Gk) tstep
- iii. If f (G6 D) > f(G+ D) then G+ = G6
- ii. = arg maxk:l..m f (Gk D)
- iu. Perform global pheromone update; eq. (7), using f (G+ : D)

## 3 Local optimization:

- (a)
- f (G8 D) () If f (Gb : D) 2 f(G+ : D) then G+ = G6
- for k = 1 to m do: GR = HillClimbing(Gk)

## 4. Return G+

Fig. 4. Description of the ACO-B learning algorithm.

Therefore, once an ant has obtained a solution, then by deleting or reversing an arc, the HCST algorithm can escape from an eventual local optima reached by the ant.

The local optimizer has been used in the very last iteration or every 10 iterations, where the result obtained by each ant is the starting point of the corresponding local search.

Fig. 3 shows the steps followed by an ant in our system to build a solution. Fig. 4 displays the overall process.

## 4. Experimental evaluation

## 4.1. Databases and algorithms

In order to test the behavior of the method proposed in the paper, three problems (domains) have been selected: ALARM [5], INSURANCE [6] and BOBLO [39]. The ALARM network has 37 nodes and 46 arcs and is used for diagnosis in a medical domain. It has been considered to be a benchmark for evaluating learning algorithms. All the experiments with ALARM have been carried out on the first 3000 cases of the ALARM database (which contains 20,000 cases, generated by probabilistic logic sampling [27]). The INSURANCE network, which contains 27 variables and 52 arcs, is a network for evaluating car insurance risks. The BOBLO network is a system which helps in the verification of the parentage of Jersey cattle through blood type identification, and contains 23 variables and 24 arcs. The experiments with INSURANCE use three databases containing 10,000 cases each and those for BOBLO use a database containing 5000 cases, all of them also generated by probabilistic logic sampling.

We have carried out an empirical comparison of the proposed learning algorithm (ACO-B) with two variants (ACO-B1, where the local optimizer is fired in the last iteration and ACO-B2, where the local optimizer is fired every 10 iterations and also in the last one) and another three algorithms using different optimization techniques: HC, ILS, which couples a HC until a local optimum is reached with a random transformation of that local optimum to be used as the starting point for the next HC phase, and finally estimation of distribution algorithms (EDAs). Below we give a brief description of EDAs. In all the cases the scoring metric used to guide the search is the K2 metric.

## 4.1.1. Estimation of distribution algorithms

EDAs [32] are a recent paradigm of evolutionary computation. EDAs are populational-based evolutionary algorithms, but differ from the well-known GAs [35] in the way the next population ( Pt þ 1) is generated. The key point in an

evolutionary algorithm is the evolution of a population of chromosomes/ individuals along time. Each chromosome in the population is a potential solution. In our case, in a n -dimensional domain, a chromosome will be a bidimensional array C of order n /C2 n , such that, /C26

C ½ i ; j /C138¼ 1 if xi ! xj is in the graph 0 otherwise

Of course, we must be careful to avoid directed cycles in the chromosomes.

In EDAs there are neither crossover nor mutation. Instead of these operators, a probabilistic model is induced from some of the individuals in population Pt , and then Pt þ 1 is obtained by sampling this probabilistic model. In this work we only consider the most simple version of EDAs, in which no dependence relation is considered among the variables (positions in the chromosome). Therefore, the joint probability distribution can be factorized as the product of marginal probabilities.

Two algorithms will be considered: UMDA [36] and PBIL [4]. The main difference between these two algorithms is that in UMDA the new probabilistic model replaces the old one, while in PBIL the new model is used to refine the old one by means of a parameter a .

We have used the following strategy in order to avoid the generation of directed cycles in the individuals sampled from the learned model: If Pi ; j ð 0 Þ and Pi ; j ð 1 Þ are the estimated probabilities of an arc xi ! xj being absent or present, respectively, we replace them by Pi ; j ð 0 Þ¼ 1 : 0and Pi ; j ð 1 Þ¼ 0 : 0, when the introduction of this arc induces a directed cycle in the graph. Notice that, if sampling is carried out by visiting the positions of the chromosome in a lexicographical way, i.e., ð 1 ; 1 Þ , ð 1 ; 2 Þ , ... , ð n /C0 1 ; n Þ , ð n ; n Þ , then the positions (arcs) sampled as 1 processed earlier have a greater probability of actually having value 1 in the sampled configuration than the positions considered later. For this reason, we manage this enumeration as a circular list, and each time sampling is carried out a new starting point is randomly generated.

The whole process of obtaining Pt þ 1 from Pt is as follows: (1) Select the best populationSize = 2 individuals from Pt ; (2) learn a probabilistic model by using the selected individuals as training data; (3) if UMDA replace the current model by the new one, else (PBIL) refine the current model by using the new one; (4) sample a new population P aux from the probabilistic model; and (5) get Pt þ 1 as the populationSize best individuals contained in Pt [ P aux.

## 4.2. Parameter settings

- · The ACO-B algorithms have been used with the following parameters in all the cases: q ¼ w ¼ 0 : 4, b ¼ 2 : 0, q 0 ¼ 0 : 8, m ¼ 10 ants and t max ¼ 100 iterations. These parameters have not been fitted by preliminary experimentation, and are similar to the ones used for other problems [22,25].

- · The HC (HCST) search uses the already mentioned operators of addition, deletion and reversal of arcs. The initialization of the search is carried out by using an empty network. This is the same local optimizer that the ACO-B algorithm uses.
- · The ILS algorithm uses a previously fixed number of random local perturbation of each local optimum reached by the HCST algorithm (avoiding directed cycles). This number has been fixed at 125, 1 together with an upper bound for the number of parents of each variable in the perturbed graph equal to 8. 2 The maximum number of iterations in this case has been fixed at 15.
- · For EDAs we have used the same parameters as in [7]: the initial population is generated randomly; population size is calculated as 10 n , n being the number of variables in the domain. In PBIL, a ¼ 0 : 5. In UMDA and PBIL the best half individuals contained in the population are used to induce the new probabilistic model (UMDA) or to refine the current probabilistic model (PBIL). The algorithms stop before carrying out the maximum number of generations (600) if the sum of the fitness in Pt þ 1 is the same as in Pt . We have also used the HCST algorithm with the best individual found in the last iteration as the starting point.

## 4.3. Performance measures

To evaluate the quality of the different algorithms, we have calculated several performance measures, some measuring the quality of the results and others measuring the complexity of the algorithms:

- · Measures to evaluate the quality of the learned networks:
- /C14 The value of the K2 metric (log version), Eq. (4).
- /C14 The KL value, defined as follows:

X

KL ð G : D Þ¼ n i ¼ 1 ; Pa ð xi Þ6¼; Dep ð xi ; Pa ð xi ÞÞ ð 12 Þ

where Dep ð/C1 ; /C1Þ is the measure of mutual information. Note that KL ð G : D Þ is a decreasing monotonic transformation of the Kullback distance [29] between the probability distribution associated with the database and the probability distribution associated with the network G [15,30]. We use this transformation because it can be calculated very

- efficiently, whereas the computation of the Kullback distance has an exponential complexity. The interpretation of KL ð G : D Þ is: the higher this parameter the better the network.
- /C14 Structural differences between the learned and the original network: the number of arcs added ( A ), deleted ( D ) and inverted 3 ( I ), compared with the original network.
- · Measures to evaluate the complexity of the algorithms:
- /C14 The number of the iterations, It , where the best individual was found. The subsequent iterations do not improve the results.
- /C14 TEst represents the total number of statistics Nijk evaluated during the learning process.
- /C14 The value TEst is not necessarily equal to the number of statistics truly computed from the data , 4 since we can use hashing techniques to avoid the necessity of recomputing previously calculated values, thus reducing substantially this number. Therefore, we also show the number of different statistics used, DEst .
- /C14 As the complexity of the computation of the statistics grows exponentially with the number of variables involved, we also show the average number of variables appearing in the different statistics evaluated, NVars . For comparative purposes, a raw estimation of the running time employed by an algorithm 5 is DEst /C2 2 NVars .

In addition to the previously described measures, we have also considered, for the ACO-B and the EDAs algorithms, the value of the K2 metric attained before using the local optimizer, K2noHC .

## 4.4. Experimental results and analysis

The results of our experimentation are displayed in Tables 1-3. In these tables l /C6 r indicates the mean and the standard deviation over the executions carried out. We have carried out 10 executions of each algorithm and for each domain considered. The value inside ð/C1Þ is the best result found along the experimentation by using the corresponding algorithm. We should note that the parameters shown in tables for the best BN are those corresponding to the best K2 value found. Notice that HCST is a deterministic algorithm, so only one execution was carried out.

L.M. de Campos et al. / Internat. J. Approx. Reason. 31 (2002) 291-311

305

t)

(bes

HCST

5.62

14,42

)

)

)

(

27.10

/C6

14,431.22

)

)

)

14,413.01

3375

)

þ

(

/C6

12.00

389,1

)

2.99

)

þ

(*

0.04

/C6

5.14

)

)

Table

| 12.61 ( ( )   | 12.61 ( ( )   | 12.61 ( ( )                                        | 12.61 ( ( )                                    |
|---------------|---------------|----------------------------------------------------|------------------------------------------------|
|               | /C6           |                                                    | 40.59 /C6                                      |
|               | 14,43 1.87    | ) 0.005 0.87 (7) 1.04 (2)                          | 1.91 (8) 50.04 1.85 /C6 13.80 0.43 /C6         |
|               | UMD A ( ) ) ) | ( ) 14,420.17 9.230 /C6 (9 .230) 7.80 /C6 2.10 /C6 | 7.50 /C6 469.5 0 /C6 ) 14,50 ( ) ) 275,3 12,03 |
|               | 0.72          |                                                    | 0.72                                           |
|               | 2 1.83 /C6    | 1.29) 0.001 0.70 (2) 0.00 (1)                      | 0.00 (0) 16.35 1.83 /C6 /C6                    |
|               |               | 14,40 /C6 ) /C6 /C6                                | /C6 /C6 3.20                                   |
|               | ACO-B 14,40   | ) 9.231 (9.231 2.10                                | 0.00 49.20 14,40 44,69 1208.56                 |
|               | )             | ( 1.00                                             | )                                              |
|               | /C6 4.54      | (2)                                                |                                                |
|               | 14,406.06     | 1.29) 0.003 0.30 (1)                               | (0) 20.68 /C6 5.01 9.60 /C6                    |
|               | ACO-B 1       | 14,40 /C6 ) /C6 1.79 /C6                           | /C6 1.90 /C6 14,407.32 43,34 7                 |
|               | ( ) ) )       | ( ) 9.231 (9.231 3.30 1.10                         | 1.70 87.10 ) ( þ ) 953.8                       |

6.64

/C6

3.20

14,41

)

)

)

(

K2

0.06

/C6

5.40

)

)

(*

rs

NVa

1

ALARM

for

Results

t)

(bes

r

/C6

l

ALARM

PBIL

ILS

)

14,403.28

)

(

9.220

0.006

/C6

9.229

0.006

/C6

9.230

KL

)

(9.232

.230)

(9

6

)

(5

2.07

/C6

6.90

(2)

2.25

/C6

5.80

A

4

)

(1

1.02

/C6

1.60

(1)

0.99

/C6

2.10

D

3

)

(5

1.76

/C6

6.10

(0)

2.67

/C6

2.30

I

1

2

100.8

/C6

458.50

5.18

/C6

7.80

It

-

43.49

/C6

14,475.13

-

HC

K2no

/C6

1.60

44,38

DEst

4.67

12,12

.15

1116

37

154,6

/C6

5

69.76E0

/C6

5

E0

31.16

TEst

E05

10.89

E03

97.90

306

L.M. de Campos et al. / Internat. J. Approx. Reason. 31 (2002) 291-311

st)

(Be

T

HCS

8.10

57,99

)

)

)

(

54.83

/C6

57,940.79

)

36.95

3.09

)

þ

(*

0.02

/C6

4.88

)

)

Table

| ( ) ( ) )   | ( ) ( ) )         | ( ) ( ) )                                             | ( ) ( ) )                                                      |
|-------------|-------------------|-------------------------------------------------------|----------------------------------------------------------------|
|             |                   | )                                                     |                                                                |
|             | 57,984.19 /C6     | 57,806.78 0.038 (4) 2.30 (8)                          | (1) 84.89 /C6 135.71 /C6                                       |
|             | UMD A ( ) ) )     | 130.66 ( ) 8.436 /C6 (8.409 ) 8.07 /C6 3.44 10.67 /C6 | 5.43 /C6 3.51 378.9 3 /C6 ) 58,05 6.38 ( ) ) 87,575.23 6157.88 |
|             | /C6 36.90         | ) 0.033 (3)                                           | 1.45 (8) (2) 24.63 /C6 37.21 8.63 /C6                          |
|             | 8.49              | 57,763.07 /C6 2.49                                    | 3.46 /C6 8.61 29,49 4                                          |
|             | O-B2 57,80        | .487) /C6 /C6                                         | /C6 57,80                                                      |
|             | A C )             | ( ) 8.450 (8 3.87 8.57                                | 3.13 48.73 ) ( ) ) 993.6                                       |
|             | 36.68             |                                                       |                                                                |
|             | /C6               | 3.07) /C6 0.036 ) /C6 1.99 (3)                        | 1.34 (8) (2) 21.57 /C6 37.27 /C6                               |
|             | ACO-B 1 57,807.81 | 57,76 (8.487 /C6                                      | /C6 2.09 /C6 57,811.04 28,738.77 3                             |
|             | )                 | ( ) 8.450 3.60 8.50                                   | 2.43 80.20 ) 882.9                                             |

/C6

8.90

57,91

)

)

)

(

K2

/C6

0.56

27,61

)

þ

(

DEst

0.10

/C6

6.06

)

)

(*

rs

NVa

2050.33

)

þ

(

/C6

3

126,897.6

2

URANCE

INS

for

Results

(best)

r

/C6

l

R-

INSU

PBIL

ILS

E

ANC

4.59)

57,83

2.20)

57,87

)

(

8.423

0.033

/C6

8.440

0.036

/C6

8.435

KL

)

(8.412

)

(8.400

10.33

(3)

2.59

/C6

7.60

(7)

3.11

/C6

8.78

A

11.67

(8)

2.00

/C6

10.83

)

(9

2.19

/C6

11.44

D

7.67

(1)

2.58

/C6

6.37

(3)

3.54

/C6

6.44

I

1

65.92

/C6

7

444.2

4.94

/C6

8.78

It

-

72.60

/C6

3.18

58,00

-

HC

K2no

5331.41

6

841.2

04

þ

7.66E

/C6

E05

34.77

/C6

5

14.11E0

TEst

4

52.79E0

E03

33.14

L.M. de Campos et al. / Internat. J. Approx. Reason. 31 (2002) 291-311

307

st)

(Be

T

HCS

11,927.00

)

)

)

(

8.71

/C6

11,898.92

)

14.91

2.90

)

þ

(*

0.02

/C6

4.73

)

)

| ( ) ( ) ) )   | ( ) ( ) ) )        | ( ) ( ) ) )                                              | ( ) ( ) ) )                                            |
|---------------|--------------------|----------------------------------------------------------|--------------------------------------------------------|
|               | 11,900.11 /C6 6.95 | 2.00) 0.001 (9) (1)                                      | (2) 81.12 /C6 7.13 /C6 /C6                             |
|               | U MD A             | ( 11,88 7.489 /C6 (7.491 ) /C6 1.17 /C6 0.78             | 2.30 /C6 0.90 327.3 0 /C6 ) 11,90 0.56 ( ) ) 42,012.50 |
|               | ( ) ) )            | ) 8.80 1.70                                              | 2417.38                                                |
|               | 6.27 /C6 1.37      | 3.68) 0.001 1.43 (15) 0.66 (1)                           | (7) 14.04 /C6 1.37 /C6                                 |
|               | 2                  | /C6 ) /C6                                                | 0.90 /C6 6.27 12,647.40                                |
|               | ACO-B              | 11,87 7.492 (7.492 12.40 /C6                             | 6.70 /C6 18.40 11,87 )                                 |
|               | ) 11,87            | ( ) 0.40                                                 | ) ( ) 413.92                                           |
|               | /C6 2.12           | (15) (1)                                                 | (7) /C6 2.12                                           |
|               | ACO-B 1 5.89       | 11,87 3.68) 7.492 /C6 0.000 (7.492 ) 13.20 /C6 1.72 0.49 | /C6 /C6 0.60 /C6 28.42 5.89 12,01 9.00 /C6 378.71      |
|               | ) 11,87            | ( ) 0.40                                                 | 6.80 48.50 ) 11,87                                     |

/C6

11,903.12

)

)

)

(

K2

17

(10)

1.11

/C6

8.60

(13)

2.99

/C6

16.40

A

0.09

/C6

6.27

)

)

(*

rs

NVa

3

ble

Ta

BOBLO

for

ults

Res

)

(best

r

/C6

l

BLO

BO

PBIL

ILS

1.03)

11,88

3.13)

11,88

)

(

7.476

0.001

/C6

7.489

0.005

/C6

7.488

KL

)

(7.491

)

(7.492

3

(0)

0.80

/C6

1.60

(0)

1.42

/C6

1.70

D

6

(3)

0.75

/C6

2.20

(6)

0.97

/C6

6.40

I

1

73.83

/C6

0

368.2

4.97

/C6

8.30

It

-

9.15

/C6

0.03

11,90

-

HC

K2no

1255

)

þ

(

/C6

62,965.00

/C6

21,854.10

)

)

(

DEst

2443.03

514.50

5

37,28

/C6

E05

20.83

/C6

E04

98.11

TEst

4

36.53E0

3

25.44E0

As a reference for the goodness of the results we can consider the K2 values for the true graphical structures, which are ) 14,412.69 for ALARM, ) 58,120.95 for INSURANCE and ) 11,907.09 for BOBLO. In order to obtain signification comparisons between the algorithms, a statistical analysis has been carried out: We have chosen the Mann-Whitney test for comparison between the stochastic algorithms and a t -test for comparison between the deterministic HCST and the stochastic algorithms. In all the cases a 5% significance level has been used. First, we have compared the two ACO-B algorithms and then we have chosen the best as the reference for comparisons. When significant differences are found, they are shown in the tables by a ( þ ) if a positive difference is found and ( ) ) if a negative difference is found for the case of K2 and DEst values, and by a (* þ )or(* ) ) for the estimation of CPU time (although it appears in the rows corresponding to

## NVars).

From the analysis of these data we can draw the following conclusions:

- · The best result found for ALARM has a K2 value of ) 14,401.29 and was found by the ACO-B algorithms. The same occurs for INSURANCE and BOBLO, where the best network found has a K2 value of ) 57,763.07 and ) 11,873.68 respectively. Notice that, in all cases, these results are considerably better than the reference values.
- · With respect to the accuracy of the algorithms, it is clear that ACO-B algorithms improve on the results obtained by the rest of algorithms. This conclusion is valid for the K2 and KL values, and also for structural differences ( A þ D þ I ), except for the BOBLO domain, where ACO-B tends to include more arcs than EDA.
- · From the results, it seems that the local search step carried out by applying HCST to the solutions obtained by the ants between iterations and in the last generation, does not significantly improve the quality of the networks. A different scheme of local optimization may work better. ACO-B2 only significantly improves ACO-B1 in the ALARM domain. In the remaining of domains both ACO algorithms obtain similar results.
- · With respect to the efficiency of the algorithms, it is clear that the fastest is HCST. Of the evolutionary algorithms, ACO is the fastest. This affirmation can be clearly deduced from the number of different statistics computed (DEst) and from the number of variables involved in these statistics (NVars).
- · As a general conclusion, we think that the use of heuristic knowledge by the ants when they are constructing a new sequence, helps to guide the search. For this reason, the results are of high quality and improve on the ones obtained by other methods. Furthermore, the use of this heuristic knowledge causes relatively small changes in the network being constructed with respect to those previously obtained, and so a great deal of computations can be reused, increasing the efficiency of the algorithm as well.

Following up this idea, we believe that a more informed initialization of the population in UMDA and PBIL, can help the process to be focused on promising regions of the search space.

## 5. Concluding remarks

In this paper a new scoring-based algorithm for learning BNs has been studied. The novelty of our method lies in the use of the ACO metaheuristic to guide the search process. This allows the algorithm to exploit heuristic knowledge about the problem, together with a simple but efficient form of cooperation between independent agents. This may be particularly important for large problems, because we can use distributed computation to obtain good solutions without increasing the computational cost. The experimental results are encouraging: Our ACO-B algorithms clearly outperform all the other algorithms based on different search methods, which we have used in this paper for comparative purposes.

For future research, our aim is to look more closely at the use of ACO for learning BNs, by refining the proposed algorithm (parameter fitting, other forms of local optimization, use of different types of ants or transition rules, etc.). More empirical studies are also required to definitively establish the validity of our approach. On the other hand, several authors [3,16,19,24,33] have successfully used the variable ordering space for learning BNs. It would also be interesting, therefore, to apply ACO to search in the space of orderings.

## Acknowledgements

This work has been supported by the Spanish Ministerio de Ciencia y Tecnolog /C19 ı ıa (MCyT), under projects TIC2001-2973-C05-01 and TIC2001-2973C05-05.

## References

- [1] S. Acid, L.M. de Campos, Benedict: an algorithm for learning probabilistic Bayesian networks, in: Proceedings of the International Conference on Information Processing and Management of Uncertainty in Knowledge Based Systems (IPMU /C213 96), 1996, pp. 979-984.

- [4] S. Baluja, Population-based incremental learning: a method for integrating genetic search based function optimization and competitive learning, Technical Report CMU-CS-94-163, Computer Science Department, Carnegie Mellon University, 1994.
- [5] I.A. Beinlich, H.J. Suermondt, R.M. Chavez, G.F. Cooper, The case study with two probabilistic inference techniques for Bayesian networks, in: Proceedings of the Second European Conference on Artificial Intelligence in Medicine, Springer-Verlag, 1989, pp. 247-256.
- [6] J. Binder, D. Koller, S. Russell, K. Kanazawa, Adaptive probabilistic networks with hidden variables, Machine Learning 29 (2) (1997) 213-244.
- [7] R. Blanco, I. Inza, P. Larra ~ n naga, Learning Bayesian networks in the space of structures by Estimation of Distribution Algorithms, International Journal of Intelligent Systems, in press.
- [8] R.R. Bouckaert, Bayesian belief networks: from construction to inference, Ph.D. thesis, University of Utrecht, 1995.
- [9] W. Buntine, Theory refinement of Bayesian networks, in: Proceedings of the 17th Conference on Uncertainty in Artificial Intelligence, Morgan Kaufmann, 1991, pp. 52-60.
- [10] W. Buntine, A guide to the literature on learning probabilistic networks from data, IEEE Transactions on Knowledge and Data Engineering 8 (1996) 195-210.
- [11] D.M. Chickering, D. Geiger, D. Heckerman, Learning Bayesian networks: search methods and experimental results, in: Preliminary Papers of the Fifth International Workshop on Artificial Intelligence and Statistics, 1995, pp. 112-128.
- [12] D.M. Chickering, D. Geiger, D. Heckerman, Learning Bayesian networks is NP-complete, in: D. Fisher, H. Lenz (Eds.), Learning from Data: Artificial Intelligence and Statistics V, Springer-Verlag, 1996, pp. 121-130.
- [13] G.F. Cooper, E. Herskovits, A Bayesian method for the induction of probabilistic networks from data, Machine Learning 9 (4) (1992) 309-348.
- [14] D. Dash, M. Druzdel, A hybrid anytime algorithm for the construction of causal models from sparse data, in: Proceedings of the 15th Conference on Uncertainty in Artificial Intelligence, Morgan Kaufmann, 1999, pp. 142-149.
- [15] L.M. de Campos, Independency relationships and learning algorithms for singly connected networks, Journal of Experimental and Theoretical Artificial Intelligence 10 (4) (1998) 511549.
- [16] L.M. de Campos, J.F. Huete, Approximating causal orderings for Bayesian networks using genetic algorithms and simulated annealing, in: Proceedings of the International Conference on Information Processing and Management of Uncertainty in Knowledge-based Systems (IPMU /C213 00), 2000, pp. 333-340.
- [17] L.M. de Campos, J.F. Huete, A new approach for learning Bayesian networks using independence criteria, International Journal of Approximate Reasoning 24 (2000) 11-37.
- [18] L.M. de Campos, J.M. Puerta, Stochastic local and distributed search algorithms for learning Bayesian networks, in: III International Symposium on Adaptive Systems (ISAS): Evolutionary Computation and Probabilistic Graphical Model, 2001, pp. 109-115.
- [19] L.M. de Campos, J.M. Puerta, Stochastic local search algorithms for learning Bayesian networks: searching in the space of orderings, in: European Conference on Symbolic and Quantitative Approaches to Reasoning with Uncertainty (ECSQARU /C213 2001), LNAI 2143, Springer, 2001, pp. 228-239.
- [20] M. Dorigo, G. Di Caro, The ant colony optimization meta-heuristic, in: D. Corne, M. Dorigo, F. Glover (Eds.), New Ideas in Optimization, McGraw-Hill, 1999, pp. 11-33.
- [21] M. Dorigo, G. Di Caro, L.M. Gambardella, Ant algorithms for discrete optimization, Artificial Life 5 (2) (1999) 137-172.
- [22] M. Dorigo, L.M. Gambardella, Ant colony system: a cooperative learning approach to the traveling salesman problem, IEEE Transactions on Evolutionary Computation 1 (1997) 53-66.
- [23] M. Dorigo, V. Maniezzo, A. Colorni, The ant system: optimization by a colony of cooperating agents, IEEE Transactions on Systems, Man and Cybernetics, Part B 26 (1996) 29-41.

- [24] N. Friedman, D. Koller, Being Bayesian about network structure, in: Proceedings of the 16th Conference on Uncertainty in Artificial Intelligence, Morgan Kaufmann, 2000, pp. 201-210.
- [25] J.A. G /C19 a amez, J.M. Puerta, Searching the best elimination sequence in Bayesian networks by using ant-colony optimization, Pattern Recognition Letters 23 (1-3) (2002) 261-277.
- [26] D. Heckerman, D. Geiger, D.M. Chickering, Learning Bayesian networks: the combination of knowledge and statistical data, Machine Learning 20 (1995) 197-244.
- [27] M. Henrion, Propagating uncertainty by logic sampling in Bayes networks, in: J. Lemmer, L.N. Kanal (Eds.), Uncertainty in Artificial Intelligence, vol. 2, North Holland, Amsterdam, 1988, pp. 149-164.
- [28] F.V. Jensen, Bayesian Networks and Decision Graphs, Springer-Verlag, 2001.
- [29] S. Kullback, Information Theory and Statistics, Dover, New York, 1968.
- [30] W. Lam, F. Bacchus, Learning Bayesian networks. An approach based on the MDL principle, Computational Intelligence 10 (4) (1994) 269-293.
- [31] P. Larra ~ n naga, Aprendizaje estructural y descomposici /C19 o on de redes Bayesianas v /C19 ı ıa algoritmos gen /C19 e eticos, Ph.D. thesis, University of Basque Country, 1995 (in Spanish).
- [32] P. Larra ~ n naga, J.A. Lozano (Eds.), Estimation of Distribution Algorithms. A New Tool for Evolutionary Computation, Kluwer Academic Press, 2001.
- [33] P. Larra ~ n naga, C. Kuijpers, R. Murga, Learning Bayesian network structures by searching for the best ordering with genetic algorithms, IEEE Transactions on System, Man and Cybernetics 26 (4) (1996) 487-493.
- [34] P. Larra ~ n naga, M. Poza, Y. Yurramendi, R. Murga, C. Kuijpers, Structure learning of Bayesian networks by genetic algorithms: a performance analysis of control parameters, IEEE Transactions on Pattern Analysis and Machine Intelligence 18 (9) (1996) 912-926.
- [35] Z. Michalewicz, Genetic Algorithms þ Data Structures ¼ Evolution Programs, SpringerVerlag, 1996.
- [36] H. M € u uhlenbein, The equation for response to selection and its use for prediction, Evolutionary Computation 5 (1998) 303-346.
- [37] J. Pearl, Probabilistic Reasoning in Intelligent Systems: Networks of Plausible Inference, Morgan Kaufmann, San Mateo, 1988.
- [38] J. Pearl, T.S. Verma, Equivalence and synthesis of causal models, in: Proceedings of the 6th Conference on Uncertainty in Artificial Intelligence, 1990, pp. 220-227.
- [39] L.K. Rasmussen, Bayesian Network for blood typing and parentage verification of cattle, Ph.D. thesis, Research Centre Foulum, Denmark, 1995.
- [40] M. Singh, M. Valtorta, Construction of Bayesian network structures from data: a brief survey and an efficient algorithm, International Journal of Approximate Reasoning 12 (1995) 111131.
- [41] P. Spirtes, C. Glymour, R. Scheines, Causation, Prediction, and Search, Lecture Notes in Statistics 81, Springer-Verlag, 1993.