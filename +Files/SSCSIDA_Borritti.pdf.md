NaΥıve Bayes Ant Colony Optimization

for Experimental Design

Matteo Borrotti and Irene Poli

Abstract. In a large number of experimental problems the high dimensionality of the search space and economical constraints can severely limit the number of experiment points that can be tested. Under this constraints, optimization techniques perform poorly in particular when little a priori knowledge is available. In this work we investigate the possibility of combining approaches from advanced statistics and optimization algorithms to effectively explore a combinatorial search space sampling a limited number of experimental points. To this purpose we propose the Na¨ıve Bayes Ant Colony Optimization (NACO) procedure. We tested its performance in a simulation study.

Keywords: Ant colony algorithm, combinatorial cptimization, na¨ıve Bayes classifier.

1

Introduction

In this work we address the problem of developing a novel approach for com-

binatorial optimization in the context of experimental design. This approach

should also improve the exploration of high dimensional spaces characterized by many interactions between variables. More precisely we are interested in development of a new approaches for effectively exploring enzyme sequence space to improve or redesign enzyme

Matteo Borrotti ≤ Irene Poli European Centre for Living Technology (ECLT), San Marco 2940, Venice

Matteo Borrotti

Irene Poli

Department of Enviromental Science, Informatics and Statistics,

≤

University Ca Foscari of Venice, Dorsoduro 2137, Venice

e-mail: {matteo.borrotti,irenpoli}@unive.it

R. Kruse et al. (Eds.): Synergies of Soft Computing and Statistics, AISC 190, pp. 489GLYPH<150>497.

springerlink.com c

© Springer-Verlag Berlin Heidelberg 2013

490 M. Borrotti and I. Poli functionality. Similar problems are presented in [1, 2]. In these works we designed a library of 95 different amino-acid sequences ( i.e. words), that are subsequently assembled to yield a full-length enzyme ( i.e. string) of length 4. To this regard, we can consider an enzyme as a string composed by 4

words. For each position in the string we can select an element from the set of 95 words, with repetition. The words are non-ordered discrete elements. The ultimate aim is to find the 'most informative' string in according with a specific function. Starting from this application, this work endeavours to define a new approach within statistical Design of Experiments for optimization based on Evolutionary Model Based Experimental Design [4]. In our case we have developed the Na¨ıve Bayes Ant Colony Optimization (NACO) approach. NACO is a mixed approach that combines different methods from statistics and computer science. More precisely: (i) Na¨ıve Bayes Classifier [6]; and (ii) Ant

Colony Optimization [3], specifically MAX -MIN Ant System [7]. In the next sections a brief description of the two approaches is given. Our strategy identifies which elements mostly affect the response of the systems and then it adopts this information to help the metaheuristc algo-

rithm in choosing the next set of candidate solutions.

## 2 NaΥıve Bayes Classifier

The Na¨ıve Bayes Classifier [6] is a classification procedure based on the Bayes' rule. It assumes that the set of variables X 1 , ..., X n , we consider for classification, are all conditionally independent of one another, given the response Y . We describe the conditional probability of X = X 1 , ..., X n on Y as:

〈 〉 n

P ( X 1 ,X 2 ,...,X n | Y )= ∏ i =1 P ( X i | Y ) This equation follows directly from the definition of conditional independence.

Starting from this point, it is possible to understand how the Na¨ıve Bayes

Approach works. Assuming in fact that Y is any discrete valued variable, and the attributes X 1 ,...,X n are any discrete or real valued variables, the goal of Na¨ıve Bayes method is to train a classifier that will output the probability distribution over possible values of Y, for each new instance X that we want to classify. The probability that Y will take on its k -th possible value, according to the Bayes' Rule, is P ( Y = y k X 1 ,...X n )= P ( Y = y k ) P ( X 1 ,...,X n | Y = y k )

| ∑ j P ( Y = y j ) P ( X 1 ,...,X n | Y = y j )

NaΥıve Bayes Ant Colony Optimization for Experimental Design 491

where the sum is taken over all possible value

y

