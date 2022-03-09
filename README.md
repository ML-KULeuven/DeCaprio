# DeCaprio
This repository contains the code for reproducing experiments found in our paper:
*Bleukx I, Berden S et al. (2022): Model-based algorithm configuration with adaptive capping and prior distributions, CPAIOR 2022*

## Overview of the code
Different scoring functions are implemented to work with the our DeCaprio algorithm based on SMBO with adaptive cappig.
The code is structured as follows:

```bash
.
├── __init__.py                 
├── algorithms.py                 # DeCaprio algorithm
├── compare_scorers.py            # Experiment for comparing scoring function without prior
├── compare_scorers_prior.py      # Experiment for comparing scoring functions with prior
├── compute_pseudocounts.py       # Code for calculaing pseudocounts from grid search data
├── cpmpy_models                  # Direcotry with all CPMpy models
├── parameters.py                 # Parameter configurations for Google's OR-tools solver
├── saved_solver.py               # Solver mimicing a CPMpy.SolverInterface
├── scorers       
│   ├── __init__.py
│   ├── baseline.py               # Random uniform scorer
│   ├── beta.py                   # Beta scorer 
│   ├── dirichlet.py              # Dirichlet scorer
│   ├── hamming.py                # Hamming scorer  
│   ├── hamming_dirichlet.py      # Hamming scorer with Dirichlet tie-breaking
│   ├── score.py                  # Scorer base class
│   └── simple_prior.py           # Sample prior and Sorted prior scorers
└── utils.py                      # Utility functions for computing config ids
```

An instance of a Scorer is used to rank all hyperparameter configurations based on previous knowledge during the search. <br>
The output of both experiment files comparing scorers is a dataframe for every model. These dataframes contain the tested configuration in order during the search. For convenience, the dataframe also contains metadata of the search itself such as wallclock time and the runtime of a tested configuration.

### Experiments
In each iteration of the algorithm, a problem instance must be solved using a certain candidate hyperparameter configuration. This step is slow compared to the rest of the algorithm, which consists of updating the model. This means that when multiple scoring functions are compared in an experiment, the same problem instance is solved with the same hyperparameter configuration multiple times (once for each scoring function in the comparison). This slows down experiments considerably.  

To prevent this needless slow-down, we performed our experiments in two phases. In the first phase, we executed a solve call for each combination of a hyperparameter configuration and problem instance, saving all the runtimes in a pandas dataframe. In the second phase, this dataframe was used to simulate solver calls with, speeding up the comparison of the scoring functions. A solve call then simply corresponds to a runtime look-up in the dataframe.

The experiments can be executed with or without the use of the precomputed dataframe; this can be controlled by setting the use_precomputed_dataframe Boolean accordingly.

### Getting the precomputed pandas dataframe
The data is structured as a pandas dataframe and contains the runtime for every configuration on every model found in [cpmpy_models/](/cpmpy_models)
All configuration runtimes were capped on 105% of the default configuration's runtime.
All runtimes were obtained using an Intel(R) Xeon(R) Silver 4214 CPU with the number of threads per problem instance limited to 1.
For every instance-hyperparameter configuration combination, 5 seperate runs of the OR-tools solver were performed.

Every configuration is assigned an id based on the hyperparameter values it contains. To convert from config ids to actual hyperparameter configuraions, use the helper functions found in [utils.py](/utils.py)


To download the dataframe, use the following command:
```console
wget -O grid_search.pickle https://www.dropbox.com/s/w4dn19p31cg7g5r/grid_search.pickle?dl=1
```
