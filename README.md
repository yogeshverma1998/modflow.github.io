*This website contains information regarding the paper Modular Flows: Differential Molecular Generation.*

> **TL;DR:** We propose generative graph normalizing flow models, based on a system of coupled node ODEs, that repeatedly reconcile locally toward globally aligned densities for high quality molecular generation

# Problem of Molecular Generation

A key challenge of molecular generative models is to be able to generate valid molecules, according to various criteria for molecular validity or feasibility. It is a common practice to use external chemical software as rejection oracles to reduce or exclude invalid molecules, or do validity checks as part of autoregressive generation [1,2,3] . An important open question has been whether generative models can learn to achieve high generative validity *intrinsically*, i.e., without being aided by oracles or performing additional checks.

<p align="center">
  <img src="https://raw.githubusercontent.com/yogeshverma1998/Modular-Flows-Differential-Molecular-Generation/main/mol_gen_intro.png" />
</p>


# Continuous Normalizing Flows

<p align="center">
  <img src="https://raw.githubusercontent.com/yogeshverma1998/Modular-Flows-Differential-Molecular-Generation/main/nf_website.png" />
</p>

Normalizing flow have seen widespread use for density modeling, generative modeling, etc which provides a general way of constructing flexible probability distributions. It is defined by a parameterized invertible deterministic transformation from a base distribution $$\mathcal{Z}$$ (e.g., Gaussian distribution) to real-world observational space $$X$$ (e.g. images and speech). When the dynamics of transformation is governed by an ODE, the method is known as Continous Normalizing Flows (CNFs). The process starts by sampling from a base distribution $$\mathbf{z}_0 \sim p_0(\mathbf{z}_0)$$, then solving the IVP $$\mathbf{z}(t_0) = \mathbf{z}_0$$, $$\dot{\mathbf{z}}(t) = \frac{\partial \mathbf{z}(t)}{\partial t} = f(\mathbf{z}(t),t;\theta)$$, where ODE is defined by the parametric function $$f(\mathbf{z}(t),t;\theta)$$ to obtain $$\mathbf{z}(t_1)$$ which constitutes our observable data. Then, using the *instantaneous change of variables* formula change in log-density under this model is given as:


<p align="center">
    $$\frac{\partial \log p_t(\mathbf{z}(t))}{\partial t} = -\texttt{tr} \left( \frac{\partial f}{\partial \mathbf{z}(t)} \right)$$
</p> 

Given a datapoint $$\mathbf{x}$$, we can compute both the point $$\mathbf{z}_{0}$$ which generates $$\mathbf{x}$$, as well as $$\log p_1(\mathbf{x})$$ by solving the initial value problem which integrates the combined dynamics of $$\mathbf{z}(t)$$ and the log-density of the sample resulting in the computation of $$\log p_{1}(\mathbf{x})$$.


# Modular Flows

## Representation

We represent molecule as a graph $$G = (V,E)$$, where each vertex takes value from an alphabet on atoms:  $$v \in \mathcal{A} = \{ \texttt{C},\texttt{H},\texttt{N},\texttt{O},\texttt{P},\texttt{S},\ldots \}$$; while the edges $$e \in \mathcal{B} = \{1,2,3\}$$ abstract the type of bond (i.e., single, double, or triple). We assume the following decomposition of the graph likelihood, over vertices conditioned on the edges and given the latent representations, 

<p align="center">
    $$p(G) := p(V | E,\{ z\}) = \prod_{i=1}^M \texttt{Cat}(v_i | \sigma(\mathbf{z}_i))$$
</p> 

We obtained an alternative representation by decomposing a molecule into a tree like structure, by contracting certain vertices into a single node such that the molecular graph $$G$$ becomes acyclic. We followed a similar decompositon as JT-VAE[7], but restrict these clusters to ring-substructures, in addition to the atom alphabet. Thus, we obtain an extended alphabet vocabulary as $$\mathcal{A}_{\mathrm{tree}} = \{ \texttt{C},\texttt{H},\texttt{N}, \ldots,  \texttt{C}_{1},\texttt{C}_{2},\ldots \}$$, where each cluster label $$\texttt{C}_{r}$$ corresponds to the some ring-substructure in the label vocabulary $$\chi$$

<p align="center">
  <img src="https://raw.githubusercontent.com/yogeshverma1998/Modular-Flows-Differential-Molecular-Generation/main/junction_mod.png" />
