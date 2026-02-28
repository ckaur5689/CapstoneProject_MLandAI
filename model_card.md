# Model Card

# 1. Model Overview
This model card describes the methodology, configuration, and performance of eight Gaussian Process (GP) surrogate models constructed via Bayesian Optimisation (BO) to approximate and find the global maximum of eight distinct synthetic black-box objective functions. The Upper Confidence Bound (UCB) acquisition function was used throughout to balance exploration and exploitation during the sequential search process.

    Model Type: Gaussian Process (GP) Surrogate Model
    Framework:	Bayesian Optimisation (BO)
    Acquisition Function: Upper Confidence Bound (UCB)
    Number of Surrogate Models: 8
    Objective: Global Maximum Discovery
    Target Functions: Synthetic Black-Box Functions (×8)

 # 2. Methodology
- 2.1 Bayesian Optimisation Framework
Bayesian Optimisation is a sequential, model-based strategy for optimising expensive or unknown black-box functions. Rather than evaluating the objective function exhaustively, BO maintains a probabilistic surrogate model that is updated with each new observation. The surrogate model provides both a predictive mean and uncertainty estimate at any queried point, which together guide the selection of the next evaluation location.

The BO loop follows the following iterative procedure:
•	Initialise the GP prior using a small set of random initial observations.
•	Fit the GP surrogate model to all available observations.
•	Optimise the UCB acquisition function to propose the next candidate point.
•	Evaluate the black-box function at the proposed candidate.
•	Update the GP with the new observation and repeat until the evaluation budget is exhausted.

- 2.2 Gaussian Process Surrogate Model
A Gaussian Process defines a distribution over functions and is fully specified by a mean function and a covariance (kernel) function. Given a set of observations, the GP posterior provides a tractable, closed-form predictive distribution over the function value at any unobserved location. This makes GPs particularly well-suited as surrogate models in BO, as their uncertainty estimates directly inform the acquisition function.

