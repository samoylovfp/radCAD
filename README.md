# radCAD
A [cadCAD](cadcad.org/) implementation in Rust, using PyO3 to generate Rust bindings for Python to be used as a native Python module. The performance and expressiveness of Rust, with the utility of the Python data-science stack.

Goals:
* simple API for ease of use
* performance driven (more speed = more experiments, larger parameter sweeps, in less time)
* cadCAD compatible (standard functions, data structures, and simulation results)
* maintainable, testable codebase

See https://github.com/cadCAD-org/cadCAD

## Features

* [x] Parameter sweeps

```python
params = {
    'a': [1, 2, 3],
    'b': [1]
}
```

* [x] Monte Carlo runs

```python
RUNS = 100
Simulation(model=model_a, timesteps=TIMESTEPS, runs=RUNS)
```

* [x] A/B tests

```python
model_a = Model(initial_state=states_a, psubs=psubs_a, params=params_a)
model_b = Model(initial_state=states_b, psubs=psubs_b, params=params_b)

simulation_1 = Simulation(model=model_a, timesteps=TIMESTEPS, runs=RUNS)
simulation_2 = Simulation(model=model_b, timesteps=TIMESTEPS, runs=RUNS)

data = rc.run([simulation_1, simulation_2])
```

* [x] cadCAD compatibility
* [x] cadCAD simulation data structure

```bash
               a          b  simulation  subset  run  substep  timestep
0       1.000000        2.0           0       0    1        0         0
1       0.540302        2.0           0       0    1        1         1
2       0.540302        7.0           0       0    1        2         1
3       0.463338        7.0           0       0    1        1         2
4       0.463338       12.0           0       0    1        2         2
...          ...        ...         ...     ...  ...      ...       ...
799999  0.003162   999982.0           1       1    1        2     99998
800000  0.003162   999982.0           1       1    1        1     99999
800001  0.003162   999992.0           1       1    1        2     99999
800002  0.003162   999992.0           1       1    1        1    100000
800003  0.003162  1000002.0           1       1    1        2    100000
```

## Development

```bash
make release
# or
make release-macos
```

## Use

```python
import output.rad_cad as rc
from output.rad_cad import Model, Simulation

TIMESTEPS = 100_000
RUNS = 1

model_a = Model(initial_state=states_a, psubs=psubs_a, params=params_a)
model_b = Model(initial_state=states_b, psubs=psubs_b, params=params_b)

simulation_1 = Simulation(model=model_a, timesteps=TIMESTEPS, runs=RUNS)
simulation_2 = Simulation(model=model_b, timesteps=TIMESTEPS, runs=RUNS)

data = rc.run([simulation_1, simulation_2])
df = pd.DataFrame(data)
```

## Benchmark

**Note:** Not conclusive, needs multiple runs and averages taken.

```bash
radCAD took 5.737071990966797 seconds
cadCAD took 22.206645011901855 seconds
Simulation output dataframes are carbon copies
Rust is 3.8707279683550997X faster than Python
```

See `benchmark.py` and `benchmark.ipynb`.

```python
import math

def policy(params, substep, state_history, previous_state):
    return {'step_size': 1}

def update_a(params, substep, state_history, previous_state, policy_input):
    a = b = c = d = e = 100.0
    return 'a', previous_state['a'] * abs(math.cos(previous_state['a']))

def update_b(params, substep, state_history, previous_state, policy_input):
    return 'b', previous_state['b'] + policy_input['step_size'] * params['a']

params = {
    'a': [1, 2, 3],
    'b': [1]
}

states = {
    'a': 1.0,
    'b': 2.0
}

psubs = [
    {
        'policies': {},
        'variables': {
            'a': update_a
        }
    },
    {
        'policies': {
            'p_1': policy,
            'p_2': policy,
            'p_3': policy,
            'p_4': policy,
            'p_5': policy,
        },
        'variables': {
            'b': update_b
        }
    }
]

TIMESTEPS = 100_000
RUNS = 1
```

### cadCAD
```bash
Execution Mode: local_proc
Configuration Count: 2
Dimensions of the first simulation: (Timesteps, Params, Runs, Vars) = (100000, 2, 2, 7)
Execution Method: local_simulations
SimIDs   : [0, 0, 1, 1]
SubsetIDs: [0, 1, 0, 1]
Ns       : [0, 1, 0, 1]
ExpIDs   : [0, 0, 1, 1]
Execution Mode: parallelized
Total execution time: 22.19s
22.206645011901855
```

### radCAD

```bash
5.737071990966797 seconds
```