</p>



## Differential Modular Flows

Based on the general recipie of normalizing flows, we propose to model the node scores $$\mathbf{z}_{i}$$ as a Continuous-time Normalizing Flow (CNF)[4] over time $$t \in \mathrm{R}_+$$. We assume the initial scores at time $$t=0$$ follow an uninformative Gaussian base distribution $$\mathbf{z}_i(0) \sim \mathcal{N}(0,I)$$ for each node $$i$$. Node scores evolve in parallel over time by a differential equation,

<p align="center">
    $$\dot{\mathbf{z}}_{i}(t) := \frac{\partial \mathbf{z}_i(t)}{\partial t} = f_\theta\big( t, \mathbf{z}_i(t), \mathbf{z}_{\mathcal{N}_i}(t),\mathbf{x}_{i}, \mathbf{x}_{\mathcal{N}_i} \big), \qquad i = 1, \ldots, M$$
  </p> 
  
where $$\mathcal{N}_{i} = \{ \mathbf{z}_{j} : (i,j) \in E \}$$ is the set of neighbor scores at time $$t$$, $$\mathbf{x}$$ is the spatial information (2D/3D), and $$\theta$$ are the parameters of the flow function $$f$$ to be learned. By collecting all node differentials we obtain a **modular** joint, coupled ODE, which is equivalent to a graph PDE [5,6], where the evolution of each node only depends on its immediate neighbors. 

<p align="center">
 $$\dot{\mathbf{z}}_{i}(t) = \begin{pmatrix} \dot{\mathbf{z}}_{i}(t)_1(t) \\ \vdots \\ \dot{\mathbf{z}}_{i}(t)_M(t) \end{pmatrix} = \begin{pmatrix} f_\theta\big( t, \mathbf{z}_1(t), \mathbf{z}_{\mathcal{N}_1}(t) \big) \\ \vdots \\ f_\theta\big( t, \mathbf{z}_M(t), \mathbf{z}_{\mathcal{N}_M}(t) \big) \end{pmatrix} $$
 </p>
## Equivariant local differential

## Training Objective

We reduce the learning problem to maximizing the score cross-entropy $$\mathrm{E}_{\hat{p}_{\mathrm{data}}(\mathbf{z}(T))}[\log p_\theta(\mathbf{z}(T))]$$, where we turn the observed set of graphs $$\{G_{n}\}$$ into a set of scores $$\{\mathbf{z}_{n}\}$$ by using one-hot encoding 
<p align="center">
$$\mathbf{z}_n (G_n; \epsilon) = (1-\epsilon)~\mathrm{onehot}(G_n) ~+~ \dfrac{\epsilon}{|\mathcal{A_s}|} \textbf{1}_{M(n)} \textbf{1}_{|\mathcal{A_s}|}^{\top}~,$$
</p>
where 

We exploit the non-reversible composition of the argmax and softmax to transition from continous space to discrete graph space, but short-circuit in reverse direction. This indeed allows to keep the forward and backward flows aligned.
<p align="center">
  <img src="https://raw.githubusercontent.com/yogeshverma1998/Modular-Flows-Differential-Molecular-Generation/main/tikz_diagram.png" />
</p>
We thus maximize an objective over $$N$$ training graphs, 

<p align="center">
$$\texttt{argmax}_\theta \qquad \mathcal{L} = \mathcal{E}_{\hat{p}_{\mathrm{data}}(\mathbf{z})} \log p_\theta(\mathbf{z}) \approx \frac{1}{N} \sum_{n=1}^N \log p_T\big( \mathbf{z}(T) = \mathbf{z}_n \big)$$     
</p>  

## Molecule Generation

We generate novel molecules by sampling an initial state $$\mathbf{z}(0) \sim \mathcal{N}(0,I)$$ based on structure, and running the modular flow forward in time until $$\mathbf{z}(T)$$. This procedure maps a tractable base distribution $$p_0$$ to some more complex distribution $$p_T$$. We follow argmax to pick the most probable label assignment for each node.

<p align="center">
  <img src="https://raw.githubusercontent.com/yogeshverma1998/Modular-Flows-Differential-Molecular-Generation/main/workflow_final.png" />
</p>

# Results
## Density Estimation

