# Quantum Gates, Circuits & Entanglement

An analytical and computational study of single-qubit unitary dynamics, non-commuting gate algebras, bipartite entanglement synthesis, Bell-state tomography, and subsystem density matrix metrics implemented in **Qiskit 2.x** and **Qiskit Aer**.

---

## Overview & Mathematical Conventions

This repository provides an end-to-end examination of fundamental quantum circuit operations, state space geometry, and bipartite quantum correlations. The executable notebook [`Quantum_Gates_Circuits_Entanglement_Final.ipynb`](Quantum_Gates_Circuits_Entanglement_Final.ipynb) integrates analytical derivations in Dirac notation with numerical simulations executing 1024 measurement shots per circuit on the `AerSimulator` backend.

### 1. State Space & Computational Basis
Single-qubit quantum states reside in a two-dimensional complex Hilbert space $\mathcal{H}_2 \cong \mathbb{C}^2$, spanned by the orthonormal computational basis states:
$$|0\rangle = \begin{pmatrix} 1 \\ 0 \end{pmatrix}, \quad |1\rangle = \begin{pmatrix} 0 \\ 1 \end{pmatrix}$$

A general single-qubit pure state is expressed as:
$$|\psi\rangle = \cos\left(\frac{\theta}{2}\right)|0\rangle + e^{i\phi}\sin\left(\frac{\theta}{2}\right)|1\rangle$$
where $\theta \in [0, \pi]$ and $\phi \in [0, 2\pi)$ define spherical coordinates on the unit Bloch sphere.

### 2. Single-Qubit Operator Generators
The unitary operators evaluated throughout this investigation include the Pauli operators and the Hadamard transformation:
$$X = \begin{pmatrix} 0 & 1 \\ 1 & 0 \end{pmatrix}, \quad Z = \begin{pmatrix} 1 & 0 \\ 0 & -1 \end{pmatrix}, \quad H = \frac{1}{\sqrt{2}}\begin{pmatrix} 1 & 1 \\ 1 & -1 \end{pmatrix}$$

These operators satisfy the algebraic identities:
$$X^2 = Z^2 = H^2 = I, \quad X Z = -Z X, \quad H X H = Z, \quad H Z H = X$$

### 3. Qiskit Little-Endian Register Indexing
Composite Hilbert spaces of $n$ qubits are constructed through the tensor product $\mathcal{H}^{\otimes n} = \bigotimes_{k=0}^{n-1} \mathcal{H}_k$. Qiskit adopts the **little-endian** indexing convention: qubit index $0$ occupies the least significant bit (LSB, rightmost position in the ket), and index $n-1$ occupies the most significant bit (MSB, leftmost position):
$$|q_{n-1} \dots q_1 q_0\rangle = |q_{n-1}\rangle \otimes \dots \otimes |q_1\rangle \otimes |q_0\rangle$$

### 4. Controlled-NOT (CNOT) Entangling Operator
The two-qubit Controlled-NOT operator with control qubit $q_0$ and target qubit $q_1$ applies a conditional Pauli-$X$ bit-flip:
$$CX_{(c=0, t=1)} |q_1 q_0\rangle = |q_1 \oplus q_0, q_0\rangle$$
In the ordered computational basis $\{|00\rangle, |01\rangle, |10\rangle, |11\rangle\}$, this operator corresponds to the matrix:
$$CX_{(c=0, t=1)} = \begin{pmatrix} 1 & 0 & 0 & 0 \\ 0 & 0 & 0 & 1 \\ 0 & 0 & 1 & 0 \\ 0 & 1 & 0 & 0 \end{pmatrix}$$

---

## Detailed Investigation & Numerical Results

### 1. Single-Qubit Gate Sequencing & Non-Commutativity
Unitary operators on Hilbert spaces do not generally commute ($[U_i, U_j] \neq 0$). We examine two tripartite gate sequences applied to the initial ground state $|\psi_0\rangle = |0\rangle$:
- **Circuit A:** $H \to X \to Z$
- **Circuit B:** $Z \to X \to H$

