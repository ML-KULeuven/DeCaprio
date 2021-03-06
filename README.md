# DeCaprio
This repository contains the code for our paper:
*Bleukx I, Berden S et al. (2022): Model-based algorithm configuration with adaptive capping and prior distributions, CPAIOR 2022*

## Overview of the code
Different scoring functions are implemented to work with our DeCaprio algorithm based on SMBO with adaptive capping.
The code is structured as follows:

```bash
.
├── __init__.py                 
├── algorithms.py                 # DeCaprio algorithm
├── compare_scorers.py            # Experiment comparing scoring functions with a uniform prior
├── compare_scorers_prior.py      # Experiment comparing scoring functions with an informed prior
├── compute_pseudocounts.py       # Code for calculating pseudocounts from precomputed grid search data
├── cpmpy_models                  # Directory with all CPMpy models
├── parameters.py                 # Parameter configurations for Google's OR-tools' CP-SAT solver
├── saved_solver.py               # Solver mimicing a CPMpy.SolverInterface
├── scorers       
│   ├── __init__.py
│   ├── baseline.py               # Random uniform scorer
│   ├── beta.py                   # Beta model and scoring function 
│   ├── dirichlet.py              # Dirichlet model and scoring function
│   ├── hamming.py                # Hamming model and scoring function
│   ├── hamming_dirichlet.py      # Hamming model and scoring function with Dirichlet tie-breaking
│   ├── score.py                  # Model and scoring function base class
│   └── simple_prior.py           # Sample prior and Sorted prior models and scoring function
└── utils.py                      # Utility functions for computing config ids
```

Each class contained in the *scorers* directory represents both a surrogate model and the associated scoring function which is used to rank hyperparameter configurations. <br>
Each experiment file comparing scorers outputs multiple dataframes, one for every problem instance. These dataframes contain the hyperparameter configurations in the order they were tried during the search. Additionally, the dataframe contains wallclock times and the runtimes of the tested configurations.

## Two-phase experiments
In each iteration of the DeCaprio algorithm, a problem instance must be solved using a certain hyperparameter configuration. This step is typically slow compared to the rest of the algorithm, which consists of updating the surrogate model and ranking the remaining hyperparameter configurations using the scoring function. This means that when multiple scorers are compared in an experiment, the same problem instance is solved with the same hyperparameter configuration multiple times (once for each scorer in the comparison). This needlessly slows down the experiments considerably.  

To prevent this slow-down, we performed our experiments in two phases. In the first phase, a solve call is executed for each combination of a hyperparameter configuration and problem instance, and all the resulting runtimes are saved in a pandas dataframe. In the second phase, the different scorers are compared, but instead of performing actual solver calls, the dataframe computed in phase 1 is used to simulate the solver calls with. A solve call then simply corresponds to a runtime look-up in the dataframe. This significantly speeds up the experiments.

The *compare_scorers* experiment can be executed with or without the use of the precomputed dataframe; this can be controlled by setting the *use_precomputed_data* Boolean variable accordingly.

## Getting the precomputed pandas dataframe
To download the dataframe, use the following command:
```console
wget -O grid_search.pickle https://rdr.kuleuven.be/api/access/datafile/163?gbrecs=true
```

The data is structured as a pandas dataframe and contains 5 runtimes (one per seed) for every configuration on every problem instance found in [cpmpy_models/](/cpmpy_models)
Every configuration is assigned an id based on the hyperparameter values it contains. To convert from config ids to actual hyperparameter configuraions, use the helper functions found in [utils.py](/utils.py)

All runtimes were obtained using OR-Tools' CP-SAT solver on an Intel(R) Xeon(R) Silver 4214 CPU with the number of threads per problem instance limited to 1.

All solver calls were capped on 105% of the default configuration's runtime. This does not affect the search in phase 2 of the experiments, as the first runtime cap in the search is always equal to the default configuration's runtime.
