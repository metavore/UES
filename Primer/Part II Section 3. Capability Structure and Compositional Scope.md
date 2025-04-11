# Part II: Mathematical Frameworks
## Section 3: Capability Structure and Compositional Scope

---

### 1. Introduction: From Tokens to Type-Level Structure

In Part I, capabilities were introduced as *unforgeable tokens of authority*, necessary for executing effects and crossing domain boundaries. But as we build toward a compositional execution theory, it becomes clear that capabilities are not merely runtime checks or static annotations—they form a **structured, compositional system** of their own.

This section develops that system. We introduce the **algebraic and categorical semantics of capabilities**, showing how they interact with decorated PROPs and contribute to formal verification at every layer of the UES.

> **Key insight:** Capabilities are not just annotations; they are *typed morphism constraints* in a decorated symmetric monoidal category, and obey formal rules for transformation, delegation, and attenuation.

---

### 2. Capabilities as Decorations on Morphisms

Recall that in a PROP:
- Morphisms \( f : m \to n \) model multi-input, multi-output operations
- Decorations on morphisms attach metadata such as types, flow constraints, and error propagation

We now treat **capabilities as a special class of decorations**. For any morphism \( f \), we associate a **capability context**:

\[ \mathsf{cap}(f) \subseteq \mathcal{C} \]

where \( \mathcal{C} \) is the global set of all capability types in the system.

These contexts form partial functions over morphisms:

- **Capability requirement**: \( f \) may only be executed in an environment \( \Gamma \) such that \( \Gamma \supseteq \mathsf{cap}(f) \)
- **Decorated equivalence**: Two morphisms are capability-equivalent only if their decorations satisfy:

\[ f \equiv g \quad \Rightarrow \quad \mathsf{cap}(f) = \mathsf{cap}(g) \]

This establishes that **capability-checking is part of the morphism structure itself**, not an external validation step.

---

### 3. Compositional Semantics of Capability Contexts

Capability contexts compose alongside the PROP morphisms. Given two morphisms \( f : m \to n \) and \( g : n \to p \), we define:

\[ \mathsf{cap}(g \circ f) = \mathsf{cap}(f) \cup \mathsf{cap}(g) \]

\[ \mathsf{cap}(f \otimes h) = \mathsf{cap}(f) \cup \mathsf{cap}(h) \]

These rules guarantee that:
- **Sequential composition** accumulates capability requirements
- **Parallel composition** merges them, under independence assumptions

Thus, capability contexts behave as **compositional side conditions**, which mirror how flow types, errors, and resources propagate through the system.

---

### 4. Capability Attenuation and Delegation

We introduce an **order-theoretic view** of capabilities. Let \( \sqsubseteq \) define a **capability attenuation relation**, where:

\[ c' \sqsubseteq c \quad \text{means } c' \text{ is a restricted or scoped form of } c \]

This relation is:
- **Reflexive**: \( c \sqsubseteq c \)
- **Transitive**: \( c' \sqsubseteq c \wedge c'' \sqsubseteq c' \Rightarrow c'' \sqsubseteq c \)
- **Not symmetric**: attenuation is irreversible unless revalidated

We define a **capability morphism**:

\[ \delta : c \to c' \]

as a structural transformation of authority, valid only if \( c' \sqsubseteq c \). These morphisms serve as:
- **Delegation steps**: yielding attenuated tokens
- **Gate-mediated transformations**: recorded in validation proofs
- **Lifting predicates**: controlling whether pattern nodes can be executed

---

### 5. Capability Structures over Patterns

In decorated PROPs, **patterns** are compound morphisms \( P : m \to n \) constructed from atomic actions. Each pattern thus has an associated **capability requirement**:

\[ \mathsf{cap}(P) = \bigcup_{v \in \text{Vertices}(P)} \mathsf{cap}(v) \]

This gives rise to several useful notions:

- **Pattern subtyping**: \( P_1 \sqsubseteq P_2 \) iff \( \mathsf{cap}(P_1) \subseteq \mathsf{cap}(P_2) \)
- **Capability-bounded lifting**: A pattern \( P \) can only be lifted in a context \( \Gamma \) if \( \Gamma \supseteq \mathsf{cap}(P) \)
- **Residual generation**: If only some required capabilities are present, lifting produces a partial FlowGraph and a residual sub-pattern (see Part V)

This integrates directly with the lifting rules in Part V. Capabilities **bound the scope of pattern instantiation**, defining what segments can be concretely realized.

---

### 6. Capability-Aware Equivalence and Optimization

In Part V, we define equivalence of patterns and effects up to proof and capability structure. This section lays the groundwork:

> **Definition (Capability-Equivalence):**  
> Two patterns \( P_1, P_2 \) are **capability-equivalent**, \( P_1 \equiv_\mathcal{C} P_2 \), if:
> \[
> \mathsf{cap}(P_1) = \mathsf{cap}(P_2) \quad \text{and} \quad P_1 \equiv P_2
> \]

Rewrites (e.g. fusions, gate elisions) are **only valid** if they preserve this equivalence. This ensures the **preservation of delegated authority** during optimization.

---

### 7. Future Extensions

This section opens the door to future enrichment of the capability model:

- **Capability types and kinds**: e.g. ReadCap<T>, WriteCap<T>, DelegableCap<T>
- **Probabilistic capabilities**: time-limited or rate-limited authorities
- **Capability lattices**: formalizing joins, meets, and constraints
- **Capability-aware pattern rewriting**: as a verified optimization layer

These ideas link back to runtime lifting, security gate chains, and compositional verification via attestation logic.

The role of capabilities in decorated string diagrams—including flow annotations and validation gates—will be visualized in Section 4, and later formalized in the cross-domain theory of Part III.