#### Analytical State Evolution
- **Circuit A ($H \to X \to Z$):**
  $$|0\rangle \xrightarrow{H} |+\rangle = \frac{|0\rangle + |1\rangle}{\sqrt{2}} \xrightarrow{X} |+\rangle \xrightarrow{Z} |-\rangle = \frac{|0\rangle - |1\rangle}{\sqrt{2}}$$
  *Algebraic mechanism:* The intermediate state $|+\rangle$ is an eigenstate of the Pauli-$X$ operator with eigenvalue $+1$ ($X|+\rangle = +1|+\rangle$), rendering the $X$ operation stationary on this state.
- **Circuit B ($Z \to X \to H$):**
  $$|0\rangle \xrightarrow{Z} |0\rangle \xrightarrow{X} |1\rangle \xrightarrow{H} |-\rangle = \frac{|0\rangle - |1\rangle}{\sqrt{2}}$$
  *Algebraic mechanism:* The ground state $|0\rangle$ is an eigenstate of the Pauli-$Z$ operator with eigenvalue $+1$ ($Z|0\rangle = +1|0\rangle$), meaning the initial $Z$ gate leaves the state vector invariant.

#### Operator and Trajectory Synthesis
1. **Unitary Product Equivalence:** Applying the Hadamard intertwining identity $X H = H Z$:
   $$U_A = Z X H = Z (H Z) = (Z H) Z$$
   $$U_B = H X Z = (Z H) Z$$
   $$U_A = U_B = \frac{1}{\sqrt{2}}\begin{pmatrix} 1 & -1 \\ -1 & -1 \end{pmatrix}$$
2. **State Fidelity:** Both circuits prepare the identical terminal statevector $|-\rangle$, yielding state fidelity:
   $$\mathcal{F}(|\psi_A\rangle, |\psi_B\rangle) = |\langle \psi_A | \psi_B \rangle|^2 = 1.000000$$
   with complex inner product $\langle \psi_A | \psi_B \rangle = 1.000000 + 0.000000j$.
3. **Bloch Sphere Geodesics:** Circuit A traverses through the positive equatorial coordinate $(+1, 0, 0)$ prior to rotating to $(-1, 0, 0)$. Circuit B traverses through the South pole $(0, 0, -1)$ before mapping to the negative equatorial axis $(-1, 0, 0)$.
4. **Empirical Measurement Statistics:** Across 1024 shots, both circuits display balanced binomial readout distributions matching theoretical expectations ($P=0.50$):
   - Circuit A: $|0\rangle = 526 \text{ counts}$, $|1\rangle = 498 \text{ counts}$
   - Circuit B: $|0\rangle = 526 \text{ counts}$, $|1\rangle = 498 \text{ counts}$

---

### 2. Custom Single-Qubit Circuit Synthesis
A 4-gate sequence satisfying the constraints (minimum 4 gates, repeated gate, non-session architecture) was implemented on $q_0$:
$$\text{Gate Sequence: } X \to H \to Z \to X$$

#### State Evolution from $|0\rangle$
1. $t=1$ ($X$ gate): $|\psi_1\rangle = X|0\rangle = |1\rangle$
2. $t=2$ ($H$ gate): $|\psi_2\rangle = H|1\rangle = |-\rangle = \frac{1}{\sqrt{2}}(|0\rangle - |1\rangle)$
3. $t=3$ ($Z$ gate): $|\psi_3\rangle = Z|-\rangle = |+\rangle = \frac{1}{\sqrt{2}}(|0\rangle + |1\rangle)$
4. $t=4$ ($X$ gate): $|\psi_4\rangle = X|+\rangle = |+\rangle = \frac{1}{\sqrt{2}}(|0\rangle + |1\rangle)$

