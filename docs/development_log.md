# Advanced Shortest Paths — Development Log

**Course:** Algorithms on Graphs, Optional Module 6  
**Institution:** University of California San Diego via Coursera  
**Language:** Python 3  
**Status:** Completed

## 1. Project Overview

This project explores how shortest-path queries can be accelerated on large directed graphs. It includes five related tasks:

1. **Friend Suggestion** — Bidirectional Dijkstra
2. **Distance with Coordinates** — A* search
3. **Distance with Preprocessing (Small)** — Contraction Hierarchies
4. **Distance with Preprocessing (Large)** — Optimized Contraction Hierarchies
5. **Travelling Salesman Problem** — CH shortest-path queries combined with bitmask dynamic programming

The main challenge was not only implementing the algorithms, but also maintaining correctness while improving query performance.

## 2. Correctness Baseline

Before optimizing, I used ordinary Dijkstra as a reference implementation on small graphs. I compared the optimized algorithms against this baseline using hand-built and randomly generated cases.

Important edge cases included:

- Source equals target
- No path exists
- Parallel directed edges with different weights
- Paths with several intermediate vertices
- Repeated queries on the same graph
- Asymmetric reachability in a directed graph

## 3. Bidirectional Dijkstra

### Goal

Instead of searching only from the source, Bidirectional Dijkstra performs two searches:

- Forward from the source
- Backward from the target using the reverse graph

The two searches can often explore a much smaller portion of the graph than ordinary Dijkstra.

### Difficulty: Returning at the First Meeting Vertex

My first concern was whether the algorithm could stop as soon as the two searches visited the same vertex.

That stopping rule is not always correct. The first meeting vertex is not necessarily part of the globally shortest path.

### Resolution

I maintained the best complete path discovered so far by considering:

`forward_distance[v] + backward_distance[v]`

The search stops only when the remaining priority-queue entries cannot improve the current best answer.

### Difficulty: Resetting State Between Queries

The assignment contains many shortest-path queries on the same graph. Resetting every entry in the distance and visited arrays after every query would add unnecessary linear-time work.

### Resolution

I maintained a `workset` containing only vertices whose state changed during the current query. Before the next query, only those vertices are reset.

This reduced repeated initialization work while keeping the queries independent.

## 4. A* Search with Coordinates

### Goal

A* uses node coordinates to guide the search toward the target.

The Euclidean distance between two coordinates provides a lower-bound heuristic because the road-network edge length is guaranteed to be at least the straight-line distance between its endpoints.

### Difficulty: Separating Distance from Priority

The heuristic-adjusted priority used by A* is not the same as the actual distance from the source.

Mixing these two values can cause incorrect relaxations or an unsafe stopping condition.

### Resolution

I kept two concepts separate:

- The actual shortest distance discovered so far
- The priority used to decide which vertex to explore next

The priority queue uses the heuristic, while edge relaxation updates the real path distance.

### Difficulty: Bidirectional Potentials

When combining coordinate heuristics with bidirectional search, the forward and backward potentials must remain balanced.

An invalid potential can create unsafe reduced edge costs and cause the search to return an incorrect distance.

### Resolution

I used balanced forward and backward potentials derived from the coordinate heuristic and compared the answers against ordinary Dijkstra on small graphs.

## 5. Contraction Hierarchies Preprocessing

### Goal

Contraction Hierarchies move computational work into a preprocessing phase so that later shortest-path queries can be answered much faster.

Vertices are contracted in a selected order. When a vertex is removed, shortcuts may need to be added to preserve shortest-path distances.

### Difficulty: Determining Whether a Shortcut Is Necessary

Suppose the algorithm is contracting vertex `v`, with an incoming edge from `u` and an outgoing edge to `w`.

The path through the contracted vertex has cost:

`cost(u, v) + cost(v, w)`

A shortcut from `u` to `w` is required only if no alternative path of equal or lower cost exists without using `v`.

### Resolution

I implemented a bounded witness search that temporarily avoids the vertex being contracted.

- If another path from `u` to `w` is no longer than the candidate shortcut, no shortcut is added.
- Otherwise, the shortcut is added to preserve the shortest-path distance.

### Difficulty: Choosing the Contraction Order

Contracting a poorly chosen vertex too early can create many shortcuts, making the graph larger and slowing preprocessing and queries.

### Resolution

I computed a node-importance score using features such as:

- Edge difference
- Number of contracted neighbors
- Shortcut coverage
- Vertex level

Because contractions can change neighboring importance scores, I recomputed importance lazily before finalizing each contraction.

### Difficulty: Maintaining Both Graph Directions

A shortcut added only to the forward graph would make the reverse graph inconsistent.

### Resolution

Whenever an edge or shortcut was added, I updated both the forward and reverse adjacency structures. If an equivalent edge already existed, I retained the smaller cost.

## 6. Contraction Hierarchies Queries

### Goal

After preprocessing, a shortest-path query should traverse only edges that move upward in the contraction hierarchy.

### Difficulty: Rank Direction

If the rank comparison is reversed, the query can skip valid paths or behave like an unrestricted shortest-path search.