We demonstrated the power of our method on learning highly discontinous patterns on 2D grid graphs. We considered patterns corresponding to two-variants of chess-board pattern as $$4 \times 4$$, where every node has opposite value to its neighbors and $$16 \times 16$$ grid where blocks of $$4 \times 4$$ nodes have uniform values, but opposite across blocks. At last, we also considered alternate stripes pattern over $$20 \times 20$$.
<p align="center">
  <img src="https://raw.githubusercontent.com/yogeshverma1998/Modular-Flows-Differential-Molecular-Generation/main/toy_final.png" width="600" height="300" />
</p>


## Molecular Experiments
We trained the model on QM9[8] and ZINC250K[9] dataset, where molecules are in kekulized form with hydrogens removed by the RDkit[10] software. We adopt common quality metrics to evaluate molecular generation as,

- **Validity**: Fraction of molecules that satisfy chemical valency rule
- **Uniqueness**: Fraction of non-duplicate generations
- **Novelty**: Fraction of molecules not present in training data
- **Reconstruction**: Fraction of molecules that can be reconstructed from their encoding

<p align="center">
  <img src="https://raw.githubusercontent.com/yogeshverma1998/Modular-Flows-Differential-Molecular-Generation/main/result_gen_combined.png" />
</p>

Some of the generated molecules via $$\texttt{ModFlow}$$ are also shown above. We visually evaluate the generated structures via out method via properties distribution. We utilize kernel density estimation of these distributions to visualize these distributions. We use

- **Molecular Weight**: Sum of the individual atomic weights of a molecule.
- **LogP**: Ratio of concentration in octanol-phase to aqueous phase, also known as the octanol-water partition coefficient.
- **Synthetic Accessibility Score (SA)**: Estimate describing the  synthesizability of a given molecule
- **Quantitative Estimation of Drug-likeness (QED)**: Value describing likeliness of a molecule as a viable candidate for a drug

<p align="center">
  <img src="https://raw.githubusercontent.com/yogeshverma1998/Modular-Flows-Differential-Molecular-Generation/main/prop_dist_combined.png" />
</p>





## Ablation Studies
We performed ablation experiments to gain further insights about $$\texttt{ModFlow}$$. Specifically, we conducted ablation study to quantify the effect of incorporating the symmetries in our model as **E(3) Equivariant vs Not Equivariant**, where we compare the results to a 3-layer GCN and investigated whether including 3D coordinate information **2D vs 3D**, improves the model and evaluate the benefit of including the geometric information. 
<p align="center">
  <img src="https://raw.githubusercontent.com/yogeshverma1998/Modular-Flows-Differential-Molecular-Generation/main/ablation_final_combined.png" />
</p>

# Conclusion
> 1. 
> 2. 
> 3. 

# References

<ol>
  <li>Youzhi Luo, Keqiang Yan, and Shuiwang Ji. Graphdf: A discrete flow model for molecular graph generation,2021</li>
  <li>Chence Shi, Minkai Xu, Zhaocheng Zhu, Weinan Zhang, Ming Zhang, and Jian Tang. Graphaf: a flow-based autoregressive model for molecular graph generation</li>
  <li>Mariya Popova, Mykhailo Shvets, Junier Oliva, and Olexandr Isayev. Molecularrnn: Generating realistic molecular graphs with optimized properties,2019 </li>
  <li> Wengong Jin, Regina Barzilay, and Tommi Jaakkola. Junction tree variational autoencoder for molecular graph generation </li>
  <li>John J Irwin, Teague Sterling, Michael M Mysinger, Erin S Bolstad, and Ryan G Coleman. Zinc: a free tool to discover chemistry for biology. Journal of chemical information and modeling, 52(7):1757–1768, 2012</li>
  <li>Raghunathan Ramakrishnan, Pavlo O Dral, Matthias Rupp, and O Anatole von Lilienfeld. Quantum chemistry structures and properties of 134 kilo molecules. Scientific Data, 1, 2014</li>
  <li>Greg Landrum et al. Rdkit: A software suite for cheminformatics, computational chemistry, and predictive modeling, 2013</li>
  <li> Valerii Iakovlev, Markus Heinonen, and Harri Lähdesmäki. Learning continuous-time pdes from sparse data with graph neural networks.</li>
  <li> Ben Chamberlain, James Rowbottom, Maria I Gorinova, Michael Bronstein, Stefan Webb, and Emanuele Rossi Grand: Graph neural diffusion. In International Conference on Machine Learning, pages 1407–1418. PMLR, 2021</li>
</ol>