#### Net Unitary & Challenge Equivalence
$$U_{\text{custom}} = X Z H X = X H = \frac{1}{\sqrt{2}}\begin{pmatrix} 1 & -1 \\ 1 & 1 \end{pmatrix}$$
Acting on $|0\rangle$, the circuit prepares $|\psi_{\text{final}}\rangle = |+\rangle$.
- **Simulation Counts (1024 shots):** $|0\rangle = 533 \text{ counts}$, $|1\rangle = 491 \text{ counts}$ ($P \approx 0.50$).
- **Challenge Alternative:** The two-gate sequence $Z \to H$ produces the identical statevector ($|+\rangle$) with state fidelity $\mathcal{F} = 1.000000$ and inner product $1.000000 + 0.000000j$.

---

### 3. Bipartite Systems: Product Superposition vs. Entanglement
Two canonical two-qubit configurations initialized in $|\psi_0\rangle = |00\rangle$ were characterized:
- **Circuit A (Independent Superposition):** Local Hadamards on both qubits ($H \otimes H$).
- **Circuit B (Entangled Bell State):** Hadamard on control qubit $q_0$ followed by $CX_{(0 \to 1)}$.

#### Mathematical Characterization & Subsystem Analytics

| Metric / Property | Circuit A (Independent Superposition) | Circuit B (Entangled State $|\Phi^+\rangle$) |
| :--- | :--- | :--- |
| **Statevector** | $\frac{1}{2}(|00\rangle + |01\rangle + |10\rangle + |11\rangle)$ | $\frac{1}{\sqrt{2}}(|00\rangle + |11\rangle)$ |
| **Separability** | Separable: $|+\rangle_1 \otimes |+\rangle_0$ | Non-Separable (Entangled) |
| **Schmidt Rank ($r$)** | $r = 1$ | $r = 2$ |
| **Reduced Density Matrix $\rho_0$** | Pure: $|+\rangle\langle+|$ | Maximally Mixed: $\frac{1}{2}I_2$ |
| **Subsystem Purity $\mathrm{Tr}(\rho_0^2)$** | $1.0000$ | $0.5000$ |
| **Von Neumann Entropy $S(\rho_0)$** | $0.0000\text{ ebits}$ | $1.0000\text{ ebit}$ |
| **Empirical Counts (1024 shots)** | $00: 243, \; 01: 272, \; 10: 278, \; 11: 231$ | $00: 521, \; 11: 503$ ($01: 0, \; 10: 0$) |
| **Subsystem Correlation** | Independent ($P(q_1, q_0) = P(q_1)P(q_0)$) | Deterministic Parity Correlation ($q_1 = q_0$) |

---

### 4. Bell State Challenge ($|\Psi^+\rangle$)
The assignment mandates synthesizing the Bell state:
$$|\Psi^+\rangle = \frac{|01\rangle + |10\rangle}{\sqrt{2}}$$
without copying the session circuit (which positioned $X$ prior to $H$ on qubit 0).

#### Circuit Architecture
The circuit applies a post-entanglement local Pauli-$X$ operation to the control register:
$$|\psi\rangle = (I_1 \otimes X_0) CX_{(0 \to 1)} (I_1 \otimes H_0)|00\rangle$$

1. **Superposition:** $(I \otimes H)|00\rangle = \frac{1}{\sqrt{2}}(|00\rangle + |01\rangle)$
2. **Entanglement:** $CX_{(0 \to 1)}\frac{1}{\sqrt{2}}(|00\rangle + |01\rangle) = \frac{1}{\sqrt{2}}(|00\rangle + |11\rangle) = |\Phi^+\rangle$
3. **Local Bit-Flip:** $(I \otimes X_0)\frac{1}{\sqrt{2}}(|00\rangle + |11\rangle) = \frac{1}{\sqrt{2}}(|01\rangle + |10\rangle) = |\Psi^+\rangle$

