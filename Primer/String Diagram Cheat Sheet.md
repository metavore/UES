### 🎛️ Unified Effect System String Diagram Cheat Sheet (Draft 1)

#### 🔹 **Basic Visual Vocabulary**

| Diagram Element | Meaning |
|------------------|--------|
| `───>` | Flow of a resource, action, or effect |
| `●` or `□` | An operation (morphism): can represent an action, effect, or gate |
| `x`-labeled wire | A typed lane (resource or capability flow) |
| Multiple wires in/out | Multi-lane group interfaces |
| `⊗` (parallel) | Horizontal juxtaposition of components |
| `∘` (sequential) | Vertical composition—output of one becomes input of the next |
| `∅` | No wire: denotes erasure, deletion, or nullity (like weakening in linear logic) |

---

#### 🔹 **Common Morphism Shapes in UES**

| Shape | Meaning | Example |
|-------|---------|---------|
| `●` | An **action** in category 𝓐 | Stateless computation |
| `□` | An **effect** in category 𝓔 | State-changing operation |
| `⬚` | A **validation gate** (morphism in 𝓥) | Domain boundary check |
| `▷` | A **protocol step** (or decorated morphism in a PROP) | e.g., commit, read, transfer |

---

#### 🔹 **Composition Patterns**

| Pattern | Semantics |
|---------|-----------|
| `A ──▶ □ ──▶ B` | A value flows into an effect and is transformed |
| `A ──▶ □ ──▶ B`<br>`C ──▶ □ ──▶ D` | Two effects composed **in parallel**: `⊗` |
| `A ──▶ □ ──▶ □ ──▶ C` | Sequential effect application: `∘` |
| `A ──▶ ⬚ ──▶ B` | Effect crosses a **validation gate** |
| `─────▶ ∅` | Resource is discarded or consumed |

---

#### 🔹 **Gate Semantics (from Part IV)**

| Symbol | Meaning |
|--------|---------|
| `⬚` | A **Validation Gate**: checks effect and emits an attestation |
| `⬚⊕` | A gate with **side output** for failure path |
| `⬚↓` | Rejection path (e.g., diagram splits into “valid” and “invalid”) |
| `⬚` with `p:` label | Emits proof `p` on success |
| `⬚` with `¬` with `\xacp:` label | Emits rejection proof on failure |

---

#### 🔹 **Decorations (from Decorated PROPs)**

| Symbol | Role |
|--------|------|
| `τ:` | Type annotation |
| `cap:` | Capability requirement |
| `err:` | Error flow wire |
| `μ:` | Decoration metadata (e.g., from a monoid or semiring) |

Decorations can appear **above** or **below** wires and boxes in diagrams.

---

#### 🧐 **How to Read a UES Diagram**

1. **Start at the top left.** Follow wires through morphisms.
2. **Each box is a transformation.** The wire types (labels) tell you what flows through.
3. **Look for gates** (`⬚`) that represent trust boundaries.
4. **Decorations** carry constraints, types, or capability data.
5. **When in doubt, read vertically as “do this, then that.”**

---

### 🧹 **Canonical String Diagram Patterns**

#### 1. Simple Validated Effect
```
Input ──▶ □ Effect ──▶ ⬚ Gate ──▶ □ Transformed ──▶ Output
                      │              │
                      └── cap: read  └── p₁: attestation
```

#### 2. Parallel Composition (⊗)
```
A ──▶ □ F1 ──▶
              │
              │
B ──▶ □ F2 ──▶  ══▶ (A', B')
```

#### 3. Sequential Composition (∘)
```
Input ──▶ □ F1 ──▶ □ F2 ──▶ Output
              │      │
              └── p₁  └── p₂
```

#### 4. Validation Chain with Short-Circuiting
```
e₀ ──▶ ⬚ G₁ ──▶ ⬚ G₂ ──▶ ⬚ G₃ ──▶ e₃
        │       │       │
        ▼       ▼       ▼
      ✗ r₁     ✗ r₂     ✗ r₃
```

#### 5. Branch and Merge (Convergent Validation)
```
             ┌────▶ □ F₁ ──▶ ⬚ G₁ ──┐
Input ──○ Split                     ║║║═──⬚ Merge ──▶ Final
             └────▶ □ F₂ ──▶ ⬚ G₂ ──┘
```

#### 6. Error Channel Extraction
```
Input ──▶ □ Effect ──▶ Output
              │
              ▼
          err: eₓ
```

---

_This cheat sheet provides both a visual vocabulary and reusable semantic patterns for reading, constructing, and verifying string diagrams in the Unified Effect System._

