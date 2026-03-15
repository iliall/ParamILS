# ParamILS Implementation Plan

## Overview
Implementing ParamILS in C++ for automatically configuring OR-Tools constraint programming solver parameters. The implementation will include both BasicILS and FocusedILS.

## Requirements Summary
- **Algorithms**: BasicILS and FocusedILS
- **Integration**: Integration with OR-Tools (Not sure about this for now!)
- **Parallelism**: Distributed run using `cbrun` (Making note that number of threads affects the algorithm output)
- **Parameter Types**: Categorical, ordinal, integer, continuous (discretized), conditional`
- **Instance Format**: Protocol Buffers
- **Config Formats**: This most likely depends on how we integrate OR-Tools???

## Implementation

### Phase 1: Initial Project Structure
**Goal**: Set up project structure, build system, and core abstractions

#### Part 1.1: Project Setup and Build System

**Description**: Initializing the C++ project with CMake build system and configuring all dependencies.

**Subtasks**:
- [ ] CMake project compiles successfully
- [ ] OR-Tools linked and importable
- [ ] Directory structure created

---

### Phase 2: Parameter Space Representation
**Goal**: Defining how algorithm parameters and configurations are represented

#### Part 2.1: Parameter Types Implementation

**Description**: Implementing the parameter class hierarchy supporting all required parameter types.

**Acceptance Criteria**:
- [ ] Base `Parameter` class with common interface
- [ ] Separate class implementation for each of the parameter types

---

#### Part 2.2: Configuration Space Implementation

**Description**: Implementing Configuration and classes for managing parameter configurations.

**Acceptance Criteria**:
- [ ] `Configuration` class with value storage and validation
- [ ] `ConfigurationSpace` with all parameters
- [ ] Random configuration generation
- [ ] One-exchange neighborhood generation
- [ ] Conditional parameter handling in neighborhoods (The implementation is mentioned in the paper)

---

#### Part 2.3: Parameter File Parser

**Description**: Parser for parameter specification files. (Dependant on how OR-Tools gets configured to the project!)

**Acceptance Criteria**:
- [ ] parser for parameter definitions
- [ ] Support all parameter types
- [ ] Conditional parameter syntax support

---

### Phase 3: Instance Management
**Goal**: Handle problem instances for training and testing

#### Part 3.1: Instance Abstraction

**Description**: Implementing classes for managing problem instances.

**Acceptance Criteria**:
- [ ] `Instance` class with path and metadata
- [ ] `InstanceSet` for collections of instances
- [ ] Load instances from directory

---

#### Part 3.2: Run Specification

**Description**: Define data structures for specifying and recording algorithm runs.

**Acceptance Criteria**:
- [ ] `RunSpec` struct with all run parameters
- [ ] `RunResult` struct with outcomes

---

### Phase 4: Target Algorithm Integration
**Goal**: Integrating OR-Tools for running evaluations

#### Part 4.1: Target Algorithm Interface

**Description**: Define abstract interface for target algorithms.

**Acceptance Criteria**:
- [ ] Abstract `TargetAlgorithm` base class
- [ ] Support for timeout/interruption
- [ ] Capture runtime and solution status

---

### Phase 5: Cost Statistics
**Goal**: Implementing cost computation and comparison

#### Part 5.1: Cost Function Implementation

**Description**: Implementing cost aggregation functions.

**Acceptance Criteria**:
- [ ] Abstract `CostFunction` interface
- [ ] `PenalizedAverageRuntime` (PAR) implementation
- [ ] Support for median, quantile aggregation
- [ ] Configurable penalty multiplier

---

#### Part 5.2: Configuration Comparison

**Description**: Implement configuration comparison utilities.

**Acceptance Criteria**:
- [ ] Compare two configurations (Cache results?)
- [ ] Handle ties consistently
- [ ] Support comparison with different N values

---

### Phase 6: BasicILS Implementation
**Goal**: Implement the simpler BasicILS variant

#### Part 6.1: BasicILS Core Algorithm

**Description**: Implementing the BasicILS algorithm.

**Acceptance Criteria**:
- [ ] Initialization with r random configurations
- [ ] Iterative first improvement local search
- [ ] Perturbation with s random moves
- [ ] Acceptance criterion (better or equal)
- [ ] Random restart with probability p_restart
- [ ] Time budget termination
- [ ] Track incumbent throughout

---

### Phase 7: Adaptive Capping
**Goal**: Implementing early termination of bad configurations

#### Part 7.1: Trajectory-Preserving Capping

**Description**: Implementing trajectory-preserving adaptive capping.

**Acceptance Criteria**:
- [ ] Compute lower bound incrementally during evaluation
- [ ] Terminate when lower bound exceeds bound
- [ ] Preserve exact search trajectory (same as no capping)
- [ ] Return appropriate cost for capped configurations

---

#### Part 7.2: Aggressive Capping

**Description**: Implementing aggressive capping with bound multiplier.

**Acceptance Criteria**:
- [ ] Bound = bm * incumbent_cost (There should be some default bm like 2 which has been specified in the paper)
- [ ] Track instances solved when capped
- [ ] Compare capped configs by instances solved
- [ ] Configurable bound multiplier

---

### Phase 8: FocusedILS Implementation
**Goal**: The adaptive FocusedILS variant

#### Part 8.1: Domination Logic

**Description**: Implement domination checking for FocusedILS.

**Acceptance Criteria**:
- [ ] Track N(θ) - number of runs per configuration
- [ ] Implement domination check
- [ ] Compare based on same number of runs (Same N)

---

#### Part 8.2: FocusedILS Core Algorithm

**Description**: Implement the FocusedILS algorithm.

**Acceptance Criteria**:
- [ ] Adaptive comparison (add runs until domination)
- [ ] Track B (configs since last improvement)
- [ ] Maintain invariant: N(incumbent) >= N(θ)
- [ ] Convergence to optimal (Probabilistic?)

---

#### Ticket 8.3: FocusedILS with Capping

**Description**: Integrate adaptive capping into FocusedILS.

**Acceptance Criteria**:
- [ ] Separate bounds per number of runs N
- [ ] Update bounds as incumbent improves
- [ ] Both TP and aggressive capping support

---

### Phase 9: CLI and Configuration
**Goal**: Command-line interface and configuration management

#### Part 9.1: Command-Line Interface

**Acceptance Criteria**:
- [ ] Parse all algorithm parameters
- [ ] Validate inputs

---

### Phase 10: Testing
**Goal**: Test coverage