#### Physical Origin of Zero Amplitudes for $|00\rangle$ and $|11\rangle$
Expanding the terminal state:
$$|\Psi^+\rangle = 0\cdot|00\rangle + \frac{1}{\sqrt{2}}|01\rangle + \frac{1}{\sqrt{2}}|10\rangle + 0\cdot|11\rangle$$
By the Born rule, projective measurement probabilities are $P(x) = |\langle x | \Psi^+ \rangle|^2$, yielding $P(00) = 0$ and $P(11) = 0$. The state possesses odd parity under the joint operator $Z \otimes Z |\Psi^+\rangle = -|\Psi^+\rangle$, guaranteeing complete anti-correlation.
- **Empirical Simulation Counts (1024 shots):** $|01\rangle = 503 \text{ counts}$, $|10\rangle = 521 \text{ counts}$.

---

### 5. Complete Orthonormal Bell Basis
The four maximally entangled Bell states form an orthonormal basis for $\mathcal{H}_2 \otimes \mathcal{H}_2$:

$$\begin{array}{cccc}
\hline
\textbf{Bell State} & \textbf{Statevector} & \textbf{Observable Outcomes} & \textbf{Empirical Counts (1024 Shots)} \\
\hline
|\Phi^+\rangle & \frac{1}{\sqrt{2}}|00\rangle + \frac{1}{\sqrt{2}}|11\rangle & \{00, 11\} & 00: 510, \; 11: 514 \\
|\Phi^-\rangle & \frac{1}{\sqrt{2}}|00\rangle - \frac{1}{\sqrt{2}}|11\rangle & \{00, 11\} & 00: 510, \; 11: 514 \\
|\Psi^+\rangle & \frac{1}{\sqrt{2}}|01\rangle + \frac{1}{\sqrt{2}}|10\rangle & \{01, 10\} & 01: 514, \; 10: 510 \\
|\Psi^-\rangle & \frac{1}{\sqrt{2}}|01\rangle - \frac{1}{\sqrt{2}}|10\rangle & \{01, 10\} & 01: 514, \; 10: 510 \\
\hline
\end{array}$$

#### Invariance of Computational Probabilities Under Relative Phase
- $|\Phi^+\rangle$ and $|\Phi^-\rangle$ differ solely by an internal phase of $\pi$ radians ($e^{i\pi} = -1$) on the $|11\rangle$ amplitude.
- Computational measurement evaluates diagonal projection operators $\Pi_x = |x\rangle\langle x|$. Because $|-1/\sqrt{2}|^2 = |+1/\sqrt{2}|^2 = 1/2$, the phase factor is annihilated under modulus squaring.
- An identical condition applies to $|\Psi^+\rangle$ and $|\Psi^-\rangle$ on the $\{|01\rangle, |10\rangle\}$ subspace.
- **Phase Tomography:** The relative phase is detected by measuring in non-diagonal complementary observable bases. Under $X \otimes X$, the states are distinguished by eigenvalue:
  $$(X \otimes X)|\Phi^+\rangle = +1|\Phi^+\rangle, \quad (X \otimes X)|\Phi^-\rangle = -1|\Phi^-\rangle$$
  $$(X \otimes X)|\Psi^+\rangle = +1|\Psi^+\rangle, \quad (X \otimes X)|\Psi^-\rangle = -1|\Psi^-\rangle$$

#### Gram Matrix Orthonormality
Mutual inner products computed between all pairs confirm exact orthonormality:
$$\langle B_i | B_j \rangle = \delta_{ij}$$

$$\begin{pmatrix}
1.00 & 0.00 & 0.00 & 0.00 \\
0.00 & 1.00 & 0.00 & 0.00 \\
0.00 & 0.00 & 1.00 & 0.00 \\
0.00 & 0.00 & 0.00 & 1.00
\end{pmatrix}$$

---

