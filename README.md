# Quantum Circuit Simulation of Black Hole Evaporation

**Author:** Karamala Aaqhil Ahamed  
**Research Focus:** Quantum information theory, open quantum systems, entanglement dynamics

---

## Overview

This repository documents an independent research project exploring **black hole evaporation as a quantum information problem**, implemented through quantum circuit simulation using Qiskit.

The central question: *Can we reproduce Page-curve behaviour — and eventually the island-corrected entropy — using explicit unitary quantum circuits?* Starting from a minimal two-qubit scrambler, the project evolves across 14+ versions toward a circuit model that incorporates the island formula for entropy correction and non-local echo coupling between the black hole interior and early radiation.

All simulations run on classical hardware via Qiskit's statevector simulator, treating the full (B+R) system as a pure state and computing subsystem entropies via partial trace.

This repository accompanies the technical report: **"Quantum Simulation of the Black Hole Information Paradox"** (2025), which documents simulation results, entropy plots, and physical interpretation for each version in detail.

---

## Physics Background

### The Problem
When a black hole evaporates via Hawking radiation, a naive quantum field theory calculation predicts the radiation entropy grows monotonically — implying information is destroyed. This contradicts unitarity (the **black hole information paradox**).

### The Page Curve
Don Page (1993) showed that for a unitary evaporation process, the entanglement entropy of radiation must:
- Increase like `S ~ N_R` (number of emitted qubits) in the early phase
- **Turn over and decrease** after the **Page time** (`N_R = N_total / 2`)
- Return to zero when the black hole is fully evaporated

This turnover is the signature of information being returned to the radiation.

### The Island Formula
Recent work (Almheiri et al., 2019; Penington, 2019) resolves the paradox via the **island formula**:

```
S(R) = min over islands I [ S_vN(R ∪ I) ]
```

An "island" `I` is a region inside the black hole whose entropy, when added to the radiation, gives a *smaller* total — reflecting non-local quantum correlations between interior and exterior. This project implements a circuit-level version of this prescription.

### Key Quantities Computed
- **Von Neumann entropy** `S_vN = -Tr(ρ log₂ ρ)` of the radiation subsystem
- **Partial trace** over black hole qubits to obtain reduced density matrix `ρ_R`
- **Island entropy** `S_vN(R ∪ I)` by tracing out `B \ I`
- **Island-corrected entropy** `S = min(S_vN(R), S_vN(R ∪ I))`

---

## Repository Structure

```
black-hole-quantum-simulation/
│
├── README.md                          ← This file
│
├── page_curve_project/                ← Main simulation series
│   ├── v1_minimal_scrambler.ipynb     ← Baseline: 2-qubit CNOT scrambler
│   ├── v4_5_multi_layer.ipynb         ← 3-layer scrambler, RX/RY/RZ rotations
│   ├── v6_0_deeper_scrambling.ipynb   ← 5-layer scrambler + long-range CX
│   ├── v6_1_8q.ipynb                  ← Scaling test: 8 qubits, 15 layers
│   ├── v6_3_12q.ipynb                 ← Scaling test: 12 qubits
│   ├── v6_4_14q.ipynb                 ← Scaling test: 14 qubits
│   ├── v8_diverse_gates.ipynb         ← Diverse gate set (CX+CZ+CY+SWAP), circuit visualization
│   ├── v11_island_mechanism.ipynb     ← Island formula first implementation
│   └── v14_echo_coupling.ipynb        ← Echo coupling + full island-corrected plotting
│
└── notes/
    └── version_history.md             ← Detailed changelog (this document's appendix)
```

---

## Version History

This project evolved through iterative hypothesis-testing. Each version introduced a specific physical or computational change and was motivated by observing the previous version's behavior.

---

## Central Physics Narrative: Evolution of the B–R Entanglement Mechanism

This is the most important conceptual thread running through the project. The question isn't just *how much* entanglement is generated — it's *what kind*, and *when*, and *whether it can be reversed*.

### Phase 1 — Hawking Pair Entanglement (v1)
In v1, the emitted qubit is entangled with a neighboring interior qubit at the moment of emission. This directly models the standard Hawking pair-creation picture: the outgoing particle is entangled with its interior partner at the horizon. Once emitted, however, the radiation qubit is decoupled — it no longer interacts with the black hole interior. This is the semiclassical approximation, and it correctly produces *growing* entropy in the radiation, but with no mechanism for the entropy to ever decrease. Information is lost.

### Phase 2 — Interior-Only Scrambling (v6.x)
The explicit emission-time B–R coupling is removed. Scrambling now occurs *only within the remaining black hole qubits* before each emission. Radiation passively accumulates entropy without any post-emission coupling to the interior. This isolates the effect of internal scrambling on the Page curve shape, and shows that scrambling alone (without information backflow) cannot produce the turnover.