### Resolution

Each contracted vertex receives a rank. Query relaxation is allowed only when moving from a lower-ranked vertex to a higher-ranked vertex.

- Forward search uses the original adjacency list.
- Backward search uses the reverse adjacency list.

### Difficulty: Stale Priority-Queue Entries

Python’s `heapq` does not directly support decrease-key operations. As a result, the same vertex can appear in the heap more than once.

### Resolution

Before processing a queue entry, I checked whether:

- Its stored distance still matched the current best distance
- The vertex had already been processed from that search direction

Outdated entries were discarded.

### Difficulty: Incorrect State Dimensions

Forward and backward searches require separate visited and distance states. Using the wrong dimensions can mix the two search directions and produce incorrect results.

### Resolution

I maintained separate state for both search directions and verified that each query reset only the vertices stored in the `workset`.

### Difficulty: Missing Imports

One implementation initially failed because `heapq` had not been imported.

### Resolution

I added the required import and reran the tester before continuing with further optimization.

### Difficulty: Query Termination

Stopping too early can miss a shorter path, while continuing until both queues are empty wastes the benefit of preprocessing.

### Resolution

I maintained the best complete-path estimate found so far. A search direction stops contributing when its smallest queued distance cannot improve that estimate.

## 7. Scaling the Implementation

The larger road-network task required the same correctness guarantees under tighter time and memory limits.

The implementation uses:

- Lazy importance recomputation
- Bounded witness searches
- Shortcut deduplication
- Forward and reverse adjacency structures
- Rank-restricted query traversal
- A workset for efficient state reset
- Compact storage for dynamic-programming states

The main tradeoff is between preprocessing quality and preprocessing time. A more thorough witness search may prevent more shortcuts, but it also makes contraction slower.

## 8. Travelling Salesman Problem

### Goal

The task is to start at a depot, visit every requested store, and return to the depot with minimum total distance.

### Step 1: Construct a Reduced Distance Graph

For each query, I used Contraction Hierarchies to calculate directed shortest-path distances among the depot and selected stores.

This produces a small distance matrix containing only the relevant locations.

### Step 2: Apply Bitmask Dynamic Programming

A dynamic-programming state records:

- Which stores have already been visited
- The last store reached
- The minimum distance needed to reach that state

The state can be represented as:

`dp[mask][last] = minimum distance ending at last after visiting the stores in mask`

After filling the table, I added the distance from the final store back to the depot and selected the minimum feasible total.

### Difficulty: Exponential State Size

The original road graph may contain many vertices, but each TSP query contains at most 20 selected locations.

### Resolution

Contraction Hierarchies handle shortest-path calculations on the large graph. Exponential dynamic programming is applied only to the much smaller selected set.

### Difficulty: Unreachable Location Pairs

Because the graph is directed, one selected location may not be able to reach another.

### Resolution

Unreachable pairwise distances remain infinite. The dynamic program ignores those transitions and returns `-1` if no complete round trip exists.

## 9. Testing Strategy

I used three levels of testing:

1. **Small hand-built graphs** to verify exact answers and directed edge cases
2. **Differential testing** by comparing optimized algorithms with ordinary Dijkstra
3. **Course tester** to check correctness under the assignment’s time and memory requirements

The final implementation passed the project tester after resolving state-management, import, rank-order, and upward-edge traversal issues.

## 10. Results

Measured benchmarks will be added after running the final implementation on reproducible datasets. I do not report course-advertised speedups as personal results.

| Algorithm | Graph Size | Queries | Preprocessing Time | Query Time | Baseline |
|---|---:|---:|---:|---:|---|
| Bidirectional Dijkstra | TBD | TBD | N/A | TBD | Dijkstra |
| A* | TBD | TBD | N/A | TBD | Dijkstra |
| Contraction Hierarchies | TBD | TBD | TBD | TBD | Dijkstra |
| CH + Bitmask DP | TBD | TBD | TBD | TBD | Brute force when feasible |

Future benchmarks will also record the computer, operating system, and Python version so the results are reproducible.

## 11. What I Learned

- A speed improvement is useful only if its stopping condition preserves correctness.
- Preprocessing can transfer work from repeated queries into a one-time graph transformation.
- Directed graphs require consistent forward and reverse representations.
- Priority queues may contain stale entries and require explicit validation.
- Differential testing is useful because optimized graph algorithms often fail only on unusual graph structures.
- A large optimization problem can sometimes be divided into a large-graph shortest-path stage and a small exponential dynamic-programming stage.

## 12. Next Steps

- Add reproducible benchmark scripts and measured results
- Add an original diagram explaining contractions, witness paths, and shortcuts
- Build an independent route-planning demo using an openly licensed road-network dataset
- Add unit tests for state reset, unreachable paths, parallel edges, and rank restrictions

## Academic Integrity

This public repository documents the algorithms, development process, debugging decisions, and testing strategy.

Course starter files, hidden tests, assignment PDFs, and full solution code are not included. The complete implementation remains in a private repository unless publication is explicitly permitted by the course policy.