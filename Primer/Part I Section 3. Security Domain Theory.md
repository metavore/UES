# Section 3: Security Domain Theory

## 1. Introduction to Security Domains

When designing secure systems, we often need to organize different components based on their security needs and trust relationships. Security domains provide a way to structure these divisions and reason about how they interact.

A security domain represents a region of a system where consistent security policies apply and trust assumptions hold uniformly. Think of it as a protected area where similar security rules govern everything inside, but when crossing to another domain, different rules might apply.

### 1.1 Definition and Conceptual Framework

> **Definition: Security Domain**
> 
> A security domain is a bounded region of computational authority characterized by:
> 1. Consistent security policies and constraints
> 2. Uniform trust assumptions
> 3. Common capability administration
> 4. Well-defined boundaries
> 5. Controlled interaction points with other domains

In practical terms, security domains might represent:
- Different microservices in an application
- Separate processes in an operating system
- Different organizational units in an enterprise
- Multiple tenants in a cloud environment

Security domains can be nested (domains within domains), overlapping, or entirely separate from one another. The relationships between domains form a critical part of the overall security architecture of a system.

Visually, we can represent security domains and their boundaries:

```
┌───────────────────────┐      ┌───────────────────────┐
│                       │      │                       │
│    Domain A           │      │     Domain B          │
│                       │      │                       │
│    ┌─────────┐        │      │       ┌─────────┐     │
│    │Resource │        │      │       │Resource │     │
│    │    A    │        │      │       │    B    │     │
│    └─────────┘        │      │       └─────────┘     │
│                       │      │                       │
└─────────┬─────────────┘      └────────────┬─────────┘
          │                                  │
          │           Boundary               │
          └────────────┬─────────────────────┘
                       │
                       ▼
                  Validation Gate
```

### 1.2 Types of Security Domains

Different types of security domains serve distinct purposes:

1. **Administrative Domains**: Regions under common administrative control
   - Example: All services managed by the DevOps team

2. **Trust Domains**: Regions sharing common trust assumptions
   - Example: All internal enterprise services that trust the same identity provider

3. **Policy Domains**: Regions where consistent security policies apply
   - Example: All components subject to HIPAA compliance requirements

4. **Execution Domains**: Regions with similar execution environments
   - Example: All containers running on the same Kubernetes node

5. **Data Domains**: Regions with consistent data sensitivity requirements
   - Example: All components handling PII (personally identifiable information)

In practice, these domain types often overlap or coincide, but distinguishing them conceptually helps reason about complex systems.

### 1.3 Domain Boundaries and Their Properties

Domain boundaries in the UES are **active morphisms**, not passive firewalls. They process incoming effects according to domain-local policies, validate capability claims, and emit cryptographic attestations. This behavior is formalized in later sections as the **Validation Category (𝓥)**, where each boundary crossing becomes a morphism with structured semantics.

Key properties of boundaries:
- **Semi-permeable**: Capabilities and effects may pass only if permitted by the local gate
- **Validating**: Each crossing attempt yields either a verified transformation or a rejection with structured proof
- **Transformative**: Effects may be reshaped or attenuated to comply with target domain policies
- **Attestable**: Successful crossings generate proofs that form part of the global trust chain

This model supports the lifting semantics in Part V, where patterns cross domains through inserted **validation gates**, forming a **proof-carrying execution structure**.


## 2. Boundary Models and Crossing Semantics

Security domain boundaries have specific operational semantics that govern how effects cross from one domain to another.

### 2.1 Mathematical Model of Boundaries

Formally, a boundary between domains can be modeled as a transformation function:

> **Definition: Domain Boundary**
> 
> A boundary between security domains A and B is a partial function:
> 
> B(e, c, d) → (e', p)
> 
> Where:
> - e is an effect attempting to cross from domain A to domain B
> - c is the set of capabilities presented for the crossing
> - d is the context of the crossing attempt
> - e' is the transformed effect (if crossing is permitted)
> - p is the proof of valid crossing (if crossing is permitted)
> 
> The function is partial because some effects may be rejected, in which case no transformed effect or proof is produced.

This mathematical model captures the essential property that boundaries actively transform effects rather than simply allowing or denying them.

### 2.2 Boundary Crossing Types

Effect crossings can be classified into several distinct types:

1. **Pass-Through**: The effect passes unchanged, suitable for fully trusted domains
   ```
   A:Effect ───► Gate ───► A:Effect
                  │
                  └───► Attestation
   ```

2. **Constrained Crossing**: The effect is enhanced with additional constraints
   ```
   A:Effect ───► Gate ───► A:Effect+Constraints
                  │
                  └───► Attestation
   ```

3. **Capability Attenuation**: The effect's required capabilities are attenuated
   ```
   A:Effect[FullCap] ───► Gate ───► A:Effect[AttenuatedCap]
                          │
                          └───► Attestation
   ```

4. **Transformation Crossing**: The effect is transformed to comply with target domain policies
   ```
   A:Effect ───► Gate ───► B:Effect'
                  │
                  └───► Attestation
   ```

5. **Delegated Crossing**: The effect triggers a proxy effect in the target domain
   ```
   A:Effect ───► Gate ───► B:ProxyEffect
                  │
                  └───► Attestation
   ```

6. **Rejection**: The effect is denied crossing, with appropriate error handling
   ```
   A:Effect ───► Gate ───► Rejection
                  │
                  └───► RejectionProof
   ```

Each crossing type has specific semantics and appropriate use cases.

### 2.3 Boundary Crossing Protocol

The process of an effect crossing a domain boundary follows a structured protocol:

```
Protocol: EffectCrossing

1. Presentation: The effect e and capabilities c are presented at boundary B
2. Validation: B verifies that c provides sufficient authority for e to cross
3. Policy Checking: B verifies that e complies with target domain policies
4. Transformation: B transforms e into e' to comply with target domain requirements
5. Proof Generation: B generates proof p that the crossing is valid
6. Authorization: B permits e' to enter the target domain with proof p
```

Visually, this protocol can be represented as:

```
      Effect
        │
        ▼
┌───────────────┐
│ 1. Present    │
└───────┬───────┘
        │
        ▼
┌───────────────┐
│ 2. Validate   │──► Reject if insufficient
└───────┬───────┘    capabilities
        │
        ▼
┌───────────────┐
│ 3. Check      │──► Reject if policy
└───────┬───────┘    violation
        │
        ▼
┌───────────────┐
│ 4. Transform  │
└───────┬───────┘
        │
        ▼
┌───────────────┐
│ 5. Generate   │
│    Proof      │
└───────┬───────┘
        │
        ▼
┌───────────────┐
│ 6. Authorize  │
└───────┬───────┘
        │
        ▼
   Transformed Effect
   with Attestation
```

This protocol ensures that all boundary crossings are explicitly validated, transformed as needed, and accompanied by verifiable proofs.

### 2.4 Example: Database Access Boundary

Consider a boundary between a web application domain and a database domain:

```javascript
// Effect in web application domain
appEffect = Effect(
  type: ResourceRequest,
  target: DatabaseTable,
  constraints: [],
  capabilities: [ApplicationDatabaseAccess],
  proof_generator: applicationProofGenerator
);

// Effect transformation at boundary
dbEffect = boundary.cross(appEffect, applicationCapabilities);

// Resulting effect in database domain
dbEffect = Effect(
  type: ResourceRequest,
  target: DatabaseTable,
  constraints: [MaxRowLimit(100), NoSensitiveColumns],
  capabilities: [RestrictedDatabaseAccess],
  proof_generator: combinedProofGenerator
);
```

Visually, this crossing would look like:

```
App Domain                  DB Domain
    │                           │
    │                           │
ResourceRequest                 │
[AppDBAccess]                   │
    │                           │
    ▼                           │
  ┌───┐                         │
  │ G │ Boundary Gate           │
  └───┘                         │
    │                           │
    └──────────► ResourceRequest
                 [RestrictedDBAccess]
                 {MaxRowLimit, NoSensitiveColumns}
                                │
                                ▼
                          Database Access
                         with Constraints
```

This example illustrates how the boundary adds constraints and attenuates capabilities during crossing, ensuring the effect complies with database domain policies.

## 3. Authority Transfer Mechanisms

Transferring authority across domain boundaries is a fundamental operation in security domain theory. The effect model provides precise mechanisms for controlled authority transfer that maintain security guarantees.

### 3.1 Types of Authority Transfer

Authority can be transferred across domains in several different ways:

1. **Delegation**: Explicitly transferring specific capabilities
   ```
   DomainA:Capability ───► DomainB:AttenuatedCapability
   ```

2. **Introduction**: Facilitating capability exchange between domains
   ```
   DomainA ───► Introduce(DomainB, DomainC) ───► DomainB ↔ DomainC connection
   ```

3. **Proxy Authorization**: Authorizing actions on another domain's behalf
   ```
   DomainA:Request ───► DomainB:ProxyAction ───► Resource access
   ```

4. **Attenuation**: Transferring restricted forms of capabilities
   ```
   DomainA:FullCapability ───► DomainB:RestrictedCapability
   ```

5. **Revocable Transfer**: Transferring authority that can be revoked
   ```
   DomainA:Capability ───► DomainB:RevocableCapability
                                        │
                       DomainA:Revoke ──┘
   ```

Each transfer type has distinct security properties and appropriate use cases.

### 3.2 Delegation Protocol

Delegation is the most direct form of authority transfer. The delegation protocol formalizes this process:

```
Protocol: Delegation

1. Verification: Domain A verifies it possesses capability c
2. Attenuation: Domain A prepares attenuated capability c' for domain B
3. Authentication: Domain A authenticates domain B
4. Transfer: Domain A transfers c' to domain B
5. Attestation: Domain A generates proof of delegation
6. Acceptance: Domain B validates and accepts c'
```

Visually, this protocol can be represented as:

```
   Domain A                               Domain B
     │                                      │
     ▼                                      │
┌──────────┐                                │
│1. Verify │                                │
└────┬─────┘                                │
     ▼                                      │
┌──────────┐                                │
│2. Prepare│                                │
└────┬─────┘                                │
     ▼                                      │
┌──────────┐                                │
│3. Auth   │◄───────Authentication─────────┐
└────┬─────┘                                │
     ▼                                      │
┌──────────┐                                │
│4. Delegate├─────────Capability c'─────────►
└────┬─────┘                                │
     ▼                                      │
┌──────────┐                                │
│5. Attest │                                │
└────┬─────┘                                │
     │                                      ▼
     │                                 ┌──────────┐
     └─────────Attestation────────────►6. Accept  │
                                       └──────────┘
```

This protocol ensures that delegation follows the principle that capabilities can only be transferred by entities that possess them, and that all delegations are explicitly attested.

### 3.3 Attenuation and Amplification

A critical principle in authority transfer is that capabilities can be attenuated but never amplified:

> **Principle: Attenuation Only**
> 
> When transferring authority across domain boundaries, capabilities can be:
> - Attenuated: restricted to provide less authority
> - Rearranged: combined or separated without changing total authority
> - Transformed: adapted to target domain requirements
> 
> But never:
> - Amplified: expanded to provide more authority
> - Forged: created without proper derivation
> - Elevated: granted higher privileges

This principle is enforced through the validation gates that mediate boundary crossings.

Visually, attenuation can be represented as:

```
FullCapability 
[read, write, delete]
      │
      ▼
┌──────────────┐
│ Attenuation  │
└──────────────┘
      │
      ▼
AttenuatedCapability
    [read]
```

### 3.4 Example: Service-to-Service Delegation

Consider a scenario where a front-end service delegates database access to a backend service:

```javascript
// Front-end service capability
frontendDbCapability = Capability(
  type: DatabaseAccess,
  target: CustomerTable,
  operations: [Read, Write],
  constraints: []
);

// Attenuated capability for backend service
backendDbCapability = frontendDbCapability.attenuate({
  operations: [Read],
  constraints: [OnlyActiveCustomers]
});

// Delegation to backend service
delegationEffect = Effect(
  type: CapabilityDelegation,
  target: [backendDbCapability, BackendService],
  constraints: [TimeLimit(hours(1))],
  capabilities: [DelegationCapability],
  proof_generator: delegationProofGenerator
);
```

Visually, this would look like:

```
Frontend Service                   Backend Service
       │                                 │
┌──────────────┐                         │
│ DB Capability│                         │
│ [Read, Write]│                         │
└──────┬───────┘                         │
       │                                 │
       ▼                                 │
┌──────────────┐                         │
│ Attenuate    │                         │
└──────┬───────┘                         │
       │                                 │
       ▼                                 │
┌──────────────┐                         │
│ DB Capability│                         │
│ [Read only]  │                         │
└──────┬───────┘                         │
       │                                 │
       ▼                                 │
┌──────────────┐                         │
│ Delegate     ├─────────────────────────►
└──────────────┘                         │
                                         ▼
                                   ┌──────────────┐
                                   │ DB Capability│
                                   │ [Read only]  │
                                   └──────────────┘
```

This example shows how a service can delegate an attenuated capability to another service, restricting the operations allowed and adding constraints, while generating proof of proper delegation.

## 4. Policy Enforcement Points

The conceptual boundaries between security domains are realized through concrete policy enforcement points. In the effect model, these are implemented as validation gates.

### 4.1 Validation Gates as Boundary Implementations

Validation gates are the practical implementation of domain boundaries:

> **Definition: Validation Gate**
> 
> A validation gate is a computational entity that:
> 1. Processes effects attempting to cross domain boundaries
> 2. Verifies capabilities and enforces security policies
> 3. Transforms effects to comply with target domain requirements
> 4. Generates proofs of policy compliance
> 5. Controls the flow of effects between domains

Validation gates operate according to explicit policies and maintain audit logs of all boundary crossings.

Visually, a validation gate can be represented as:

```
    Effect from Domain A
           │
           ▼
┌─────────────────────────┐
│                         │
│       Validation        │
│          Gate           │
│                         │
└───────┬─────────────┬───┘
        │             │
        ▼             ▼
 Validated Effect   Rejection
 with Attestation  with Proof
```

### 4.2 Gate Structure and Components

A validation gate consists of several key components:

```javascript
ValidationGate = {
  id: "unique-identifier",
  domain: sourceDomain,
  policies: securityPolicies,
  validators: validationFunctions,
  transformers: transformationFunctions,
  proof_generators: proofGeneratorFunctions,
  metrics: operationalMetrics
}
```

These components work together to implement the boundary crossing semantics described earlier.

Visually, the internal structure of a gate can be represented as:

```
           Input Effect
                │
                ▼
┌───────────────────────────────┐
│         Validation Gate       │
│                               │
│  ┌───────────────────────┐    │
│  │ Policy Enforcement    │    │
│  └───────────┬───────────┘    │
│              │                │
│              ▼                │
│  ┌───────────────────────┐    │
│  │ Capability Validation │    │
│  └───────────┬───────────┘    │
│              │                │
│              ▼                │
│  ┌───────────────────────┐    │
│  │ Effect Transformation │    │
│  └───────────┬───────────┘    │
│              │                │
│              ▼                │
│  ┌───────────────────────┐    │
│  │ Attestation Generation│    │
│  └───────────┬───────────┘    │
│              │                │
└──────────────┼────────────────┘
               │
               ▼
      Output Effect + Attestation
```

### 4.3 Gate Chains and Composition

Complex systems may require effects to cross multiple domain boundaries. This is facilitated through gate chains:

```
GateChain = [Gate1, Gate2, ..., GateN]
```

When an effect traverses a gate chain, it must satisfy the policies of each gate in sequence. The resulting effect accumulates transformations and constraints, and the final proof incorporates attestations from each gate.

Visually, a gate chain can be represented as:

```
Effect₀ ───► Gate₁ ───► Effect₁ ───► Gate₂ ───► Effect₂ ───► Gate₃ ───► Effect₃
             │            │            │            │            │
             ▼            │            ▼            │            ▼
          Proof₁ ─────────┘         Proof₂ ─────────┘         Proof₃
```

Gate chains have important compositionality properties:
- **Sequential Composition**: The effect of passing through gates in sequence is the composition of their individual effects
- **Policy Accumulation**: Constraints accumulate as effects pass through multiple gates
- **Proof Composition**: Attestations compose to prove compliance with all gates

These properties enable reasoning about complex cross-domain interactions by understanding the composition of the individual gates involved.

### 4.4 Gate Implementation Patterns

Several implementation patterns have emerged for validation gates:

1. **Inline Gates**: Embedded within the execution path
   - Direct code integration for high performance
   - Tightly coupled with execution environment

2. **Proxy Gates**: Operating as intermediaries between domains
   - Standalone services that intercept and process effects
   - Can be deployed independently from the communicating components

3. **Interceptor Gates**: Automatically intercepting cross-domain calls
   - Often implemented as middleware or aspect-oriented features
   - Minimal code changes required for existing systems

4. **Membrane Gates**: Encapsulating domains with a protective layer
   - All interactions with the domain pass through the membrane
   - Comprehensive policy enforcement for the entire domain

5. **Grid Gates**: Operating at the intersections of a domain matrix
   - Deployed at well-defined interaction points
   - Enable structured communication in complex multi-domain systems

Each pattern has specific advantages for different architectural contexts.

### 4.5 Example: Multi-Gate Validation

Consider an effect traversing a chain of gates from a user domain through an application domain to a database domain:

```javascript
// Initial effect in user domain
userEffect = Effect(
  type: DatabaseQuery,
  target: CustomerTable,
  constraints: [],
  capabilities: [UserQueryAccess],
  proof_generator: userProofGenerator
);

// After passing through application gate
appEffect = appGate.validate(userEffect, userCapabilities);
// appEffect now has additional constraints limiting query complexity

// After passing through database gate
dbEffect = dbGate.validate(appEffect, appCapabilities);
// dbEffect now has additional constraints limiting data access

// Final effect execution in database domain
result = dbDomain.execute(dbEffect, combinedProof);
```

Visually, this would look like:

```
User Domain       App Domain        DB Domain
    │                │                 │
    │                │                 │
DatabaseQuery        │                 │
[UserQueryAccess]    │                 │
    │                │                 │
    ▼                │                 │
  ┌───┐              │                 │
  │App│              │                 │
  │Gate──────────────►                 │
  └───┘              │                 │
                     │                 │
                DatabaseQuery          │
                [AppQueryAccess]       │
                {QueryComplexity}      │
                     │                 │
                     ▼                 │
                   ┌───┐               │
                   │DB │               │
                   │Gate───────────────►
                   └───┘               │
                                       │
                                 DatabaseQuery
                                 [DBQueryAccess]
                                 {QueryComplexity,
                                  DataAccess}
                                       │
                                       ▼
                                  DB Execution
```

This example shows how constraints accumulate and capabilities are verified at each boundary crossing, ensuring security policy enforcement across domains.
