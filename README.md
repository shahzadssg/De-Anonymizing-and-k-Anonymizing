
# De-Anonymization and $k$-Anonymization in Subject-Object Knowledge Networks

This repository provides Python implementations and an analysis framework for addressing the critical, yet inherently conflicting, challenges of de-anonymization and $k$-anonymization within subject-object knowledge networks. Our work leverages bipartite graph models to represent relationships (e.g., users accessing resources in cloud storage) and offers heuristic algorithms to either strengthen unique identification for security or enhance privacy through group indistinguishability.

## Introduction

In today's interconnected world, data often takes the form of complex networks. While this offers incredible opportunities for analysis, it also introduces significant privacy and security vulnerabilities. Our research focuses on two key aspects:

1.  **De-Anonymization**: Identifying individuals based on their unique access patterns. This is crucial for security applications like detecting collusion, tracing intellectual property leakage, or identifying compromised accounts. We aim to achieve a "Strong Unique Neighbourhood Network (UNN)" property, ensuring that each individual has a unique set of direct connections.

2.  **k-Anonymization**: Protecting individual privacy by ensuring that each individual's access patterns are indistinguishable from at least `k-1` other individuals. This not only safeguards privacy but can also improve system robustness by reducing reliance on single, identifiable points of failure.

This project demonstrates the fundamental trade-off between these two objectives: strengthening one inherently weakens the other. We provide practical implementations and a framework for understanding and manipulating graph structures to navigate these competing demands.

## Problem Formulation

Both de-anonymization and k-anonymization are formalized within a unified bipartite graph model:

*   **Subjects**: Individuals, users, or entities (e.g., employees in a company, students).
*   **Objects/Resources**: Items, questions, or data points accessed by subjects (e.g., files in cloud storage, survey questions).
*   **Edges**: Represent a subject's access or relation to an object.

The goal is to modify this bipartite graph by adding a minimal number of edges (and potentially synthetic nodes) to achieve either the Strong UNN property or k-neighbourhood anonymity.

## Algorithms Implemented

This repository includes Python implementations for the following algorithms:

1.  **KL-Divergence Calculation**: A utility function to compute Kullback-Leibler divergence between degree distributions, used as a metric for utility.
2.  **Bipartite Graph Creation**: Functions to construct NetworkX bipartite graphs from various data formats (e.g., student-question mappings, pandas DataFrames).
3.  **`check_strong_unn`**: Verifies if the Strong UNN property is satisfied for a given bipartite graph.
4.  **`naive_deanonymization`**: A baseline de-anonymization approach that adds a unique dummy node (question) for each student to ensure strong UNN.
5.  **`deanon_min_graph`**: Our proposed heuristic for de-anonymization, aiming to achieve Strong UNN by adding minimal edges, prioritizing existing items.
6.  **`optimal_strong_unn_bruteforce`**: An optimal (brute-force) solution for achieving Strong UNN, suitable only for very small graphs due to computational complexity. This is primarily for illustration and deriving optimal values for toy examples.
7.  **`kAnonHomogenize`**: Our proposed heuristic for k-anonymization, which enforces k-neighbourhood anonymity by homogenizing neighborhoods within groups, prioritizing existing resources.
8.  **`k_degree_anonymization`**: A basic implementation for k-degree anonymization, inspired by Liu & Terzi (2008), aiming to make the degree of each user indistinguishable from `k-1` others by adding edges.

## Getting Started

### Prerequisites

*   Python 3.x
*   `pandas`
*   `numpy`
*   `matplotlib`
*   `networkx`
*   `collections`
*   `itertools`
*   `seaborn`
*   `typing`
*   `random`
*   `warnings`
*   `openpyxl`

You can install these dependencies using pip:

```bash
pip install pandas numpy matplotlib networkx seaborn openpyxl
```

### Usage

The `main_analysis_suite()` function (defined in the code) orchestrates the execution of the de-anonymization and k-anonymization algorithms, generates performance tables, and saves illustrative graphs.

To run the analysis:

1.  Clone this repository:
    ```bash
    git clone https://github.com/your-username/your-repo-name.git
    cd your-repo-name
    ```
2.  Run the main script:
    ```bash
    python your_main_script_name.py
    ```

(Replace `your-username/your-repo-name.git` and `your_main_script_name.py` with your actual GitHub details and the name of the Python file containing the `main_analysis_suite` function, likely `main.py` or similar).

The script will output performance metrics and generate CSV files (`table1_resource_comparison.csv`, `table2_comparative_performance.csv`, `table3_heuristic_performance.csv`) and PNG images (`1.png` for de-anonymization, `2.png` for k-anonymization) in the same directory.

## Analysis and Results

The `main_analysis_suite` demonstrates the trade-offs involved:

*   **Table 1: Resource Comparison (Student Example)**: Compares the naive, heuristic (DeAnonMinGraph), and optimal (brute-force) de-anonymization approaches for a small, illustrative dataset. It shows the number of new nodes and edges added to achieve Strong UNN.
*   **Figure 1: Modified Graph (Strong UNN achieved)**: Visualizes the graph after applying de-anonymization, illustrating the structural changes.
*   **Table 2: Comparative Performance for K-Anon**: Evaluates the performance of our `kAnonHomogenize` method against the basic k-degree anonymization and conceptually discusses other k-anonymization methods. Metrics include edges added, nodes added, runtime, and utility (measured by KL-Divergence from original degree distribution).
*   **Table 3: Heuristic Performance on Synthetic Graphs**: Provides an averaged performance evaluation of `kAnonHomogenize` and k-degree anonymization on larger, synthetically generated datasets, showcasing their scalability and effectiveness.
*   **Figure 2: Modified Graph (k=2 Anonymity)**: Visualizes the graph after applying k-anonymization, highlighting how neighbourhoods are homogenized.

Our findings consistently highlight that achieving robust de-anonymization often requires significant structural modification (adding edges/nodes), similarly, achieving k-anonymity also necessitates graph alterations. The choice between these two goals involves a careful consideration of security requirements, privacy needs, and acceptable levels of data utility degradation.

## Contributions

We welcome contributions to this project! If you have suggestions for improvements, new algorithms, or wish to report issues, please feel free to open an issue or submit a pull request.

