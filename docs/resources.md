# Resources — ECEG-4192

---

## Core textbooks

| Book | Chapters used | Why |
|---|---|---|
| **M. A. Weiss**, *Data Structures and Algorithm Analysis in Java* (or C++ ed.) | All | The best single reference — rigorous complexity analysis with clean implementations |
| **R. Lafore**, *Data Structures & Algorithms in Java* | All | Gentler, more visual, good for UG |
| **Y. Langsam, M. Augenstein, A. Tanenbaum**, *Data Structures in C and C++* | All | Classic, comprehensive |
| **A. Drozdek**, *Data Structures and Algorithms in Java* | All | Good coverage of balanced trees and graphs |
| **S. Lipschutz**, *Theory and Problems of Data Structures*, Schaum's | All | Worked problems for exam prep |

### Supplemental

| Book | Why |
|---|---|
| **T. H. Cormen, C. E. Leiserson, R. L. Rivest, C. Stein**, *Introduction to Algorithms* (CLRS), 4th ed. | The definitive reference — deeper than this course needs, but the source of truth for anything ambiguous |
| **R. Sedgewick**, *Algorithms*, 4th ed. | Great visual explanations for sorting and trees |
| **N. Wirth**, *Algorithms + Data Structures = Programs* | Classic for intuition |
| **S. Skiena**, *The Algorithm Design Manual* | Practical problem-solving after this course |

---

## Free online materials 🆓

- **VisuAlgo** — [visualgo.net](https://visualgo.net/) — animated visualisations of every data structure and algorithm in this course. **Start here** for anything visual.
- **MIT 6.006 — Introduction to Algorithms** (OCW) — [ocw.mit.edu/courses/6-006-introduction-to-algorithms-spring-2020/](https://ocw.mit.edu/courses/6-006-introduction-to-algorithms-spring-2020/) — lecture videos and problem sets. The best match for this course.
- **Princeton Algorithms, Part I & II** (Coursera) — [coursera.org/learn/algorithms-part1](https://www.coursera.org/learn/algorithms-part1) — Sedgewick's course, with code and visualisations.
- **Open Data Structures** by Pat Morin — [opendatastructures.org](https://opendatastructures.org/) — free online textbook, rigorous, with Java/Python/C++ implementations.
- **Python Tutor** — [pythontutor.com](https://pythontutor.com/) — step through code execution to understand recursion and pointer manipulation.
- **CS50 — Introduction to Computer Science** (Harvard) — [cs50.harvard.edu](https://cs50.harvard.edu/x/) — for review of C/Python foundations.
- **cp-algorithms.com** — [cp-algorithms.com](https://cp-algorithms.com/) — dense reference for graph and tree algorithms.

---

## Software

| Language | Tool | Use |
|---|---|---|
| **Java** | JDK 17+, IntelliJ IDEA or VS Code | All labs (primary language) |
| **Python** | Python 3.10+, NumPy for benchmarks | All labs (secondary language) |
| **Visualisation** | [VisuAlgo](https://visualgo.net/), [Python Tutor](https://pythontutor.com/) | Understand what the code is doing |
| **Debugging** | JDB / pdb | Tracing pointer and recursion problems |
| **Benchmarking** | JMH (Java), `timeit` (Python) | Sorting / structure comparisons |

**Jupyter** is optional but useful for tracking lab experiments.

---

## Datasets and benchmarks

- **Random integers** (generated) — for sorting benchmarks
- **Real text** (Project Gutenberg, `/usr/share/dict/words`) — for hash table and BST experiments
- **Small graphs** (US road subset, social network samples) — for shortest-path labs
- **DIMACS graph challenges** — [dimacs.rutgers.edu/pub/challenge/](http://dimacs.rutgers.edu/pub/challenge/) — for graph algorithm verification

---

## Recommended first-week reading

- **Weiss, Chapters 1–2** — complexity analysis and Big-O definitions
- **Lafore, Chapter 1–2** — gentle introduction
- **VisuAlgo** — click through the "Sorting" and "List" sections to see the shapes