### 6. Entanglement Detective Diagnostic
Three test circuits were subjected to statevector derivation, density matrix diagnostics, and 1024-shot simulation:

#### Diagnostic Summary Table

| Circuit | Gate Operations | Terminal State | Classification | Schmidt Rank | Entropy $S(\rho_0)$ | Empirical Counts (1024 Shots) |
| :---: | :---: | :---: | :---: | :---: | :---: | :--- |
| **1** | $H_0 \to CX_{(0 \to 1)}$ | $\frac{1}{\sqrt{2}}(|00\rangle + |11\rangle)$ | **Entangled** | $r = 2$ | $1.0000\text{ ebit}$ | $00: 503, \; 11: 521$ |
| **2** | $H_0, H_1$ | $|+\rangle_1 \otimes |+\rangle_0$ | **Separable** | $r = 1$ | $0.0000\text{ ebits}$ | $00: 267, \; 01: 272, \; 10: 236, \; 11: 249$ |
| **3** | $X_0 \to H_0, \; H_1 \to CX_{(0 \to 1)}$ | $|+\rangle_1 \otimes |-\rangle_0$ | **Separable** | $r = 1$ | $0.0000\text{ ebits}$ | $00: 267, \; 01: 272, \; 10: 236, \; 11: 249$ |

#### Mechanism of Circuit 3 Separability
In Circuit 3, prior to the CNOT operation, the control qubit is prepared in $|-\rangle_0 = \frac{1}{\sqrt{2}}(|0\rangle_0 - |1\rangle_0)$ and the target qubit is prepared in $|+\rangle_1 = \frac{1}{\sqrt{2}}(|0\rangle_1 + |1\rangle_1)$.
Applying the controlled operator $CX_{(0 \to 1)} = |0\rangle\langle 0|_0 \otimes I_1 + |1\rangle\langle 1|_0 \otimes X_1$:
$$CX_{(0 \to 1)}(|+\rangle_1 \otimes |-\rangle_0) = \frac{1}{\sqrt{2}}\left(|+\rangle_1 \otimes |0\rangle_0 - (X_1|+\rangle_1) \otimes |1\rangle_0\right)$$
Because $|+\rangle_1$ is an eigenstate of the Pauli-$X$ operator with eigenvalue $+1$ ($X|+\rangle = +1|+\rangle$):
$$X_1 |+\rangle_1 = |+\rangle_1 \implies CX_{(0 \to 1)}(|+\rangle_1 \otimes |-\rangle_0) = |+\rangle_1 \otimes |-\rangle_0$$
The CNOT operation acts as an exact identity transformation. The state remains an unentangled product state despite the application of a nominal entangling gate.

---

### 7. Conceptual Solutions

1. **Single-Qubit Superposition vs. Two-Qubit Entanglement:**  
   A single-qubit superposition is a linear combination of basis vectors within an isolated two-dimensional Hilbert space ($|\psi\rangle = \alpha|0\rangle + \beta|1\rangle \in \mathcal{H}_2$), possessing fully determined local physical properties. A two-qubit entangled state is a non-factorable vector in $\mathcal{H}_2 \otimes \mathcal{H}_2$ ($|\Psi\rangle \neq |\psi_1\rangle \otimes |\psi_0\rangle$); isolated subsystems do not possess independent statevectors and exhibit correlations that cannot be accounted for by local hidden-variable models.

2. **Role of CNOT in Bell State Creation:**  
   The CNOT gate acts as a conditional unitary that applies a Pauli-$X$ bit-flip to the target qubit conditioned on the control qubit occupying $|1\rangle$. When the control qubit is initialized in an equal superposition state $|+\rangle = \frac{1}{\sqrt{2}}(|0\rangle + |1\rangle)$ and the target qubit in $|0\rangle$, the linear action of CNOT transforms the product state $\frac{1}{\sqrt{2}}(|00\rangle + |01\rangle)$ into the non-separable state $\frac{1}{\sqrt{2}}(|00\rangle + |11\rangle)$, mapping local phase coherence into non-local bipartite quantum entanglement.

