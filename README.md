# Unified Effect System (UES) – Theory Layer

Welcome! This is a living document — a structured but still-evolving theory corpus behind a project I’ve been working on called **Capillary**. It’s a system for secure, compositional execution of effects across trust boundaries. Think: capability systems, algebraic graphs, decorated PROPs, and runtime-safe lifting, all woven into a cohesive model.

This repo is *just* the theoretical layer. The actual implementation lives in another workspace (written in Rust), but this is where I’ve been working through the core semantics, proofs, and design structure.

---

## What's in Here?

The repo is organized into five parts:

- **Part I: Foundations** – Capability-based security, effect semantics, and domain theory.
- **Part II: Mathematical Frameworks** – Category theory, decorated PROPs, and the algebraic backbone.
- **Part III: Dual-Category Structure** – Connecting patterns, flows, effects, and protocols.
- **Part IV: Correctness and Verification** – Validation gates, trust topologies, and verification transfer.
- **Part V: Execution Semantics** – Runtime lifting, rewriting systems, and the bridge from design to execution.

There are also some strategy notes and roadmap documents for keeping track of progress.

---

## Who is this for?

Mostly? Me. But also:

- Friends who like formal systems, runtime safety, and graph theory.
- Engineers or researchers interested in effect systems or capability models.
- Anyone curious about structured security and compositional semantics.

If you’ve ever said “I wish I could *prove* this system is safe to execute” and meant it, this might be your jam.

---

##  Disclaimer

This is a **work in progress**, and subject to ongoing edits. Some proofs are sketched, some ideas are half-baked, and the structure is still being smoothed out. But I’ve uploaded it here to keep myself accountable, to make sharing easier, and because honestly, I think some of this stuff is just cool.

---

## Contributing / Feedback

If you find something interesting, confusing, broken, or beautiful—feel free to open an issue, file a PR, or just send me a message. I’m especially interested in:

- Feedback on structure, clarity, or notation
- Suggestions for related work I might’ve missed
- Curious tangents that this sparks in your mind

You can also check out the [implementation repo (link pending)] when it's ready, where this theory is being put into practice.

---

## 💬 Why “Unified Effect System”?

Because it's a direct frontal assault on the default "separation of concerns", and because it structurally distinguishes between cause and effect. What started as a capability-secured graph runtime evolved into something a little stranger — a fusion of compositional algebra, decorated categorical semantics, and a local, verifiable way to model trust. Whether or not that turns out to be novel, I’ve learned a ton along the way. Maybe you will too.

Thanks for stopping by
