# Part V: Advanced Semantics and Theory
## Section 2: The Formal Lifting Theorem and Residual Semantics

---

### 1. The Formal Lifting Theorem

In this section, we formalize the semantics of **pattern-flow lifting** as a structure-preserving, conditionally defined transformation from abstract Patterns to executable FlowGraphs. We introduce a central theorem that captures the conditions under which lifting is successful, the structure of partial lifting, and the treatment of residual patterns.

This completes the bridge between static design-time reasoning and runtime instantiation.

---

### 1.1 Lifting as a Conditional Realization of the Functor \( F : \mathcal{A} \to \mathcal{E} \)

In Part III, we defined a functor \( F \) from the **Action Category** \( \mathcal{A} \) to the **Effect Category** \( \mathcal{E} \), preserving structure and associating every well-formed action with a corresponding effect.

However, this mapping is abstract and total. In practice, **instantiating** a pattern as an executable effect requires resolving capability constraints in a local context \( \Gamma \). Thus, the lifting operation is a **guarded, partial realization** of \( F \):

> **Definition (Capability-Guarded Realization):**  
> The lifting operation \( \text{Lift}_\Gamma \) is a context-sensitive mapping:
>
> \[ \text{Lift}_\Gamma : \mathcal{P} \rightharpoonup \mathcal{F}_{\text{raw}} \]
>
> where \( \mathcal{P} \) is the category of verified patterns, and \( \mathcal{F}_{\text{raw}} \) is the space of partially constructed FlowGraphs. This operation is:
>
> - **Total** when \( \Gamma \vDash \text{can\_execute}(a, r) \) for all pattern components
> - **Partial** otherwise, yielding a **residual pattern** \( P' \subset P \) of unlifted structure

This reflects the key insight:

> **Lifting is functorial where capability predicates hold; otherwise, it is conditionally compositional and yields residuals.**

---

### 1.2 The Lifting Theorem

We now state the central result governing pattern lifting.

> **Theorem (Lifting and Residual Decomposition):**  
> Let \( P \in \mathcal{P} \) be a statically verified pattern, and \( \Gamma \) a capability environment. Then:
>
> \[ \Gamma \vdash P \rightsquigarrow (F, P') \]
>
> where:
> - \( F \in \mathcal{F}_{\text{raw}} \) is the partially lifted FlowGraph
> - \( P' \subseteq P \) is the **residual pattern**
>
> such that:
> - If \( P' = \emptyset \), then \( F \in \mathcal{F}_{\text{valid}} \) and lifting corresponds to \( F(P) \)
> - Otherwise, \( F \) is a valid prefix of a full FlowGraph, and \( P' \) is subject to future lifting when \( \Gamma' \supseteq \Gamma \)

---

### 1.3 Proof Sketch

We proceed by structural induction over the composition of Patterns:

- **Base case (Atomic Pattern):**
    - If \( \mathsf{can\_execute}(\Gamma, a, r) = \text{true} \), then lifting succeeds:  
    \[ \Gamma \vdash \mathsf{Act}(a, r) \rightsquigarrow (\mathsf{Exec}(a, r), \emptyset) \]
    - Otherwise, produce a residual:
    \[ \Gamma \vdash \mathsf{Act}(a, r) \rightsquigarrow (\text{Deferred}, \mathsf{Act}(a, r)) \]

- **Inductive case (Sequential \( P_1 \circ P_2 \)):**
    - Lift \( P_1 \): \( \Gamma \vdash P_1 \rightsquigarrow (F_1, P_1') \)
    - Update \( \Gamma' \) with postconditions from \( F_1 \)
    - Lift \( P_2 \): \( \Gamma' \vdash P_2 \rightsquigarrow (F_2, P_2') \)
    - Then:
    \[ \Gamma \vdash P_1 \circ P_2 \rightsquigarrow (F_1 \circ F_2, P_1' \circ P_2') \]

- **Inductive case (Parallel \( P_1 \otimes P_2 \)):**
    - Lift each independently in disjoint environments
    \[ \Gamma_1 \vdash P_1 \rightsquigarrow (F_1, P_1'), \quad \Gamma_2 \vdash P_2 \rightsquigarrow (F_2, P_2') \]
    - Then:
    \[ \Gamma_1 \uplus \Gamma_2 \vdash P_1 \otimes P_2 \rightsquigarrow (F_1 \otimes F_2, P_1' \otimes P_2') \]

This constructs \( F \) and \( P' \) compositionally.

---

### 1.4 Properties of Residual Patterns

> **Definition (Residual Pattern):**  
> A **residual** \( P' \subseteq P \) is the sub-pattern that could not be lifted under capability environment \( \Gamma \).

Residuals may include:
- Actions with unmet capability predicates
- Lanes without resolved resource bindings
- Boundary elements requiring gate mediation

> **Lemma (Residual Completeness):**  
> \( P' = \emptyset \) iff all components of \( P \) are liftable under \( \Gamma \).

> **Lemma (Residual Preservation):**  
> If \( \Gamma \vdash P \rightsquigarrow (F, P') \), then \( \Gamma \cup \Delta \vdash P' \rightsquigarrow (F', \emptyset) \Rightarrow \Gamma \cup \Delta \vdash P \rightsquigarrow (F \oplus F', \emptyset) \)

Where \( F \oplus F' \) denotes FlowGraph extension by composition.

---

### 1.5 Implications

- **Partial Planning**: FlowGraphs can be constructed incrementally, using only available capabilities.
- **Progressive Evaluation**: Lifting can resume as capabilities are delegated or granted.
- **Attestation-Centric Execution**: Full FlowGraph instantiation corresponds to a complete attestation tree.

This theorem establishes a formal backbone for **composable, staged execution planning** in the UES.

---

### 1.6 Reflecting on Part III and IV: The Structural–Operational Bridge

The lifting theorem completes a missing link between the two worlds of UES:

- **Part III (Dual-Category Semantics)** focused on structural verification: the functor \( F: \mathcal{A} \to \mathcal{E} \), protocol-effect duality, and compositional correctness at the level of idealized morphisms.

- **Part IV (Correctness and Attestation)** brought those ideas into the runtime setting, modeling validation gates, gate chains, and attestation-based verification in distributed, multi-domain contexts.

**Lifting** exists precisely at the boundary between these two: it is the structured operationalization of a verified pattern into a runtime execution plan, guided by available capabilities and mediated by gate attestations.

It aligns with:
- The **Functorial Transfer Theorem** of Section IV.4, showing that properties proven at the structural level can persist when lifted
- The **Validation Chain Composition Theorem** of Section IV.3, since each lifted executor is ultimately subjected to runtime validation or rejection

In this sense, lifting makes the **bridge from F to V**: from static transformation (\( F \)) to attested, verified runtime execution (\( \mathcal{V} \)).

---

### 1.7 Diagram: Lifting with Residual Decomposition

```
         Pattern P
            │
    ┌───────┴────────┐
    │                │
Lifting succeeds   Lifting fails
   on P₁              on P₂
    │                 │
    ▼                 ▼
 Flow F₁         Residual P₂
    │
    ▼
Partial FlowGraph F
```

Where:
- \( P = P_1 \otimes P_2 \)
- \( \Gamma \vdash P_1 \rightsquigarrow F_1 \)
- \( \Gamma \not\vdash P_2 \Rightarrow \text{residual} \ P_2 \)
- Result: \( \Gamma \vdash P \rightsquigarrow (F_1, P_2) \)

---

### 1.8 Example: Two-Action Pattern with Missing Capability

Let:
- Pattern \( P = \mathsf{Act}(a, r_1) \circ \mathsf{Act}(b, r_2) \)
- \( \Gamma = \{ \mathsf{Cap}(a, r_1) \} \), but \( \mathsf{Cap}(b, r_2) \notin \Gamma \)

Then:
- \( \Gamma \vdash \mathsf{Act}(a, r_1) \rightsquigarrow (\mathsf{Exec}(a, r_1), \emptyset) \)
- \( \Gamma' \vdash \mathsf{Act}(b, r_2) \rightsquigarrow (\text{Deferred}, \mathsf{Act}(b, r_2)) \)

Therefore:
\[
\Gamma \vdash P \rightsquigarrow (\mathsf{Exec}(a, r_1) \circ \text{Deferred}, \mathsf{Act}(b, r_2))
\]

The FlowGraph \( F \) contains one valid executor and a deferred node, while the residual \( P' \) contains the second action pending future lifting.

---

This completes the lifting theorem and its grounding in the broader semantics of the UES.