j i conditionally independent given Y , we can write P ( Y = y k | X 1 ,...X n )= P ( Y = y k ) ∏ i P ( X 1 ,...,X n | Y = y k ) ∑ j P ( Y = y j ) ∏ i P ( X i | Y = y j ) (1)

of

Y

. Assuming that

X

are

Equation (1) is the base for the Na¨ıve Bayes Classifier. Given a new instance

X new = 〈 X 1 , ..., X n 〉 , it is possible to calculate the probability that Y will take on any given value. If we want to know the most probable value of Y , we obtain the Na¨ıve Bayes' Rule:

Y ← arg max y k P ( Y = y k ) ∏ i P ( X 1 ,...,X n | Y = y k ) ∑ j P ( Y = y j ) ∏ i P ( X i | Y = y j ) This approach reduces the complexity for learning Bayesian classifier by mak-

ing a conditional independence assumption that dramatically reduces the

number of parameters to be estimated when modeling

P

(

X

3

Ant Colony Optimization (ACO)

Ant Colony Optimization (ACO) [3] is a population-based, general-purpose stochastic search technique for the solution of difficult combinatorial problems, which is inspired by the pheromone trail laying and following foraging behaviour of some real ant species.

In ACO, each 'ant' builds a solution starting from an initial state selected according to some problem dependent criteria. A solution is expressed as minimum cost (shortest) path through the states of the problem in accordance with the problem's constraints. A single ant is able to build a solution but only the cooperation among all the agents of the colony, concurrently building different solutions, is able to find high quality solutions. In the algorithm an environment is simulated by a graph composed of a set N of states, representing nodes, and a set E of arcs fully connecting the

nodes N .Let d ij be the length of the arc ( i, j ) ∈ E , that is the distance between nodes i and j , with i, j ∈ N . The aim of the optimization in ACO is to find on the graph G =( N,E ) the minimal length path connecting nest to the source. ACO uses two different types of information: pheromone and a priori problem-specific information (heuristic values). The combination of available pheromone and heuristic values defines ant-decision tables , that are, proba-

bilistic tables used by the ants' decision policy to direct their search towards the most interesting regions of the search space.

|

Y

).

492

M. Borrotti and I. Poli

The ant-decision table A i =[ a j ( t )] | N i | of node i is obtained by the composition of the pheromone trail values with heuristic values as follow: a ij = [ τ ij ( t )] α [ η ij ] β ∑ l ∈ N i [ τ ij ( t )] α [ η ij ] β ∀ j ∈ N i (2)

i

where τ ij ( t ) is the amount of pheromone trail on arc ( i, j ) at time t ; η i,j = 1 /d ij is the heuristic value to move from node i to node j ; N i is the set of neighbours of node i ;and α and β are two parameters that control the

relative weight of pheromone trail and heuristic information. The probability with which an ant k chooses to go from node i to node j N k while building its tour at the t -th algorithm iteration, is:

∈

i

p k ij ( t )= a ij ( t ) ∑ i ∈ N k i a ij ( t ) (3)

where N k i ⊆ N i is the set of nodes in the neighbourhood of node i that ant k has not visited yet. After all ants have completed their tour, pheromone evaporation on all arcs is applied in according with the pheromone trail decay coefficient (or

evaporation factor) ρ , ρ ∈ (0 , 1]. In our case, we apply the MAX -MIN Ant System ( MM AS), proposed by Stutzle and Hoos in [7] because it is demonstrated that MM AS is able to reach a strong exploitation of the search space by adding pheromone only to the best solution during the pheromone trail update. Moreover they applied a simple method for limiting the strength of the pheromone trails that effectively avoids premature convergence of the search.

4

The NACO Approach

We introduce a novel approach for optimization called Na¨ıve Bayes Ant

Colony Optimization (NACO) by combining Ant Colony Optimization tech-

nique and Na¨ıve Bayes Classifier. NACO extracts the information from the data using the Na¨ıve Bayes Approach and explores the search space by the ACO algorithm. At the same time, the most informative variables and interactions are identified. In our problem we are considering a specific problem caracterized by a discrete search space where a solution is a string composed by 4 words. The words are selected from a set of 95 discrete elements with repetition. We