Formally, the GP is defined as: f(x) ~ GP(μ(x), k(x, x')), where μ(x) is the prior mean function (typically zero) and k(x, x') is the kernel function encoding assumptions about the smoothness and structure of the objective function.

  2.3 Upper Confidence Bound (UCB) Acquisition Function
The Upper Confidence Bound acquisition function selects the next evaluation point by maximising a weighted combination of the GP predictive mean and predictive standard deviation:

                                                  **UCB(x) = μ(x) + κ · σ(x)**
where μ(x) is the posterior predictive mean, σ(x) is the posterior predictive standard deviation, and κ ≥ 0 is a hyperparameter controlling the exploration–exploitation trade-off. A higher κ encourages broader exploration of uncertain regions, while a lower κ promotes exploitation of areas with high predicted values. The UCB formulation provides theoretical regret bounds under certain regularity conditions, making it a principled and well-studied acquisition strategy.

 # 3. Model Configuration

 - 3.1 Kernel & Hyperparameters
Each of the eight GP surrogate models was configured with a kernel and hyperparameter settings appropriate to the dimensionality and characteristics of the corresponding synthetic black-box function. Kernel hyperparameters (e.g., length-scale, output variance) were optimised by maximising the log marginal likelihood at each BO iteration to ensure the surrogate remained well-calibrated as observations accumulated.

        Configuration                        Parameter Setting
        GP Prior Mean                        Zero mean function
        Kernel Type                        Radial Basis Function (RBF) / Matérn (per function)
        Hyperparameter Optimisation                        Root Mean Square Error, Negative R^2 
        Observation Noise                        Modelled (nugget / noise variance)
        UCB κ (kappa)                        Tuned per function
        Initial Random Samples                        Defined per experimental setup
        Total BO Iterations                        Defined per experimental setup

 - 3.2 Synthetic Black-Box Functions
Eight synthetic benchmark functions were used as the black-box objectives. These functions are designed to test different challenges in global optimisation, including multimodality, varying curvature, and flat regions. A separate GP surrogate model was trained and optimised independently for each function.

**Function 1:**
- Description of sample application: Detect likely contamination sources in a two-dimensional area, such as a radiation field, where only proximity yields a non-zero reading. The system uses Bayesian optimisation to tune detection parameters and reliably identify both strong and weak sources.
  - Initial input data (x) provided:  2D array (10,2)
  - Initial output data (y) provided: 1D array (10,)  

**Function 2:**
- Description of sample application: Imagine a black box, or a mystery ML model, that takes two numbers as input and returns a log-likelihood score. Your goal is to maximise that score, but each output is noisy, and depending on where you start, you might get stuck in a local optimum. To tackle this, you use Bayesian optimisation, which selects the next inputs based on what it has learned so far. It balances exploration with exploitation, making it well suited to noisy outputs and complex functions with many local peaks.
  - Initial input data (x) provided:  2D array (10,2)
  - Initial output data (y) provided: 1D array (10,)  

**Function 3:**
- Description of sample application: You’re working on a drug discovery project, testing combinations of three compounds to create a new medicine. 
Each experiment is stored in initial_inputs.npy as a 3D array, where each row lists the amounts of the three compounds used. After each experiment, you record the number of adverse reactions, stored in initial_outputs.npy as a 1D array. Your goal is to minimise side effects; in this competition, it is framed as maximisation by optimising a transformed output (e.g. the negative of side effects). 
  - Initial input data (x) provided:  3D array (15,3)
  - Initial output data (y) provided: 1D array (15,)  

**Function 4:**
- Description of sample application: Address the challenge of optimally placing products across warehouses for a business with high online sales, where accurate calculations are costly and only feasible biweekly. To speed up decision-making, an ML model approximates these results within hours. The model has four hyperparameters to tune, and its output reflects the difference from the expensive baseline. Because the system is dynamic and full of local optima, it requires careful tuning and robust validation to find reliable, near-optimal solutions.
  - Initial input data (x) provided:  4D array (30,4)
  - Initial output data (y) provided: 1D array (30,)  

**Function 5:**
- Description of sample application: You’re tasked with optimising a four-variable black-box function that represents the yield of a chemical process in a factory. The function is typically unimodal, with a single peak where yield is maximised. Your goal is to find the optimal combination of chemical inputs that delivers the highest possible yield, using systematic exploration and optimisation methods.
  - Initial input data (x) provided:  4D array (20,4)
  - Initial output data (y) provided: 1D array (20,)

**Function 6:**
- Description of sample application: You’re optimising a cake recipe using a black-box function with five ingredient inputs, for example flour, sugar, eggs, butter and milk. Each recipe is evaluated with a combined score based on flavour, consistency, calories, waste and cost, where each factor contributes negative points as judged by an expert taster. This means the total score is negative by design. To frame this as a maximisation problem, your goal is to bring that score as close to zero as possible or, equivalently, to maximise the negative of the total sum.
  - Initial input data (x) provided:  5D array (20,5)
  - Initial output data (y) provided: 1D array (20,)

**Function 7:**
- Description of sample application: 	
You’re tasked with optimising an ML model by tuning six hyperparameters, for example learning rate, regularisation strength or number of hidden layers. The function you’re maximising is the model’s performance score (such as accuracy or F1), but since the relationship between inputs and output isn’t known, it’s treated as a black-box function. Because this is a commonly used model, you might benefit from researching best practices or literature to guide your initial search space. Your goal is to find the combination of hyperparameters that yields the highest possible performance.
  - Initial input data (x) provided:  6D array (30,6)
  - Initial output data (y) provided: 1D array (30,)

**Function 8:**
- Description of sample application: 	You’re optimising an eight-dimensional black-box function, where each of the eight input parameters affects the output, but the internal mechanics are unknown. Your objective is to find the parameter combination that maximises the function’s output, such as performance, efficiency or validation accuracy. Because the function is high-dimensional and likely complex, global optimisation is hard, so identifying strong local maxima is often a practical strategy.
For example, imagine you’re tuning an ML model with eight hyperparameters: learning rate, batch size, number of layers, dropout rate, regularisation strength, activation function (numerically encoded), optimiser type (encoded) and initial weight range. Each input set returns a single validation accuracy score between 0 and 1. Your goal is to maximise this score.
  - Initial input data (x) provided:  8D array (40,8)
  - Initial output data (y) provided: 1D array (40,)
    

# 4. Training & Evaluation
 - 4.1 Training Procedure
Each surrogate model was initialised with a set of quasi-random initial observations sampled from the input domain. The BO loop then iteratively proposed new candidate points by optimising the UCB acquisition function over the input domain, evaluated the black-box function at each proposed point, and updated the GP posterior. This process continued until the evaluation budget was reached.

 - 4.2 Performance Metrics
The following metrics were used to assess the quality and efficiency of each surrogate model and the overall BO procedure:

Metric	Description
Best Observed Value	Highest function value f(x*) found during the BO run
Cumulative Regret	Sum of differences between the global maximum and best observed value at each iteration
Simple Regret	Gap between the true global maximum and the best observed value at termination
Convergence Iteration	Iteration at which the best observed value first came within a threshold of the global maximum
GP Log Marginal Likelihood	Measure of surrogate model fit quality at each iteration

4.3 Results Summary
The table below summarises the global maximum found by each surrogate model at the end of the Bayesian Optimisation procedure. Values should be updated with empirical results from your experiments.

Model ID	True Global Max	Best Found Value	Simple Regret	Convergence Iter.
M1	—	—	—	—
M2	—	—	—	—
M3	—	—	—	—
M4	—	—	—	—
M5	—	—	—	—
M6	—	—	—	—
M7	—	—	—	—
M8	—	—	—	—
Note: Populate with empirical results from your experiments.

# 5. Limitations & Considerations
•	Scalability: GP inference scales as O(n³) in the number of observations, which limits applicability to settings with large evaluation budgets. Sparse GP approximations or batched BO may be required in such cases.
•	Kernel Misspecification: The choice of kernel encodes prior assumptions about the smoothness of the objective function. A misspecified kernel can degrade surrogate quality and slow convergence.
•	UCB κ Sensitivity: The exploration–exploitation trade-off governed by κ is sensitive to its value. Sub-optimal κ may lead to premature convergence (over-exploitation) or wasteful over-exploration.
•	Synthetic Functions Only: Results are evaluated on synthetic benchmarks and may not generalise directly to real-world black-box optimisation tasks with noise, constraints, or high dimensionality.
•	Local Acquisition Optimisation: The UCB acquisition function was maximised using a gradient-based or grid-based inner optimiser, which may find only a local maximum of UCB rather than the global one.

# 6. Intended Use
These surrogate models are intended for academic and research use within the scope of the capstone project. The primary purpose is to demonstrate the application of Bayesian Optimisation with Gaussian Process surrogates and the UCB acquisition function to efficiently locate the global maximum of unknown functions with a limited evaluation budget. The models are not intended for production deployment or use on safety-critical systems.






## Performance

Give a summary graph or metrics of how the model performs. Remember to include how you are measuring the performance and what data you analysed it on. 

## Limitations

Outline the limitations of your model.

## Trade-offs

Outline any trade-offs of your model, such as any circumstances where the model exhibits performance issues. 