### Phase 3 — Post-Emission Feedback and Information Backflow (v11, v14)
The key physical transition. After Page time, controlled operations couple the outgoing qubit to *old radiation* before it leaves. In v14, probabilistic echo pulses create bidirectional feedback between the black hole interior and previously emitted radiation. Unlike the original Hawking entanglement (which *generates* entropy by building correlations across the horizon), this late-time coupling *enables purification* — it allows information encoded in the black hole interior to be recovered from the radiation, consistent with the island/QES prescription.

**In summary:** the project moves from one-time entanglement at emission → passive scrambling → dynamic post-emission feedback that restructures the entanglement and restores unitarity at late times. Each phase corresponds to a distinct physical assumption about what the black hole can "do" with information.

---

### v1 — Minimal Two-Qubit Scrambler
**File:** `v1_minimal_scrambler.ipynb`  
**System size:** 8 qubits

**What changed:** Baseline implementation. The scrambler is a single `CNOT` between the outgoing qubit and its nearest neighbor in B, followed by fixed `Rz(π/4)` and `Ry(π/2)` rotations.

**Physical motivation:** Model Hawking emission as a minimal entangling interaction before each qubit leaves the black hole.

**Limitation identified:** Fixed rotation angles and only nearest-neighbor entanglement produces weak, structured scrambling — not the chaotic, all-to-all mixing expected from a real black hole. The Page curve shape is approximate but doesn't saturate the theoretical bound.

---

### v4.5 — Multi-Layer Scrambler with Random Rotations
**File:** `v4_5_multi_layer.ipynb`  
**System size:** 8 qubits

**What changed:**
- Scrambler expanded to **3 layers** of repeated entangling + rotation
- Fixed rotation angles replaced with **random `Rx`, `Ry`, `Rz`** angles (drawn from `[0, 2π]`)
- Added bidirectional CNOTs (`cx(i, i+1)` and `cx(i+1, i)`)

**Physical motivation:** Black holes are fast scramblers — quantum information injected into a black hole becomes maximally mixed across all degrees of freedom in ~`log(N)` time. Random unitaries better approximate this Haar-random mixing.

**Observation:** Entropy values move closer to the `min(N_R, N_B)` theoretical bound. Randomness introduces run-to-run variation, motivating ensemble averaging in later versions.

---

### v6.0 — Deeper Scrambling + Long-Range Entanglement
**File:** `v6_0_deeper_scrambling.ipynb`  
**System size:** 8 qubits

**What changed:**
- Layers increased to **5**
- Added `CZ` gates alongside `CX` for phase entanglement
- Added a **long-range `CX`** between the first and last qubit in B (`cx(b[0], b[-1])`)

**Physical motivation:** Real black hole scrambling is not nearest-neighbor. Long-range interactions encode the fact that information is distributed non-locally across the horizon. The `CZ` gate adds phase-type entanglement in addition to bit-flip entanglement, enriching the Hilbert space mixing.

**Observation:** Entropy curve tracks the theoretical bound more closely; the long-range gate noticeably affects late-time entropy values.

---

### v6.1 / v6.3 / v6.4 — System Size Scaling
**Files:** `v6_1_8q.ipynb`, `v6_3_12q.ipynb`, `v6_4_14q.ipynb`  
**System sizes:** 8, 12, 14 qubits  
**Layers:** increased to 15

**What changed:** Same architecture as v6.0 but scaled to larger system sizes, with layers bumped to 15. Added `CY` and additional `CZ` long-range gates (`cy(b[0], b[-1])`, `cz(b[0], b[-1])`).

**Physical motivation:** Finite-size effects are significant in small systems. Testing whether Page-curve behavior is robust across system sizes, and whether the turnover sharpens with more qubits (as expected theoretically).

**Key observation:** Larger systems produce smoother, more symmetric Page curves. The 14-qubit version (v6.4) is the cleanest reproduction of the theoretical `min(N_R, N_B)` shape with this scrambler architecture.

---

### v8 — Diverse Gate Set + Circuit Visualization
**File:** `v8_diverse_gates.ipynb`  
**System size:** 12 qubits

**What changed:**
- Gate set diversified: each layer now applies `CX + CZ + CY + SWAP` between adjacent qubits, varying target/control orientation
- Long-range gates extended to include `cy(b[1], b[-2])` and `cz(b[0], b[-2])` for richer non-local structure
- Added `create_circuit_at_step()` function to **visualize the circuit at any evaporation step**
- Added `pylatexenc` install for LaTeX-quality circuit diagrams

**Physical motivation:** The diversity of gate types better approximates a Haar-random unitary (the theoretical model for a maximally scrambling black hole). SWAP gates model physical qubit hopping/exchange. CY gates add imaginary-component entanglement.

**New capability:** Circuit visualization allows direct inspection of what the scrambler looks like at depth — important for understanding whether the circuit is genuinely scrambling or just permuting.

---

### v11 — Island Mechanism (First Implementation)
**File:** `v11_island_mechanism.ipynb`  
**System size:** 12 qubits