need to reformulate the problem as a path search problem to apply ACO algorithm. For this purpose, we create a graph where each node represents a specific word. A solution is a path with length 4 composed of 4 nodes connected by 3 arcs, as shown in Fig. 1. In the biological application [1], each node corresponds to an amino-acid sequence from the initial dataset

glyph<c=2,font=/HEMEDM+Calibri-Bold>glyph<c=5,font=/HEMEDM+Calibri-Bold>glyph<c=4,font=/HEMEDM+Calibri-Bold>glyph<c=6,font=/HEMEDM+Calibri-Bold>glyph<c=4,font=/HEMEDM+Calibri-Bold>glyph<c=7,font=/HEMEDM+Calibri-Bold>glyph<c=10,font=/HEMEDM+Calibri-Bold>glyph<c=1,font=/HEMEDM+Calibri-Bold>glyph<c=12,font=/HEMEDM+Calibri-Bold>glyph<c=1,font=/HEMEDM+Calibri-Bold> . . .

picture-1.png

glyph<c=2,font=/HEMEDM+Calibri-Bold>glyph<c=5,font=/HEMEDM+Calibri-Bold>glyph<c=4,font=/HEMEDM+Calibri-Bold>glyph<c=6,font=/HEMEDM+Calibri-Bold>glyph<c=4,font=/HEMEDM+Calibri-Bold>glyph<c=7,font=/HEMEDM+Calibri-Bold>glyph<c=10,font=/HEMEDM+Calibri-Bold>glyph<c=1,font=/HEMEDM+Calibri-Bold>glyph<c=16,font=/HEMEDM+Calibri-Bold>glyph<c=15,font=/HEMEDM+Calibri-Bold>glyph<c=1,font=/HEMEDM+Calibri-Bold>

glyph<c=3,font=/HEMEDM+Calibri-Bold>glyph<c=8,font=/HEMEDM+Calibri-Bold>glyph<c=9,font=/HEMEDM+Calibri-Bold>glyph<c=11,font=/HEMEDM+Calibri-Bold>glyph<c=1,font=/HEMEDM+Calibri-Bold>

glyph<c=3,font=/HEMEDM+Calibri-Bold>glyph<c=8,font=/HEMEDM+Calibri-Bold>glyph<c=9,font=/HEMEDM+Calibri-Bold>glyph<c=12,font=/HEMEDM+Calibri-Bold>glyph<c=1,font=/HEMEDM+Calibri-Bold>

glyph<c=3,font=/HEMEDM+Calibri-Bold>glyph<c=8,font=/HEMEDM+Calibri-Bold>glyph<c=9,font=/HEMEDM+Calibri-Bold>glyph<c=13,font=/HEMEDM+Calibri-Bold>glyph<c=1,font=/HEMEDM+Calibri-Bold>

glyph<c=3,font=/HEMEDM+Calibri-Bold>glyph<c=8,font=/HEMEDM+Calibri-Bold>glyph<c=9,font=/HEMEDM+Calibri-Bold>glyph<c=14,font=/HEMEDM+Calibri-Bold>glyph<c=1,font=/HEMEDM+Calibri-Bold>

Fig. 1 A new representation of the graph where ants move. A solution is a path composed by 4 nodes and 3 arcs. and an arc to the connection between amino-acid sequence i in position k and amino-acid sequence j in position k +1. A candidate solution is a path

composed by 4 amino-acid sequences which represents a full-length enzyme. The total number of nodes is 95 × 4 = 380 with equal size intervals. Candidate solutions' response is calculated and subsequently discretized in according with a certain constant threshold ( γ , with γ ∈ R )fixedby the experimenter. The candidate solutions (strings) are assigned to class 1 if the responses exceed the threshold otherwise to class 0. The Na¨ıve Bayes Classifier is applied on each position of the sequence or string with class equal to 1. It calculates the maximum likelihood estimate for each word in each position given the training sequences or strings. We obtain a set of probability distributions, one for each position. These probability distributions are used to weigh the arcs on the ACO algorithm.