3. **Invariance of Measurement Probabilities:**  
   Under the Born rule, projective measurement evaluates the absolute squared modulus of state amplitudes, $P(x) = |\langle x | \psi \rangle|^2$. Expressing amplitudes in polar form as $\alpha_x = r_x e^{i\theta_x}$, the outcome probability is $P(x) = r_x^2$, which is invariant under variations of the phase angle $\theta_x$. Consequently, orthogonal states differing solely in relative phases—such as $|\Phi^+\rangle$ and $|\Phi^-\rangle$—yield identical probability distributions in the computational basis while remaining distinguishable in non-commuting bases such as $X \otimes X$.

4. **Statevector vs. Measurement Counts in Qiskit:**  
   A `Statevector` represents the exact complex wave function $|\psi\rangle$ in Hilbert space, retaining all probability amplitudes and phase relationships without state collapse. Measurement counts represent empirical integer frequencies obtained from a finite number of simulated or hardware shots, subject to wavefunction collapse and statistical sampling fluctuations ($\sigma \propto 1/\sqrt{N}$).

5. **Computational-Basis Dimensional Scaling:**  
   - 1 qubit: $2^1 = 2$ states ($\{|0\rangle, |1\rangle\}$)  
   - 2 qubits: $2^2 = 4$ states ($\{|00\rangle, |01\rangle, |10\rangle, |11\rangle\}$)  
   - 3 qubits: $2^3 = 8$ states ($\{|000\rangle, \dots, |111\rangle\}$)  
   - 5 qubits: $2^5 = 32$ states ($\{|00000\rangle, \dots, |11111\rangle\}$)  
   - $n$ qubits: $2^n$ states  
   **Mathematical Foundation:** The state space of an $n$-qubit register is the tensor product of $n$ two-dimensional single-qubit spaces, $\mathcal{H}^{\otimes n} = \bigotimes_{k=1}^n \mathbb{C}^2 \cong \mathbb{C}^{2^n}$. The dimension of the composite product space equals the product of subsystem dimensions: $\dim(\mathcal{H}^{\otimes n}) = \prod_{k=1}^n 2 = 2^n$.

---

## Execution & Environment Setup

### Prerequisites
- **Python Version:** 3.10+ (tested on Python 3.14)
- **Required Packages:**
  - `qiskit >= 1.0.0`
  - `qiskit-aer >= 0.14.0`
  - `matplotlib >= 3.7.0`
  - `pylatexenc >= 2.11`

### Running Locally
```bash
# 1. Create and activate a virtual environment
python -m venv .venv
# On Windows:
.venv\Scripts\activate
# On Unix/macOS:
source .venv/bin/activate

# 2. Install dependencies
pip install -r requirements.txt

# 3. Launch Jupyter and open the notebook
jupyter lab
```
Open [`Quantum_Gates_Circuits_Entanglement_Final.ipynb`](Quantum_Gates_Circuits_Entanglement_Final.ipynb) and select **Run All**.

### Running in Google Colab
The notebook incorporates an automated environment check in Cell 1:
```python
try:
    import qiskit
    import qiskit_aer
    import pylatexenc
except ImportError:
    !pip install -q --disable-pip-version-check qiskit qiskit-aer pylatexenc matplotlib
```
Upload [`Quantum_Gates_Circuits_Entanglement_Final.ipynb`](Quantum_Gates_Circuits_Entanglement_Final.ipynb) to Google Colab and click **Runtime $\to$ Run all**. Missing libraries are installed automatically during runtime initialization.

---

## Repository Structure

```
├── Quantum_Gates_Circuits_Entanglement_Final.ipynb  # Primary executed Jupyter notebook with all outputs
├── requirements.txt                                 # Dependency specifications
└── README.md                                        # Technical documentation and mathematical derivations
```
