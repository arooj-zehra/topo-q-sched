# Topo-Q-Sched

Companion code for **"Topo-Q-Sched: A Topology-Grounded Hybrid Quantum-Classical Graph Neural Network for NVLink-Aware GPU Scheduling."**

A hybrid quantum-classical framework for NVLink-aware GPU scheduling. A topology-structured Quantum Graph Neural Network (QGNN) predicts pairwise NVLink interference between co-located jobs, and a Quantum Approximate Optimization Algorithm (QAOA) uses that prediction to solve a bridge-bipartition scheduling decision — validated against classical baselines and on real IBM Quantum hardware.

## What's in this repo

A single self-contained notebook, `tpq.ipynb`, that reproduces every number, table, and figure reported in the manuscript, end to end:

1. **Interference/slowdown ground truth** — a saturating contention model whose link-bandwidth constants are taken from published NVLink measurements (MAPA, SC '21), with any non-literature-derived constant disclosed explicitly as a modeling assumption.
2. **Topology-structured QGNN** — qubits represent physical GPUs, entangling gates are restricted to the real NVLink adjacency graph, and predictions are read out per physical edge.
3. **Classical baselines** — MLP, GCN (Kipf & Welling), and GAT (Veličković et al.), trained and evaluated on identical data splits for a fair comparison.
4. **Topology ablation and permutation-equivariance test** — direct empirical evidence that the QGNN's advantage is tied to the real topology, not just the presence of a quantum circuit.
5. **QAOA scheduling** — evaluated on a single, explicitly stated 8-qubit decision window, on both a noiseless simulator and real IBM Quantum hardware (`ibm_fez`), including a transpilation diagnostic explaining the hardware degradation.

## Results summary

| Model | MAE ↓ | RMSE ↓ | R² ↑ | Spearman ρ ↑ |
|---|---|---|---|---|
| MLP | 0.0540 | 0.0718 | 0.598 | 0.824 |
| GCN | 0.0681 | 0.0879 | 0.398 | 0.661 |
| GAT | 0.0762 | 0.0955 | 0.290 | 0.635 |
| **QGNN (topology-structured)** | **0.0474** | **0.0589** | **0.730** | **0.843** |

- **Topology ablation:** randomizing the entangling structure (same edge count) raises MAE from 0.047 to 0.063.
- **Permutation equivariance:** mean prediction deviation of 0.009 under a true graph automorphism.
- **QAOA (simulator):** approximation ratio improves from 0.512 (p=1) to 0.702 (p=3).
- **QAOA (real hardware, `ibm_fez`):** approximation ratio 0.269 (p=1) and 0.252 (p=3) — degradation traced via a transpilation diagnostic to connectivity mismatch between the circuit and the processor's heavy-hexagonal qubit layout.

## Repository layout

```
tpq.ipynb                 # the companion notebook (run top-to-bottom)
outputs/
├── figures/               fig1_topology.png, fig2a_interference_error.png,
│                           fig2b_ordering_fidelity.png, fig3_topology_ablation.png,
│                           fig4_qaoa_depth.png   (all 400 DPI)
├── data/                   scheduling_instances.pkl, model_comparison.csv,
│                           topology_ablation.csv, qaoa_results.json,
│                           permutation_equivariance_result.json,
│                           ibm_hardware_result_p1.json, ibm_hardware_result_p3.json
└── models/                 qgnn_model.pt
```

## How to run

1. Open `tpq.ipynb` in Google Colab (or any Jupyter environment with pip access).
2. Run top-to-bottom. **Sections 1–9** use only free, local CPU compute and complete in well under 15 minutes.
3. **Section 10** (real IBM Quantum hardware validation) is gated behind a manual flag (`RUN_ON_HARDWARE`, `RUN_P1_HARDWARE_COMPARISON`) that defaults to `False`, since the free IBM Quantum Open Plan allowance is only 10 minutes per 28-day period. The outputs already present in that section are preserved from the real hardware run reported in the manuscript. To re-run it yourself, read that section's markdown, set the flag(s) to `True`, and provide **your own** IBM Quantum API token (`QiskitRuntimeService.save_account(...)` is recommended over pasting a token directly into the notebook).

### Requirements

```
pennylane==0.45.1
torch==2.11.0
qiskit==2.5.2
qiskit-ibm-runtime==0.49.0
qiskit-aer
networkx
matplotlib
pandas
scipy
```

## Honesty / scope notes

- The 8-GPU ring-with-two-bridges topology is an **illustrative digital twin**, not a claim about the real physical topology of any specific NVIDIA product (real DGX-1 systems use a hybrid cube-mesh topology; real DGX A100 systems use a 6-NVSwitch full mesh).
- The interference model's bandwidth constants are literature-sourced; the contention-onset and saturation constants are disclosed modeling assumptions, not calibrated values (see Section 2 of the notebook).
- QAOA is validated at a single 8-qubit decision-window scale; claims about larger job queues (repeated application of this window) are not benchmarked end-to-end here.
- No live Kubernetes/cluster-scheduler deployment was built; the interference-prediction and QAOA-scheduling components are evaluated in isolation.

See the manuscript's Limitations section (and Section 12 of the notebook) for the full list.

## Citation

If you use this code, please cite the manuscript:

```bibtex
@article{kamran2026topoqsched,
  title   = {Topo-Q-Sched: A Topology-Grounded Hybrid Quantum-Classical Graph Neural Network for NVLink-Aware GPU Scheduling},
  author  = {Kamran, Muhammad and Malik, Tahir and Zahra, Arooj},
  journal = {Quantum Information Processing},
  year    = {2026},
  note    = {Under review}
}
```

## License

_Add your chosen license here (e.g., MIT) before publishing._