The following steps summarize the NACO approach:

- 1. Random generation and evaluation of an initial population (set of candidate points); 2. Identification of the Iteration Best Solution (best solution in the current
- iteration); 3. Calculation of the Na¨ıve Bayes Classifier on the available evaluated solutions ( N ). At each iteration, it focuses on values of the response greater
- than a problem-specific threshold γ , with γ ∈ R ; 4. Updating the probabilities with which an ant k chooses to go from element i to element j using the information extracted in points 2 and 3;
- 5. Selection of the next population of candidate solutions using the principle of MAX -MIN Ant System; 6. Evaluation of the new set of candidate solutions and inclusion of the new
- set in the set of solutions that has already been evaluated; 7. If stop criterion is reached, then stop. Otherwise repeat points from 2 to
- 6; Fig. 2 describes point number 4. At iteration t , agents move over the graph
- according to the best paths identified in the previous steps (Fig. 2 (a)).

493

Fig. 2

picture-2.png

(c)

Updating Phase of the NaΥıve Bayes Ant Colony Optimization

Following candidate solution evaluation, the Iteration Best Solution is identified and the corresponding pheromone path is updated (Fig. 2 (c)). At this point, using the Na¨ıve Bayes Classifier the best variables are identified, namely those that are anticipated to yield a fitness value higher than the chosen fitness treshold γ (Fig. 2 (b)). For any given arc connecting variable (node) i with variable j ,theweight λ ij is changed according to the Na¨ıve Bayes Classifier. The set of { λ ij } is called NaΥıve Information .Now,theantdecision table A i =[ a i j ( t )] | N i | of node i will be obtained by the composition of the pheromone trail values with heuristic values and with NaΥıve Information as follows:

[

τ

ij

(

t

)]

α

β

δ

a ij = ∑ l ∈ N i [ τ ij ( t )] α [ η ij ] β [ λ ij ] δ ] ∀ j ∈ N i ij ( t ) is the amount of pheromone trail on arc ( i, j ) at time

where

τ

t

;

η

1

/d

ij

is the heuristic value of moving from node

i

to node

j

;

λ

ij

is the

i,j

=

NaΥıve

Information on arc ( i, j ) at time t . N i is the set of neighbors of node i ;and α , β and δ are parameters that control the relative weight of pheromone trail, heuristic information and NaΥıve Information . The probability, which an ant k chooses to go from element i to element j , is then calculated according to Equation (3). Generally, NACO will extract information from few data and it will individuate the best connection between variables.

[

η

ij

]

[

λ

ij

]

NaΥıve Bayes Ant Colony Optimization for Experimental Design 495 5 Monte Carlo Simulations In this work we develop simulative studies with the aim of testing the performance of the NACO approach. In these simulations the experiments are generated by the model y = ϕ h ( x )+ /epsilon1 , /epsilon1 ∼ N (0 ,σ 2 h ), where h =1 , 2 denotes two different response surfaces, ϕ 1 ( x )and ϕ 2 ( x ), taken from [2]. ϕ 1 ( x )is called Polynomial Regression Model (PRM) and it represents a fitness landscape dominated by strong interactions, which occurs when the effect of one variables depends on the presence of another. The second formal structure, ϕ 2 ( x ), is called Polynomial Sparse Regression Model (PSRM) and it represents the situation where only few variables highly influence the response of

the system and the others are close to 0. This kind of fitness landscapes is characterized by ruggedness and local optima. ϕ 1 ( x )and ϕ 2 ( x )simulatea search space of 95 4 experimental points.

In both simulations, we computed 50 Monte Carlo runs with the aim to maximize the deterministic function ϕ h . In Fig. 3, we compare the convergence of our method with the basic MM AS. The parameter settings for MM AS is: population size ( N ) 100, number of generations or experimental batches 50, evaporation factor ( ρ ) 0.96 and weight for the pheromone ( β ) 1. In this case no heuristic information is used. NACO includes two more parameters: δ that controls the weight of NaΥıve Information and γ that is the threshold considered by the Na¨ıve Bayes Classifier. In our case δ and γ are equal to 2 and 80%, respectively. The parameter setting is chosen in accordance with preliminary studies [1].

