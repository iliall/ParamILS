# ParamILS Implementation Plan for OR-Tools Configuration

## Overview
ParamILS (Parameter Iterated Local Search) in C++ for automatically configuring OR-Tools constraint programming solver parameters. The implementation will include both BasicILS and FocusedILS variants with multi-threaded evaluation support.

## Requirements Summary
- **Variants**: BasicILS and FocusedILS
- **Integration**: Integration with OR-Tools (Don't sure about this for now!)
- **Parallelism**: Distributed run using `cbrun` (Making notes that number of threads affects the algorithm output)
- **Parameter Types**: Categorical, ordinal, integer, continuous (discretized), conditional
- **Instance Format**: Protocol Buffers
- **Config Formats**: This most likely depends on how we integrate OR-Tools???

## Implementation

### Phase 1: Project Foundation
**Goal**: Set up project structure, build system, and core abstractions

#### Part 1.1: Project Setup and Build System

**Description**: Initialize the C++ project with CMake build system and configure all dependencies.

**Subtasks**:
- [ ] CMake project compiles successfully
- [ ] OR-Tools linked and importable
- [ ] Directory structure created

**Implementation Details**:
```
paramils/
├── CMakeLists.txt
├── src/
│   ├── core/
│   ├── algorithms/
│   ├── evaluation/
│   ├── integration/
│   ├── cli/
│   └── worker/
├── include/paramils/
├── tests/
├── configs/
└── examples/
```

**Build Targets**:
- `paramils` - Main CLI for running configuration
- `paramils_worker` - Worker binary executed by cbrun/srun

---

#### Part 1.2: Core Data Types and Utilities

**Description**: Implement foundational utilities used throughout the codebase.

**Acceptance Criteria**:
- [ ] `RandomNumberGenerator` class with reproducible seeds
- [ ] `Timer` class for timing
- [ ] `Statistics` utilities (mean, median, stddev)
- [ ] Logging infrastructure (configurable verbosity)
- [ ] JSON serialization helpers

**Files to Create**:
- `include/paramils/core/rng.hpp`
- `include/paramils/core/timer.hpp`
- `include/paramils/core/statistics.hpp`
- `include/paramils/core/logging.hpp`
- `src/core/rng.cpp`
- `src/core/timer.cpp`

---

### Phase 2: Parameter Space Representation
**Goal**: Define how algorithm parameters and configurations are represented

#### Part 2.1: Parameter Types Implementation

**Description**: Implement the parameter class hierarchy supporting all required parameter types.

**Acceptance Criteria**:
- [ ] Base `Parameter` class with common interface
- [ ] `CategoricalParameter` for discrete unordered choices
- [ ] `OrdinalParameter` for ordered discrete values
- [ ] `IntegerParameter` with range and discretization
- [ ] `ContinuousParameter` with range, discretization, log-scale support
- [ ] Conditional parameter dependencies

**Files to Create**:
- `include/paramils/core/parameter.hpp`
- `src/core/parameter.cpp`

---

#### Part 2.2: Configuration Space Implementation

**Description**: Implement Configuration and ConfigurationSpace classes for managing parameter configurations.

**Acceptance Criteria**:
- [ ] `Configuration` class with value storage and validation
- [ ] Configuration hashing for cache keys
- [ ] `ConfigurationSpace` with all parameters
- [ ] Random configuration generation
- [ ] One-exchange neighborhood generation
- [ ] Conditional parameter handling in neighborhoods

---

#### Part 2.3: Parameter File Parser

**Description**: Implement parser for parameter specification files.

**Acceptance Criteria**:
- [ ] YAML/JSON parser for parameter definitions
- [ ] Support all parameter types
- [ ] Conditional parameter syntax support
- [ ] Validation with clear error messages
- [ ] Example parameter file for OR-Tools

---

### Phase 3: Instance Management
**Goal**: Handle problem instances for training and testing

#### Part 3.1: Instance Abstraction

**Description**: Implement classes for managing problem instances.

**Acceptance Criteria**:
- [ ] `Instance` class with path and metadata
- [ ] `InstanceSet` for collections of instances
- [ ] Load instances from directory
- [ ] Train/test split functionality
- [ ] Shuffling with reproducible RNG

---

#### Ticket 3.2: Run Specification

**Description**: Define data structures for specifying and recording algorithm runs.

**Acceptance Criteria**:
- [ ] `RunSpec` struct with all run parameters
- [ ] `RunResult` struct with outcomes
- [ ] Serialization for caching

---

### Phase 4: Target Algorithm Integration
**Goal**: Integrate with OR-Tools for running evaluations

#### Part 4.1: Target Algorithm Interface

**Description**: Define abstract interface for target algorithms.

**Acceptance Criteria**:
- [ ] Abstract `TargetAlgorithm` base class
- [ ] Support for timeout/interruption
- [ ] Capture runtime and solution status

---

#### Part 4.2: OR-Tools Wrapper Implementation

**Description**: Implement OR-Tools CP-SAT solver wrapper.

**Acceptance Criteria**:
- [ ] Map configuration values to CP-SAT parameters
- [ ] Proper timeout handling
- [ ] Thread-safe (each instance independent)
- [ ] Support all relevant OR-Tools parameters

---

#### Ticket 4.3: Instance Loader for OR-Tools

**Description**: Implement Protocol Buffer instance loader.

**Acceptance Criteria**:
- [ ] Load `.pb` (binary) format
- [ ] Load `.pbtxt` (text) format
- [ ] Handle file not found gracefully
- [ ] Validate model structure

---

### Phase 5: Evaluation Framework
**Goal**: Multi-threaded evaluation with result caching

#### Part 5.1: Run Cache Implementation

**Description**: Implement thread-safe cache for run results.

**Acceptance Criteria**:
- [ ] Thread-safe get/put operations
- [ ] Key based on (config_hash, instance_id, seed)
- [ ] Save/load cache to disk
- [ ] Memory-efficient storage

---

#### Part 5.2: Distributed Execution via cbrun/srun

**Description**: Implement distributed evaluation using Cerebras `cbrun` (Slurm `srun` wrapper).

**Acceptance Criteria**:
- [ ] Submit evaluation jobs via `cbrun -- srun -c <cores> <command>`
- [ ] Batch multiple evaluations into job arrays
- [ ] Poll for job completion
- [ ] Collect results from output files
- [ ] Handle job failures and timeouts
- [ ] Configurable parallelism level

---

#### Part 5.3: Evaluator Implementation

**Description**: Implement main evaluator class coordinating algorithm runs via cbrun.

**Acceptance Criteria**:
- [ ] Evaluate configuration on multiple instances
- [ ] Use CbrunExecutor for distributed evaluation
- [ ] Use cache to avoid redundant runs
- [ ] Support evaluation bounds (for capping)
- [ ] Blocking strategy (same instances/seeds)
- [ ] Batch submissions for efficiency

---

### Phase 6: Cost Statistics
**Goal**: Implement cost computation and comparison

#### Part 6.1: Cost Function Implementation

**Description**: Implement cost aggregation functions.

**Acceptance Criteria**:
- [ ] Abstract `CostFunction` interface
- [ ] `PenalizedAverageRuntime` (PAR) implementation
- [ ] Support for median, quantile aggregation
- [ ] Configurable penalty multiplier

---

#### Part 6.2: Configuration Comparison

**Description**: Implement configuration comparison utilities.

**Acceptance Criteria**:
- [ ] Compare two configurations based on cached results
- [ ] Handle ties consistently
- [ ] Support comparison with different N values

---

### Phase 7: BasicILS Implementation
**Goal**: Implement the simpler BasicILS variant

#### Ticket 7.1: BasicILS Core Algorithm

**Description**: Implement the BasicILS algorithm as described in the ParamILS paper.

**Acceptance Criteria**:
- [ ] Initialization with r random configurations
- [ ] Iterative first improvement local search
- [ ] Perturbation with s random moves
- [ ] Acceptance criterion (better or equal)
- [ ] Random restart with probability p_restart
- [ ] Time budget termination
- [ ] Track incumbent throughout

---

#### Part 7.2: BasicILS Logging and Monitoring

**Description**: Add comprehensive logging and monitoring to BasicILS.

**Acceptance Criteria**:
- [ ] Log each ILS iteration
- [ ] Track configurations evaluated
- [ ] Track incumbent trajectory over time
- [ ] Report time breakdown (evaluation vs search)
- [ ] Configurable verbosity levels

---

### Phase 8: Adaptive Capping
**Goal**: Implement early termination of poor configurations

#### Part 8.1: Trajectory-Preserving Capping

**Description**: Implement trajectory-preserving adaptive capping.

**Acceptance Criteria**:
- [ ] Compute lower bound incrementally during evaluation
- [ ] Terminate when lower bound exceeds bound
- [ ] Preserve exact search trajectory (same as no capping)
- [ ] Return appropriate cost for capped configurations

---

#### Ticket 8.2: Aggressive Capping

**Description**: Implement aggressive capping with bound multiplier.

**Acceptance Criteria**:
- [ ] Bound = bm * incumbent_cost (default bm = 2)
- [ ] Track instances solved when capped
- [ ] Compare capped configs by instances solved
- [ ] Configurable bound multiplier

---

### Phase 9: FocusedILS Implementation
**Goal**: Implement the adaptive FocusedILS variant

#### Part 9.1: Domination Logic

**Description**: Implement domination checking for FocusedILS.

**Acceptance Criteria**:
- [ ] Track N(θ) - number of runs per configuration
- [ ] Implement domination check
- [ ] Compare based on same number of runs

---

#### Part 9.2: FocusedILS Core Algorithm

**Description**: Implement the FocusedILS algorithm.

**Acceptance Criteria**:
- [ ] Adaptive comparison (add runs until domination)
- [ ] Bonus runs for improving configurations
- [ ] Track B (configs since last improvement)
- [ ] Maintain invariant: N(incumbent) >= N(θ)
- [ ] Convergence to optimal (probabilistic)

---

#### Ticket 9.3: FocusedILS with Capping

**Description**: Integrate adaptive capping into FocusedILS.

**Acceptance Criteria**:
- [ ] Separate bounds per number of runs N
- [ ] Update bounds as incumbent improves
- [ ] Both TP and aggressive capping support

---

### Phase 10: CLI and Configuration
**Goal**: Command-line interface and configuration management

#### Part 10.1: Command-Line Interface

**Description**: Implement command-line interface for ParamILS.

**Acceptance Criteria**:
- [ ] Parse all algorithm parameters
- [ ] Validate inputs
- [ ] Support both BasicILS and FocusedILS
- [ ] Help text and usage examples

---

#### Part 10.2: Results Output

**Description**: Implement results output and export.

**Acceptance Criteria**:
- [ ] Output best configuration found
- [ ] Incumbent trajectory over time
- [ ] Export to JSON, CSV, parameter file
- [ ] Include run metadata

---

### Phase 11: Testing
**Goal**: Comprehensive test coverage

#### Part 11.1: Unit Tests

**Description**: Implement unit tests for core components.

**Test Coverage**:
- [ ] Parameter types and validation
- [ ] Configuration and ConfigurationSpace
- [ ] Neighborhood generation
- [ ] Conditional parameters
- [ ] Cost functions (PAR, median)
- [ ] Capping logic
- [ ] Domination logic
- [ ] RNG reproducibility

---

#### Part 11.2: Integration Tests

**Description**: Implement integration tests with OR-Tools.

**Test Coverage**:
- [ ] OR-Tools wrapper with simple problems
- [ ] Full BasicILS run on toy problem
- [ ] Full FocusedILS run on toy problem
- [ ] Reproducibility with fixed seeds
- [ ] Multi-threaded evaluation

---

#### Part 11.3: Validation Tests

**Description**: Validate algorithm correctness.

**Test Coverage**:
- [ ] Synthetic benchmark with known optimal
- [ ] Verify ParamILS finds optimal
- [ ] Compare BasicILS vs FocusedILS
- [ ] Verify TP capping preserves trajectory

---
