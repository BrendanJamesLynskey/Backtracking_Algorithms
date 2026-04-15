# Backtracking Algorithms

**Computer Science Fundamentals Series**

State space trees · N-Queens · Sudoku · Constraint satisfaction · Pruning · Branch and bound

*Mid-level software engineer track -- 20 slides*

---

## Table of Contents

1. [What Is Backtracking?](#slide-02--what-is-backtracking)
2. [State Space Trees](#slide-03--state-space-trees)
3. [The Backtracking Template](#slide-04--the-backtracking-template)
4. [N-Queens Problem](#slide-05--n-queens-problem)
5. [N-Queens -- Pruning in Action](#slide-06--n-queens--pruning-in-action)
6. [Sudoku Solver](#slide-07--sudoku-solver)
7. [Subset Sum](#slide-08--subset-sum)
8. [Permutations](#slide-09--permutations)
9. [Combinations](#slide-10--combinations)
10. [Graph Colouring](#slide-11--graph-colouring)
11. [Hamiltonian Path & Cycle](#slide-12--hamiltonian-path--cycle)
12. [Constraint Satisfaction Problems](#slide-13--constraint-satisfaction-problems)
13. [CSP Solving Strategies](#slide-14--csp-solving-strategies)
14. [Pruning Strategies](#slide-15--pruning-strategies)
15. [Optimisation Techniques](#slide-16--optimisation-techniques)
16. [Branch and Bound](#slide-17--branch-and-bound)
17. [Backtracking vs Brute Force](#slide-18--backtracking-vs-brute-force)
18. [Classic Problem Complexities](#slide-19--classic-problem-complexities)
19. [Summary & Further Reading](#slide-20--summary--further-reading)

---

## Slide 02 -- What Is Backtracking?

### The paradigm

Backtracking is a systematic method for exploring all potential solutions by building candidates incrementally and **abandoning** ("backtracking" from) a candidate as soon as it is determined that it cannot lead to a valid solution.

### Core idea

- Build a solution one decision at a time
- At each step, check constraints — if the partial solution violates any constraint, **prune** this branch
- If a dead end is reached, undo the last decision and try the next option
- If all options exhausted at a level, backtrack further

### When to use backtracking

- Constraint satisfaction problems (CSPs)
- Combinatorial search — permutations, combinations, subsets
- Puzzle solving — Sudoku, crosswords, N-Queens
- Graph problems — colouring, Hamiltonian paths
- Optimisation problems (with branch and bound extension)

> **Key insight:** backtracking turns exponential brute force into something practical by cutting off invalid branches early. The earlier you prune, the faster the search.

---

## Slide 03 -- State Space Trees

### Definition

A **state space tree** is a rooted tree that represents all possible states of a backtracking algorithm. Each node represents a partial solution, and edges represent decisions.

### Structure

- **Root** — empty solution (no decisions made)
- **Internal nodes** — partial solutions with some decisions made
- **Leaf nodes** — complete candidates (valid solutions or dead ends)
- **Pruned branches** — subtrees skipped because constraints are already violated

### Example: binary subsets of {a, b, c}

```
                  {}
           /             \
        {a}               {}
       /    \           /    \
    {a,b}   {a}      {b}     {}
    / \     / \      / \    / \
{a,b,c} {a,b} {a,c} {a} {b,c} {b} {c} {}
```

### Key properties

- Depth = number of decisions
- Branching factor = number of choices per decision
- Without pruning: tree has `b^d` leaves (b = branching factor, d = depth)
- With pruning: many subtrees are never explored

---

## Slide 04 -- The Backtracking Template

### Generic pseudocode

```python
def backtrack(state, decisions):
    if is_solution(state):
        record_solution(state)
        return

    for choice in get_choices(state, decisions):
        if is_valid(state, choice):       # pruning check
            apply(state, choice)          # make the decision
            backtrack(state, decisions)   # recurse
            undo(state, choice)           # backtrack
```

### The three pillars

- **is_valid()** — the pruning function; rejects partial solutions that cannot lead to a valid complete solution
- **apply() / undo()** — state mutation and reversal; the "choose / unchoose" pattern
- **is_solution()** — recognises when a complete, valid solution has been built

### Recursion vs iteration

- Recursive backtracking uses the call stack to store state — natural and readable
- Iterative backtracking uses an explicit stack — avoids stack overflow for very deep searches
- Both have identical time complexity; iterative can save constant-factor memory

> **Pattern recognition:** if a problem asks "find all configurations satisfying constraints" or "does a valid arrangement exist", backtracking is the first technique to consider.

---

## Slide 05 -- N-Queens Problem

### Problem statement

Place N queens on an N x N chessboard so that no two queens attack each other — no shared row, column, or diagonal.

### Strategy

- Place queens one row at a time (row 0, row 1, ..., row N-1)
- For each row, try each column position
- Check constraints: no column conflict, no diagonal conflict
- If no valid column exists in the current row, backtrack to the previous row

### Constraint checking

```python
def is_safe(board, row, col):
    for prev_row in range(row):
        prev_col = board[prev_row]
        if prev_col == col:                    # same column
            return False
        if abs(prev_col - col) == row - prev_row:  # same diagonal
            return False
    return True
```

### Solution counts

| N | Solutions | Unique (up to symmetry) |
|---|----------|------------------------|
| 4 | 2 | 1 |
| 8 | 92 | 12 |
| 12 | 14,200 | 1,787 |
| 14 | 365,596 | 45,752 |

---

## Slide 06 -- N-Queens -- Pruning in Action

### Visualising the search

For 4-Queens, the algorithm explores far fewer than 4^4 = 256 leaf nodes:

```
Row 0: try col 0  ✓
  Row 1: try col 0  ✗ (column)
  Row 1: try col 1  ✗ (diagonal)
  Row 1: try col 2  ✓
    Row 2: try col 0  ✗ (diagonal)
    Row 2: try col 1  ✗ (column conflict with row 1)
    Row 2: try col 2  ✗ (column conflict with row 1)
    Row 2: try col 3  ✗ (diagonal)
    ← backtrack to row 1
  Row 1: try col 3  ✓
    Row 2: try col 0  ✗ (diagonal)
    Row 2: try col 1  ✓
      Row 3: try col 0  ✗
      Row 3: try col 1  ✗
      Row 3: try col 2  ✗
      Row 3: try col 3  ✗
      ← backtrack
    ...
```

### Efficiency

- Brute force: check all `N^N` arrangements — 4^4 = 256 for N=4
- Backtracking: typically explores only a small fraction of the tree
- For 8-Queens: ~114 nodes visited vs 16,777,216 brute-force arrangements

> **The power of pruning:** each invalid placement at row `r` eliminates an entire subtree of `N^(N-r-1)` nodes.

---

## Slide 07 -- Sudoku Solver

### Problem

Fill a 9x9 grid so every row, column, and 3x3 box contains digits 1-9 exactly once.

### Backtracking approach

```python
def solve_sudoku(board):
    cell = find_empty(board)
    if cell is None:
        return True                   # all cells filled — solved

    row, col = cell
    for num in range(1, 10):
        if is_valid_placement(board, row, col, num):
            board[row][col] = num     # place
            if solve_sudoku(board):
                return True
            board[row][col] = 0       # undo

    return False                      # trigger backtracking
```

### Optimisations

- **Naked singles** — if only one number is valid for a cell, place it immediately
- **Hidden singles** — if a number can only go in one cell in a row/col/box, place it
- **Constraint propagation** — reduce domains before branching (Arc Consistency)
- **Most Constrained Variable (MRV)** — fill the cell with fewest remaining options first

> Sudoku with constraint propagation solves most newspaper puzzles without any backtracking at all. Hard puzzles (17-clue minimum) still require search.

---

## Slide 08 -- Subset Sum

### Problem

Given a set S = {s1, s2, ..., sn} and a target T, find all subsets of S that sum to T.

### Example

S = {3, 7, 1, 8, 4}, T = 11

Solutions: {3, 8}, {7, 4}, {3, 4, 1, ...} — enumerate via backtracking.

### Backtracking solution

```python
def subset_sum(nums, target, start, current, result):
    if target == 0:
        result.append(current[:])
        return
    if target < 0:
        return                       # prune: overshot

    for i in range(start, len(nums)):
        current.append(nums[i])
        subset_sum(nums, target - nums[i], i + 1, current, result)
        current.pop()                # backtrack
```

### Pruning strategies

- **Sort the array first** — if `nums[i] > remaining target`, skip all subsequent elements (they are larger)
- **Skip duplicates** — if `nums[i] == nums[i-1]` and we are at the same recursion level, skip to avoid duplicate subsets
- **Running sum** — maintain remaining sum of unused elements; if `remaining_sum < target`, prune (no way to reach target)

> Subset sum is NP-complete. Backtracking with pruning is practical for moderate inputs (n < ~40). For larger inputs, dynamic programming or meet-in-the-middle may be more appropriate.

---

## Slide 09 -- Permutations

### Generate all permutations of [1, 2, ..., n]

```python
def permute(nums, start, result):
    if start == len(nums):
        result.append(nums[:])
        return

    for i in range(start, len(nums)):
        nums[start], nums[i] = nums[i], nums[start]   # choose
        permute(nums, start + 1, result)
        nums[start], nums[i] = nums[i], nums[start]   # unchoose
```

### State space

- Level 0: n choices for position 0
- Level 1: n-1 choices for position 1
- ...
- Total leaves: n! (all are valid — no pruning needed for plain permutations)

### Permutations with constraints

When constraints are added (e.g., "no two adjacent elements differ by more than 2"), pruning becomes effective:

- Check the constraint **at each level** before recursing
- Invalid placements prune entire subtrees
- Transforms O(n!) into something much smaller in practice

### Avoiding duplicates

For inputs with repeated elements (e.g., [1, 1, 2]):

- Sort the input first
- At each recursion level, skip element `i` if `nums[i] == nums[i-1]` and `i-1` was not used at this level

---

## Slide 10 -- Combinations

### C(n, k): choose k elements from n

```python
def combine(n, k, start, current, result):
    if len(current) == k:
        result.append(current[:])
        return

    # pruning: need (k - len(current)) more elements;
    # only proceed if enough remain
    for i in range(start, n - (k - len(current)) + 2):
        current.append(i)
        combine(n, k, i + 1, current, result)
        current.pop()
```

### Pruning the search space

Without pruning: explore all 2^n subsets, filter by size k.

With pruning: at each level, if there are not enough remaining elements to fill the combination, **stop immediately**.

### Combinations vs permutations

| Property | Permutations | Combinations |
|----------|-------------|-------------|
| Order matters | Yes | No |
| Count | n! / (n-k)! | n! / (k!(n-k)!) |
| Start index | Fixed (swap-based) | Advances (avoid revisiting) |
| Typical pruning | Constraint-based | Size-based + constraint |

> Combinations are a strict subset of the permutation search space. Using `start` index avoids generating [1,2] and [2,1] separately.

---

## Slide 11 -- Graph Colouring

### Problem

Assign colours to vertices of an undirected graph such that no two adjacent vertices share the same colour, using at most k colours.

### Backtracking approach

```python
def graph_colour(graph, k, colours, vertex):
    if vertex == len(graph):
        return True                    # all vertices coloured

    for c in range(1, k + 1):
        if is_safe_colour(graph, colours, vertex, c):
            colours[vertex] = c        # assign colour
            if graph_colour(graph, k, colours, vertex + 1):
                return True
            colours[vertex] = 0        # backtrack

    return False                       # no valid colour found
```

### Applications

- **Map colouring** — four colour theorem guarantees k=4 suffices for planar graphs
- **Register allocation** — compilers assign variables to CPU registers (graph = interference graph)
- **Scheduling** — exams, tasks, or frequencies with conflict constraints
- **Timetabling** — courses sharing students cannot be in the same slot

### Chromatic number

The minimum k for which a valid colouring exists is the **chromatic number** X(G). Finding X(G) is NP-hard; backtracking with pruning is the standard exact approach.

---

## Slide 12 -- Hamiltonian Path & Cycle

### Definitions

- **Hamiltonian path** — visits every vertex exactly once
- **Hamiltonian cycle** — a Hamiltonian path that returns to the starting vertex

### Backtracking solution

```python
def hamiltonian(graph, path, visited):
    if len(path) == len(graph):
        # check cycle: is there an edge back to start?
        if graph[path[-1]][path[0]]:
            return True                # Hamiltonian cycle found
        return False

    for v in range(len(graph)):
        if not visited[v] and graph[path[-1]][v]:
            visited[v] = True
            path.append(v)
            if hamiltonian(graph, path, visited):
                return True
            path.pop()                 # backtrack
            visited[v] = False

    return False
```

### Pruning strategies

- **Degree check** — if an unvisited vertex has no unvisited neighbours, prune immediately
- **Connectivity check** — if removing the current vertex disconnects the unvisited subgraph, prune
- **Warnsdorff's heuristic** — choose the vertex with fewest unvisited neighbours next (reduces branching)

> Hamiltonian cycle is NP-complete. No polynomial algorithm is known. Backtracking is the standard exact solver, but even with pruning it is impractical for graphs with hundreds of vertices.

---

## Slide 13 -- Constraint Satisfaction Problems

### Formal definition

A CSP is defined by:

- **Variables** — X1, X2, ..., Xn
- **Domains** — D1, D2, ..., Dn (possible values for each variable)
- **Constraints** — restrictions on which combinations of values are allowed

### Examples as CSPs

| Problem | Variables | Domains | Constraints |
|---------|----------|---------|-------------|
| N-Queens | Queen positions per row | Columns {1..N} | No shared column or diagonal |
| Sudoku | Each empty cell | {1..9} | Row, column, box uniqueness |
| Graph colouring | Vertex colours | {1..k} | Adjacent vertices differ |
| Map colouring | Region colours | {R, G, B, Y} | Neighbouring regions differ |
| Scheduling | Time slots per task | Available slots | No resource conflicts |

### Why the CSP framing matters

- Provides a **uniform representation** — one algorithm solves many problems
- Enables **general-purpose solvers** with domain-independent heuristics
- Separates **problem modelling** from **search strategy**

---

## Slide 14 -- CSP Solving Strategies

### Backtracking search for CSPs

Standard backtracking assigns one variable at a time and checks constraints after each assignment.

### Variable ordering heuristics

- **MRV (Minimum Remaining Values)** — choose the variable with the smallest domain; "fail-first" principle — detect dead ends sooner
- **Degree heuristic** — choose the variable involved in the most constraints on unassigned variables; break MRV ties

### Value ordering heuristics

- **Least Constraining Value (LCV)** — choose the value that rules out the fewest options for neighbouring variables; maximises remaining flexibility

### Inference / constraint propagation

- **Forward checking** — after assigning Xi, remove inconsistent values from unassigned neighbours' domains
- **Arc consistency (AC-3)** — enforce that for every value in Di, there exists a consistent value in each neighbouring Dj
- **MAC (Maintaining Arc Consistency)** — run AC-3 after every assignment during search

### Impact

| Strategy | Effect |
|----------|--------|
| Plain backtracking | Explores many dead-end branches |
| + MRV | Detects failures earlier |
| + Forward checking | Prunes domains proactively |
| + MAC | Near-optimal pruning; solves most CSPs efficiently |

---

## Slide 15 -- Pruning Strategies

### Why pruning matters

Without pruning, backtracking degenerates to brute force. Effective pruning is the difference between practical and intractable.

### Types of pruning

- **Feasibility pruning** — reject partial solutions that already violate a constraint
- **Bound pruning** — in optimisation, reject branches that cannot improve the best known solution
- **Symmetry breaking** — avoid exploring configurations that are rotations/reflections of already-explored solutions
- **Dominance pruning** — skip a partial solution if another partial solution is provably at least as good
- **Constraint propagation** — proactively reduce variable domains rather than waiting for conflicts

### Implementing effective pruning

- Check constraints **as early as possible** — ideally after each decision, not only at leaf nodes
- Maintain **incremental data structures** — e.g., boolean arrays for used columns/diagonals in N-Queens
- **Sort choices** — try the most constrained or most promising option first to find solutions (or contradictions) sooner
- **Memoisation** — cache results of subproblems when the state space has overlapping structure

> **Rule of thumb:** the cost of the pruning check must be less than the cost of exploring the pruned subtree. Cheap checks that eliminate large subtrees are the sweet spot.

---

## Slide 16 -- Optimisation Techniques

### Iterative deepening

Combine depth-first search with depth limits. Useful when the solution depth is unknown and memory is constrained.

### Bit manipulation

For problems with small state spaces (e.g., N-Queens with N <= 30), represent sets as bitmasks for O(1) constraint checks.

```python
# N-Queens with bitmasks
def solve(row, cols, diag1, diag2, n):
    if row == n:
        return 1
    count = 0
    available = ((1 << n) - 1) & ~(cols | diag1 | diag2)
    while available:
        bit = available & (-available)       # lowest set bit
        count += solve(row + 1,
                       cols | bit,
                       (diag1 | bit) << 1,
                       (diag2 | bit) >> 1, n)
        available ^= bit
    return count
```

### Randomised restarts

For hard CSPs, restart from a random initial state if the search stalls. Avoids getting trapped in unproductive subtrees.

### Parallelism

- Distribute independent subtrees across threads/processes
- Work stealing — idle workers take subtrees from busy workers
- Near-linear speedup for problems with many independent branches

---

## Slide 17 -- Branch and Bound

### Extending backtracking for optimisation

Branch and bound adds a **bounding function** that estimates the best possible solution achievable from a partial solution.

### How it works

1. **Branch** — split the problem into subproblems (like backtracking)
2. **Bound** — compute an optimistic estimate for each subproblem
3. **Prune** — if the bound is worse than the best known solution, discard the subproblem

### Backtracking vs branch and bound

| Aspect | Backtracking | Branch and Bound |
|--------|-------------|-----------------|
| Goal | Find feasible solutions | Find optimal solution |
| Pruning | Constraint violations | Bound vs best-so-far |
| Bounding function | Not required | Essential |
| Typical problems | CSPs, enumeration | TSP, knapsack, scheduling |

### Example: 0/1 Knapsack

- **Branch** — for each item, branch on include/exclude
- **Bound** — use fractional knapsack (greedy) as upper bound
- **Prune** — if upper bound <= current best profit, skip this subtree

> Branch and bound is the standard approach for integer linear programming (ILP) solvers like CPLEX and Gurobi. The quality of the bounding function determines solver performance.

---

## Slide 18 -- Backtracking vs Brute Force

### Brute force

Generate **all** possible candidates, then check each for validity.

- Always explores the full search space
- Simple to implement but impractical for large spaces
- Time: O(b^d) where b = branching factor, d = depth

### Backtracking

Generate candidates **incrementally**, pruning invalid branches early.

- Explores only a fraction of the search space
- Worst case is the same as brute force (no pruning effective)
- Best case is dramatically faster

### Empirical comparison

| Problem (typical) | Brute Force | Backtracking | Speedup |
|-------------------|------------|-------------|---------|
| 8-Queens | 16,777,216 | ~114 nodes | ~147,000x |
| Sudoku (easy) | 9^51 states | ~100 nodes | astronomical |
| Graph colouring (sparse) | k^n | ~O(k * n) | exponential |

### When backtracking does not help

- All solutions are valid (plain enumeration) — no pruning opportunities
- Constraints are only checkable at leaf nodes — no early termination
- Highly connected constraint graphs — pruning eliminates few branches

> Backtracking is not a different complexity class — it is a constant-factor (often enormous) improvement within the same exponential class. For polynomial-time solutions, you need a fundamentally different algorithm.

---

## Slide 19 -- Classic Problem Complexities

### Time complexities

| Problem | Worst Case | Typical with Pruning | Notes |
|---------|-----------|---------------------|-------|
| N-Queens | O(N!) | Much less in practice | One queen per row eliminates N^N |
| Sudoku | O(9^81) | Near-instant for most | Constraint propagation dominates |
| Subset Sum | O(2^n) | Depends on target/values | DP may be better for small targets |
| Permutations | O(n!) | O(n!) (all valid) | No pruning for unconstrained case |
| Combinations C(n,k) | O(C(n,k)) | O(C(n,k)) | Size pruning is the only gain |
| Graph Colouring | O(k^n) | Problem-dependent | Sparse graphs prune well |
| Hamiltonian Cycle | O(n!) | Problem-dependent | NP-complete; no known poly algorithm |
| TSP (B&B) | O(n!) | Often manageable for n<25 | Bounding function quality is key |

### Space complexity

- Recursive backtracking: O(d) stack space where d = maximum recursion depth
- Storing all solutions: O(S * d) where S = number of solutions

> **Practical takeaway:** backtracking complexity is problem-instance dependent. Analyse the constraint structure, not just the worst case, to predict real-world performance.

---

## Slide 20 -- Summary & Further Reading

### Key takeaways

- Backtracking is **brute force with pruning** — build candidates incrementally and abandon invalid branches early
- The **state space tree** is the mental model: every node is a decision, pruning removes subtrees
- The generic template — **choose, check, recurse, unchoose** — applies to N-Queens, Sudoku, graph colouring, and hundreds of other problems
- **Constraint satisfaction problems** unify many backtracking applications under one formal framework
- **Variable and value ordering** heuristics (MRV, LCV) and **constraint propagation** (AC-3, MAC) dramatically reduce search effort
- **Pruning quality** is the single biggest performance lever — cheap checks that eliminate large subtrees win
- **Branch and bound** extends backtracking to optimisation by adding bounding functions
- Backtracking does not change the complexity class — but the constant factor reduction is often the difference between seconds and centuries

### Recommended reading

| Source | Description |
|--------|------------|
| **Cormen et al.** | *Introduction to Algorithms* (CLRS) — Chapter on backtracking and branch-and-bound |
| **Skiena** | *The Algorithm Design Manual* — practical backtracking strategies and war stories |
| **Russell & Norvig** | *Artificial Intelligence: A Modern Approach* — CSP chapter with AC-3, MRV, MAC |
| **Knuth** | *The Art of Computer Programming, Vol. 4* — exhaustive enumeration and Dancing Links |
| **LeetCode** | [Backtracking tag](https://leetcode.com/tag/backtracking/) — curated practice problems |
| **Sedgewick** | *Algorithms* — permutations, combinations, and constraint-based search |