Our empirical results (Fig. 3) show remarkable efficiency of the proposed method. The main difference between NACO and MM AS is that the last

180

200

picture-3.png

496 M. Borrotti and I. Poli one does not benefit from the information obtained using the Na¨ıve Bayes Classifier. Our results show that the inclusion of the Na¨ıve Bayes Classifier

boosts the convergence rate. This is visible after few iterations of the NACO algorithm and is due to the fact that the Na¨ıve Bayes Classifier extracts useful information on the main variables that influence the response. The

information extrapoleted from the Na¨ıve Bayes Classifier compensates for

the absence of heuristic information.

In term of computational time the two approaches do not show significante

## difference.

6 Conclusions In this work, we have explored the possibility of tackling problems characterized by a very large experimental space combining bio-inspired algorithms with advanced statistical techniques. To achieve this aim, we have developed

an algorithmic approach which combines some powerful features of known approaches: the Na¨ıve Bayes Ant Colony Optimization (NACO). We have shown that the Na¨ıve Bayes Ant Colony Optimization (NACO) approach improves upon the limits of the individual techniques, enabling us to deal with very large experimental spaces. In fact, the Na¨ıve Bayes Approach has a strong assumption and it assumes that the attributes X 1 ,...,X n are all conditionally independent of one another, given the response Y .Ithasthe advantage of simplifying the representation of the probability of X given Y but with the Na¨ıve Bayes Classificator it is not possible to understand the relations between the attributes. This aspect can be extremely important in some experimental problems. The combination of ACO and Na¨ıve Bayes Classifier can describe the rel-

ative network between variables. ACO, in fact, is based on probabilistic matrices where the best path has a higher probability of being chosen. A path is composed of nodes and arcs. In our problem nodes can be seen as variables and an arc, connecting a variable to the next one, can be seen as the relation that exists between the two variables. Then, ACO implies the sequential relationship between variables. At last, NACO can be used in the context of combinatorial optimization in absence of heuristc information since Na¨ıve Bayes Classifier is used to extract information from the available data.

References

1. Borrotti, M.: An evolutionary approach to the design of experiments for com-

binatorial optimization with an application to enzyme engineering. PhD thesis,

Department of Statistical Science, University of Bologna, Italy (2011)

- NaΥıve Bayes Ant Colony Optimization for Experimental Design 497
- 2. Borrotti, M., De Lucrezia, D., Minervini, G., Poli, I.: A Model Based Ant Colony
- Design for the Protein Engineering Problem. In: Dorigo, M., Birattari, M., Di
- Caro, G.A., Doursat, R., Engelbrecht, A.P., Floreano, D., Gambardella, L.M., GroGLYPH<223>, R., S˘ahin, E., Sayama, H., StΥutzle, T. (eds.) ANTS 2010. LNCS, vol. 6234, pp. 352GLYPH<150>359. Springer, Heidelberg (2010) 3. Dorigo, M., Di Caro, G.: The ant colony optimization meta-heuristic. In: Corne,
- D., et al. (eds.) New Ideas Optim., pp. 11GLYPH<150>32. McGraw-Hill, New York (1999) 4. Forlin, M., Slanzi, D., Poli, I.: Combining Probabilistic Dependency Models and Particle Swarm Optimization for Parameter Inference in Stochastic Biological
- Systems. In: Gaol, F.L., Nguyen, Q.V. (eds.) Proc. of the 2011 2nd International
- Congress CACS. AISC, vol. 145, pp. 437GLYPH<150>444. Springer, Heidelberg (2012) 5. Kohonen, J., Talikota, S., Corander, J., Auvinen, P., Arjas, E.: A naΥıve bayes classifier for protein function prediction. Silico Biol. 9, 23GLYPH<150>34 (2009)

6. Mitchell, T.M.: Machine Learning. McGraw-Hill, New York (1997)

7. StΥutzle, T., Hoos, H.: Max-min ant system. Future Gener. Comp. Sy. 16, 889GLYPH<150>914

(2000)