**What changed:** The most physically significant version. Added the **island mechanism** — after Page time, the outgoing qubit is explicitly entangled with *old* radiation qubits before emission:
```python
# After Page time:
current_circuit.cx(qubit_to_emit, r_qubits[0])   # couple to oldest radiation
current_circuit.cz(qubit_to_emit, r_qubits[1])   # couple to second oldest
```

**Physical motivation:** The island formula requires non-local correlations between the black hole interior and early radiation. After Page time, Hawking quanta must carry information that is *entangled with previously emitted radiation* — not just with the current black hole state. This circuit-level coupling is a concrete implementation of that non-local entanglement switch.

**Observation:** The entropy curve shows modified late-time behavior. The entropy begins to decrease after Page time more consistently than in versions without the island coupling, qualitatively tracking the Page curve turnover.

**Limitation identified:** The coupling is deterministic and hardcoded to the two oldest radiation qubits. A more physical implementation should be probabilistic and possibly involve more radiation qubits, motivating v14.

---

### v14 — Echo Coupling + Full Island-Corrected Entropy Computation
**File:** `v14_echo_coupling.ipynb`  
**System size:** 12 qubits

**What changed:** The most complete version. Two major additions:

**1. Echo coupling** — a new `apply_echo_coupling()` function applies a probabilistic "feedback pulse" between old radiation and the black hole interior *after* each emission step:
```python
# Echo pulse: bidirectional feedback
qc.cx(r_target, b_target)  # radiation → black hole
qc.cz(b_target, r_target)  # black hole → radiation (phase)
qc.cy(r_target, b_target)  # radiation → black hole (y-type)
# + random single-qubit rotations for chaotic feedback
```
The `p_echo` parameter (default 0.6) controls the probability of applying the echo on any given step. `echo_after_page=True` restricts echo coupling to post-Page-time steps only.

**Physical motivation:** Echo coupling models the idea that the black hole interior is not causally isolated from radiation it has already emitted — consistent with the non-local correlations implied by the island formula. The probabilistic nature introduces a realistic stochastic element to the information backflow.

**2. Full island entropy computation** — entropy is now computed *two ways* at each step:
- `S_vN(R)`: standard radiation entropy (trace out all of B)
- `S_vN(R ∪ I)`: island-corrected entropy (trace out `B \ I`, where `I = b_qubits[-1]`)
- Final reported entropy: `S = min(S_vN(R), S_vN(R ∪ I))` — the island prescription

**Output:** The plot now shows all three curves simultaneously: island-corrected `S`, raw `S_vN(R)`, and `S_vN(R ∪ I)`, against the theoretical `min(N_R, N_B)` bound. This directly mirrors the structure of figures in island formula papers.

---

## Key Results

| Version | System Size | Scrambler Depth | Island? | Echo? |
|---------|------------|-----------------|---------|-------|
| v1      | 8 qubits   | 1 layer, fixed  | No      | No    |
| v4.5    | 8 qubits   | 3 layers, random| No      | No    |
| v6.x    | 8–14 qubits| 15 layers       | No      | No    |
| v8      | 12 qubits  | 15 layers + diverse gates | No | No  |
| v11     | 12 qubits  | 15 layers       | Yes (deterministic) | No |
| v14     | 12 qubits  | 15 layers       | Yes (island formula) | Yes (probabilistic) |

---

## How to Run

All notebooks are designed to run on **Google Colab** (free tier sufficient for ≤12 qubits).

```python
# Install dependencies (already included in each notebook)
!pip install qiskit qiskit-aer

# Run the main simulation (v14 example)
initial_circuit = initialize_system(N_QUBITS)
results = simulate_evaporation(initial_circuit, N_QUBITS, echo_after_page=True, p_echo=0.6)
plot_page_curve(results, N_QUBITS)
```

**Note on system size:** Statevector simulation scales as `2^N`. 12 qubits runs in ~1-2 minutes on Colab. 14 qubits is near the practical limit (~5-10 min). 16+ qubits requires significant memory.

---

## Dependencies

```
qiskit >= 2.2
qiskit-aer >= 0.17
numpy
matplotlib
pylatexenc  # for circuit diagram rendering (v8+)
```

---

## Related Reading

- Page, D.N. (1993). *Information in black hole radiation.* PRL 71, 3743.
- Hayden, P. & Preskill, J. (2007). *Black holes as mirrors.* JHEP 09, 120.
- Almheiri, A. et al. (2019). *The entropy of Hawking radiation.* Rev. Mod. Phys. 93, 035002.
- Penington, G. (2019). *Entanglement wedge reconstruction and the information paradox.*

---

## About

This project was developed independently as part of ongoing self-directed research in quantum information theory, prior to formal graduate study. The simulation work complements theoretical reports on non-Markovian open quantum system models of black hole evaporation (2026) and quantum circuit models of the information paradox (2025).

For questions or collaboration: **aaqhilahamed2004@gmail.com**
