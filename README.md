# Advanced Shortest Paths

Python implementations and technical notes from the optional Advanced Shortest Paths module in UC San Diego's *Algorithms on Graphs* course.

## Algorithms Studied

- Bidirectional Dijkstra
- A* search with coordinate heuristics
- Contraction Hierarchies preprocessing and queries
- Witness searches, shortcuts, contraction ranks, and node-importance heuristics
- Travelling Salesman Problem using CH distances and bitmask dynamic programming

## What This Project Demonstrates

- Implementation of nontrivial graph algorithms
- Correctness testing against a Dijkstra baseline
- Preprocessing/query-time tradeoffs
- Efficient state reset across repeated queries
- Debugging of stopping conditions, reverse graphs, stale heap entries, shortcuts, and hierarchy ranks

See the [development log](docs/development_log.md) for the design process, problems encountered, resolutions, and testing plan.

## Results

Measured benchmarks will be added after running the final implementation on reproducible datasets. I do not report course-advertised speedups as personal results.

## Academic Integrity

This public documentation describes the algorithms and my development process. Course starter files, hidden tests, and full assignment solutions are not included. The implementation remains private unless publication is explicitly permitted by the course policy.