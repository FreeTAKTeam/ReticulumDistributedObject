# Reticulum Distributed Objects (RDO)

Protocol: Reticulum Distributed Objects
Acronym: RDO
Status: Draft
Version: 0.1
Transport: Reticulum Network Stack

## 1. Status of This Document

This document is the normative Draft 0.1 specification for Reticulum Distributed Objects (RDO).

The protocol is defined at the level of distributed-object semantics, evidence, resolution, replication, and projections. A conforming implementation MAY choose its own storage model, programming language, serialization library, query system, or simulation system.

**Publication note (non-normative):** The source working document available in the project material was labeled “v0.3-draft”. This repository publishes the requested Draft 0.1 specification and does not claim that the earlier working label was a released protocol version.

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **NOT RECOMMENDED**, **MAY**, and **OPTIONAL** in this document are to be interpreted as described in RFC 2119 and RFC 8174 when, and only when, they appear in all capitals.

Text marked as **Non-normative** explains intent or gives implementation guidance. It does not add protocol requirements.

## 2. Purpose and Scope

RDO defines a generic protocol abstraction for distributed objects over systems using the Reticulum Network Stack. It represents domain objects, properties, observations, assertions, relations, events, provenance, authority, resolution, replication, synchronization, and projections.

RDO is intended for asynchronous and intermittently connected systems in which multiple parties can observe, describe, update, or resolve the same logical record without a permanently authoritative server.

RDO is not a protocol for a particular application. It MUST remain independent of any specific enterprise architecture tool, situational-awareness application, database, programming language, or simulation engine.

RDO describes protocol meaning and required behavior. It does not prescribe a database schema, object-relational mapping, graph database, file layout, API style, or concrete wire encoding for the abstract operations in this draft.

## 3. Design Goals and Non-Goals

### 3.1 Design goals

RDO has the following goals:

1. Preserve evidence before applying a local resolution.
2. Distinguish the identity of an RDO record from the semantic identity of a real-world entity.
3. Represent conflicting observations and assertions without silent overwriting.
4. Keep provenance attributable through forwarding, replication, and federation.
5. Permit local, scoped, policy-driven resolution without requiring global consensus.
6. Support partial replication and disconnected additive merge.
7. Keep observed or resolved state separate from forecasts, plans, scenarios, and simulations.
8. Support independent authorities, semantic namespaces, trust domains, and selective disclosure.
9. Permit large external resources to be referenced without making them part of the object-semantic protocol.
10. Remain usable over Reticulum without requiring a particular Reticulum application layer.

### 3.2 Non-goals

RDO does not define:

- a particular application domain or mission vocabulary;
- a central registry or authoritative master server;
- a particular database or persistence technology;
- a particular simulation engine or programming language;
- a mandatory global conflict policy;
- a mandatory global naming service;
- a particular cryptographic identity format beyond the requirements stated here;
- a concrete wire representation for the abstract operations;
- a package or bulk-resource transfer protocol.

## 4. Terminology

**RDO node**: A participant that stores, originates, forwards, queries, observes, resolves, or projects RDO information.

**RDO object**: A logically identified record representing a subject of description. An RDO object is not an assertion that the record corresponds to one and only one real-world entity.

**RDO Identifier**: The stable, globally distinguishable identifier of an RDO object record.

**Assertion**: A separately identifiable statement about an object, relation, event, or other RDO subject.

**Property**: A predicate/value interpretation supplied by one or more assertions. A property is not a mutable slot that replaces its history.

**Observation**: Information recorded by a participant about something it observed, measured, received, or otherwise treated as evidence.

**Relation**: A separately identifiable semantic connection. RDO defines ObjectRelation and EventRelation.

**ObjectRelation**: A relation between RDO objects, such as a structural, ownership, membership, dependency, or semantic relation.

**EventRelation**: A temporal, causal, evidential, or other relation between events.

**Event**: An independently identifiable occurrence or record of an occurrence, observation, change, or stimulus. Events MAY refer to participating objects through typed roles.

**Provenance**: Information identifying the origin, custody, forwarding, observation, resolution, and other relevant history of an assertion or record.

**Semantic equivalence**: An explicit assertion that two or more independently identified records may denote the same or related real-world subject.

**Entity resolution**: Determining whether independently identified records refer to the same or related real-world subject.

**State resolution**: Deriving a usable local state from assertions, observations, relations, events, authority, trust, time, and policy.

**Resolution**: A traceable, derived result produced by a stated resolution policy over a stated evidence set.

**Current Resolution**: A local resolution representing the node's current derived view of available evidence. It is not a universal truth.

**Projection**: A derived representation of possible, planned, forecast, hypothetical, counterfactual, or simulated state.

**Federation**: Cooperation between independently governed namespaces or trust domains without requiring a single central semantic authority.

**Replica**: A node's retained subset of RDO information.

## 5. Protocol Model

### 5.1 Distributed and asynchronous operation

RDO assumes that messages may be delayed, duplicated, reordered, or lost. Nodes MAY be disconnected for arbitrary periods. A node MAY maintain a local view that differs from the view maintained by another node.

A node MUST NOT treat message arrival order as event order unless the relevant event semantics explicitly establish that ordering.

RDO MUST support operation without a permanent central authority. A deployment MAY designate authorities for particular scopes, but authority MUST remain scoped and expressible as evidence or policy rather than being inferred merely from possession of data.

### 5.2 Core model

RDO separates the following concerns:

- **Objects** provide persistent record identity.
- **Assertions and properties** provide statements about objects and their state.
- **Relations** provide explicit object-to-object and event-to-event connections.
- **Events** provide occurrences, observations, changes, and stimuli.
- **Resolutions** provide derived local interpretations of available evidence.
- **Projections** provide future, hypothetical, or simulated interpretations.

These categories MUST remain distinguishable even when an implementation materializes them in one local store.

### 5.3 Evidence before resolution

A node MUST retain the distinction between source assertions and any resolution derived from them.

A node MUST NOT replace a source assertion solely because a later assertion has arrived. It MAY derive a resolution that prefers one assertion for a stated scope and policy, but the source assertions and their provenance MUST remain available unless a separate privacy or deletion policy permits their removal.

Conflicting assertions are valid protocol data. A conflict MUST NOT be treated solely as a synchronization failure.

## 6. RDO Object Identity

### 6.1 Record identity

Every independently identified RDO object MUST have a stable, globally distinguishable RDO Identifier within the protocol's identifier rules.

The RDO Identifier identifies the logical RDO record. It does not, by itself, prove that the record is the same as a real-world entity.

An implementation MUST NOT silently change an RDO Identifier because a name, alias, external identifier, or semantic interpretation has changed.

### 6.2 Names and aliases

Names, labels, aliases, descriptions, and local display values are properties or assertions about an object. They are not primary object identity.

Two records MUST NOT be merged solely because their names or aliases match. A name match MAY be evidence for a separately represented semantic-equivalence assertion.

### 6.3 External identifiers

An external identifier MUST be represented with its namespace and identifier value. An external identifier MAY be associated with more than one RDO object when the available evidence is conflicting or the namespace is ambiguous.

An external identifier MUST NOT replace the RDO Identifier.

### 6.4 Semantic identity

Semantic identity is established through explicit, provenance-bearing equivalence assertions. It is not inferred as a protocol fact from matching names, addresses, labels, or local database keys.

Different RDO records MAY be resolved as equivalent, identical, probable matches, related subjects, or unresolved, according to the available evidence and local policy. The equivalence result MUST remain traceable to the records and evidence that support it.

## 7. Assertions and Properties

### 7.1 Assertion-based state

An RDO object MUST be represented as an assertion-based record rather than as an overwritten mutable document.

An assertion SHOULD identify, when applicable:

- the target object or RDO subject;
- a predicate or property;
- a value;
- the asserting or observing source;
- creation time;
- observation time;
- a validity interval;
- confidence;
- authority information;
- supporting references.

The absence of an optional value MUST NOT be interpreted as proof that the value did not exist.

### 7.2 Distinct assertions

Assertions about the same property MUST remain individually distinguishable when they originate from different sources, have different observations, or otherwise carry different provenance.

A node MAY materialize a convenient property view, but that view MUST be traceable to the assertions from which it was derived.

### 7.3 Negative assertions

A negative assertion MAY state that a property, relation, or event is absent, false, invalid, withdrawn, or otherwise not applicable for a stated scope or time.

A negative assertion MUST identify its scope and provenance sufficiently for a receiving node to distinguish it from absence of information.

### 7.4 Supersession and withdrawal

An assertion, definition, or object MAY explicitly supersede an earlier one.

Supersession MUST NOT mean silent deletion. The superseded item remains historical evidence unless a separate deletion or privacy policy applies.

Withdrawal, invalidation, revocation, inactivity, local deletion, and restricted-data erasure are distinct conditions and MUST NOT be treated as synonyms. An explicit withdrawal or invalidation SHOULD itself be represented traceably.

## 8. Provenance, Authority, and Trust

### 8.1 Provenance

Provenance MUST be retained for assertions, observations, relations, events, equivalence claims, resolutions, and projections where applicable.

Provenance SHOULD distinguish, when applicable:

- originator;
- observer;
- current holder;
- forwarder or replicator;
- resolver;
- authority or authorizing scope;
- creation and observation times;
- receipt or knowledge-availability time;
- supporting references.

Forwarding or replication MUST NOT replace the originator with the forwarder. A forwarder MAY add its own custody or receipt provenance.

### 8.2 Authority

Authority is scoped. A participant MAY have authority for one property, object set, namespace, time interval, or operation and not for another.

Possession of an assertion or replica MUST NOT by itself imply authority to modify, resolve, withdraw, or delete it.

Authority claims MAY be disputed. A dispute MUST be representable without destroying the original authority assertion.

### 8.3 Trust

Trust is local and policy-driven. A node MUST NOT assume that another node's trust judgment is globally binding unless a separate authority policy explicitly makes it so.

Trust decisions MAY consider source identity, authority, ownership, time, corroboration, history, local policy, and security state. The protocol does not require one universal trust algorithm.

## 9. Relations and Events

### 9.1 Relations

A Relation is a first-class, separately identifiable semantic record.

An ObjectRelation connects objects and MAY express structural, semantic, dependency, membership, ownership, or other domain-neutral relationships.

An EventRelation connects events and MAY express temporal, causal, evidential, derivational, or other relationships.

Relations MUST support provenance, validity, and historical retention where applicable. Conflicting relations MUST remain representable.

### 9.2 Events

An Event is an independently identifiable record of an occurrence, observation, change, or stimulus.

An event MAY assert changes in object state, but the event and the resulting object assertions MUST remain distinguishable.

Events SHOULD identify participating objects through typed roles when applicable. A participant role is semantic information and MUST NOT be inferred solely from the order in which object identifiers appear.

Events MAY be received out of order. A node MUST use explicit temporal or causal metadata, not network arrival order alone, when deriving event order.

### 9.3 Temporal metadata

RDO distinguishes, when known:

- creation time;
- observation or occurrence time;
- validity time or validity interval;
- receipt time;
- knowledge-availability time.

These times MUST NOT be collapsed when doing so would change the meaning of an assertion, event, resolution, or projection.

## 10. Semantic Equivalence and Resolution

### 10.1 Explicit equivalence

An equivalence claim MUST be represented explicitly and MUST carry provenance.

A node MUST preserve conflicting equivalence claims. It MAY select a local equivalence result, but the result MUST identify the evidence, policy, and scope used.

### 10.2 Entity resolution and state resolution

Entity resolution and state resolution are separate operations.

- Entity resolution determines whether records may denote the same or related real-world subject.
- State resolution derives a usable view of properties, relations, and lifecycle state from available assertions.

A node MUST NOT merge records or rewrite identifiers merely because entity resolution produced an equivalence result.

### 10.3 Resolution policies

A Resolution MUST identify, directly or by reference:

- the evidence set or knowledge boundary;
- the resolution scope;
- the policy or policy version;
- the resolver or resolving authority, where applicable;
- the time at which the resolution was derived;
- unresolved conflicts that materially affect the result.

A policy MAY consider source authority, trust, ownership, temporal validity, corroboration, confidence, and property-specific authority. No policy is globally mandatory.

### 10.4 Current Resolution

Current Resolution is a local derived result. It MUST remain distinguishable from source assertions and historical resolutions.

Current Resolution MUST remain traceable to its evidence and MUST NOT conceal material unresolved conflicts. A node MAY provide a simplified consumer view, but it MUST retain a way to determine that a conflict existed and which assertions were considered.

A Current Resolution at one node is not automatically a global consensus.

## 11. Object State, Lifecycle, and History

Object state is derived from assertions, relations, events, time, and resolution policy. It is not a single authoritative mutable value.

Lifecycle interpretations MAY include creation, active state, inactivity, retirement, withdrawal, or other domain-neutral statuses. A lifecycle interpretation MUST be scoped and traceable to its evidence.

Historical assertions, events, relations, and previous resolutions SHOULD be retained. A local absence MAY mean that the item is not present in that replica; it MUST NOT be interpreted as proof of non-existence.

## 12. Projection and Simulation

### 12.1 Projection

A Projection represents planned, forecast, hypothetical, counterfactual, scenario, or simulated state.

A Projection MUST have an identity distinct from the Current Resolution and SHOULD identify:

- the baseline resolution or evidence boundary;
- assumptions;
- the originating actor, process, or method;
- creation time;
- projected events;
- projected object states;
- projected relations;
- confidence or other stated qualification;
- lineage to source material.

A Projection MUST NOT automatically modify Current Resolution or source assertions.

Multiple projections MAY coexist for the same objects or baseline.

### 12.2 Scenarios

A Scenario MAY contain a baseline, assumptions, injected events, constraints, projected changes, and one or more resulting projections.

A Scenario MUST remain distinguishable from observed reality and from a Current Resolution.

### 12.3 Simulation abstraction

RDO defines a semantic boundary for simulation, not a simulation engine.

A simulation integration using RDO:

1. exports a Current Resolution or other identified Resolution as a baseline;
2. represents assumptions and injected events;
3. executes in an external simulation system;
4. imports results as RDO Projections;
5. preserves simulation provenance and lineage.

A simulation MUST NOT modify Current Resolution directly. Simulation output remains a Projection until a separate observation, assertion, or explicit promotion policy creates new current-state evidence.

RDO does not mandate a simulation engine, simulation algorithm, programming language, or execution environment.

## 13. Replication and Synchronization

### 13.1 Partial replication

A node MAY replicate only a selected scope of objects, assertions, relations, events, projections, or provenance.

A replica MUST be allowed to be incomplete. Absence from a replica MUST NOT imply non-existence.

Selective disclosure MAY limit the information exchanged. A receiving node MUST NOT represent undisclosed information as a negative assertion merely because it was not received.

### 13.2 Synchronization knowledge

Synchronization SHOULD allow peers to exchange knowledge about:

- information held by the peer;
- information locally missing;
- information believed missing at the peer;
- changed or newly available information;
- scope, authority, and disclosure limitations.

A synchronization process SHOULD avoid retransmitting information already known to be present, subject to uncertainty and policy.

### 13.3 Merge

Synchronization MUST preserve assertion identity, provenance, temporal metadata, and conflict information.

Disconnected histories MUST be mergeable without requiring last-writer-wins semantics. A merge MAY produce a new local Resolution, but it MUST NOT discard source assertions solely to make the result singular.

An implementation MAY use local deduplication, content addressing, or indexes, provided that such mechanisms do not remove required semantic distinctions.

### 13.4 Subscriptions and queries

A node MAY subscribe to changes within a stated scope and MAY issue queries over the information available to another node.

A subscription or query MUST be interpreted according to its scope, authority, disclosure policy, and time boundary. It MUST NOT be treated as proof that the responding node has complete knowledge.

## 14. Federation and Semantic Models

### 14.1 Independent namespaces

Federating nodes MAY use independent identifier namespaces, vocabularies, authority models, trust policies, and resolution policies.

Cross-namespace identity or semantic relationships MUST be represented by explicit equivalence or relation assertions. A federation MUST NOT require central migration of identifiers.

### 14.2 Federated resolution

A federation MAY exchange assertions and evidence while retaining local resolution. A node MUST be able to distinguish a remote resolution from the source assertions on which it was based.

Remote resolution MAY be used as evidence, but it MUST NOT silently become a local Current Resolution without the local policy permitting it.

### 14.3 Semantic models

Semantic definitions, predicates, relation types, event roles, and lifecycle meanings MUST be identifiable and versionable.

RDO MUST allow assertions created under older semantic definitions to remain interpretable. A semantic change SHOULD create a new identifiable version rather than silently changing the meaning of historical assertions.

## 15. External Resources and Relationship to RPP

Large binary content, snapshots, datasets, repositories, or other external resources SHOULD normally be referenced rather than embedded as an RDO property.

A reference to external content SHOULD provide enough information for a recipient to verify the intended content, such as an identifier, digest, version, or provenance reference when available.

RDO and RPP are related but distinct:

```text
Reticulum Network Stack
│
├── RDO
│   └── Distributed object semantics
│
└── RPP
    └── Package/resource distribution
```

An RDO implementation MAY use RPP for a referenced large resource, attachment, snapshot, repository, or other bulk artifact:

```text
RDO object
   │
   ├── normal object/event data -> RDO
   │
   └── large external resource -> optionally RPP
```

RDO MUST NOT require RPP. RPP MUST NOT be required to represent RDO objects. RPP MUST remain a package/resource protocol and MUST NOT be used to represent semantic relationships that belong to RDO.

An RPP package MAY contain serialized RDO information, but that does not create a runtime dependency between the protocols.

## 16. Core Protocol Operations

The following are abstract protocol operations. They define required semantic behavior, not a concrete message format:

- **Publish**: make an identified assertion, relation, event, projection, or other RDO record available to a selected peer or scope.
- **Query**: request information within a stated scope and authority.
- **Exchange**: transfer identified RDO records or evidence.
- **Synchronize**: compare knowledge and exchange missing or changed records.
- **Resolve**: derive a Resolution from a stated evidence set and policy.
- **Subscribe**: request future information within a stated scope.
- **Export baseline**: identify and provide a Resolution for external processing or simulation.
- **Import projection**: provide a traceable Projection and its lineage.

An implementation MAY combine or split these operations. It MUST preserve their semantic requirements and MUST NOT imply that a local resolution is global consensus.

The wire representation of these operations is outside this specification.

## 17. Reticulum Transport

RDO semantics are independent of a particular Reticulum application layer. When RDO is deployed over Reticulum, its transport binding MUST use Reticulum Network Stack primitives and MUST preserve the identity, integrity, provenance, scope, and ordering semantics defined by this document.

A binding MAY use Reticulum identities, destinations, links, packets, resources, or other RNS primitives as appropriate to the payload and link conditions.

RDO does not prescribe a concrete RNS packet type, field map, channel layout, or serialization. The transport binding MUST define how an implementation detects malformed, duplicated, replayed, unauthorized, or incomplete RDO data without changing RDO semantics.

RDO MAY be carried by another transport only through a separate binding that preserves these requirements. Such a binding is outside this specification.

## 18. Encoding

This draft does not mandate a concrete wire encoding. Implementations MAY use CBOR, another compact encoding, or a local representation before applying a transport binding.

Any encoding used for exchange MUST preserve:

- stable record and assertion identity;
- all required provenance;
- distinctions between absent, unknown, negative, and conflicting information;
- temporal metadata;
- semantic equivalence and resolution scope;
- projection lineage;
- version and semantic-model identifiers.

An implementation MUST NOT use encoding normalization as a reason to merge distinct assertions or discard provenance.

## 19. Error Handling and Rejection

A node MUST reject or quarantine data that is malformed, cannot be authenticated when authentication is required by policy, violates the receiving scope or authorization policy, or would overwrite required historical evidence.

A node SHOULD report errors with enough information for the sender or operator to distinguish malformed data, unsupported semantic versions, unauthorized data, failed integrity checks, unavailable evidence, and policy rejection.

Temporary transport failure MUST NOT be interpreted as semantic negation, object deletion, or proof that a remote object does not exist.

## 20. Security and Privacy Considerations

RDO deployments MUST support authenticated origin and integrity verification for exchanged data where the deployment's security policy requires them.

A receiving node MUST preserve provenance through forwarding and replication and MUST reject unauthorized modification of authenticated records.

Trust decisions MUST remain local or explicitly scoped. A remote trust result MUST NOT automatically override local policy.

Federated deployments SHOULD apply selective disclosure and minimize information exposed outside the relevant scope. Privacy policies MAY require withdrawal, redaction, or deletion, but such actions MUST be distinguishable from ordinary absence and SHOULD remain traceable where permitted.

References to external resources SHOULD be verified before their content is treated as the referenced resource.

RDO does not by itself guarantee confidentiality, anonymity, availability, or protection against traffic analysis. These properties depend on the Reticulum deployment, its identity and link configuration, the selected transport binding, and local policy.

## 21. Extensibility and Versioning

Extensions MAY add new predicates, relation types, event roles, projection metadata, semantic models, and transport bindings.

An extension MUST have an identifiable name or version and MUST define whether an unknown receiver may safely ignore it.

A receiver MUST reject or quarantine an extension when ignoring it would change the meaning of the record or violate a declared criticality rule.

Protocol versions use semantic versioning where practical:

- incompatible wire-level changes require a major version change;
- compatible protocol additions require a minor version change;
- editorial corrections may use a patch version;
- Git tags identify published specification versions;
- released tagged specifications are immutable;
- implementations SHOULD explicitly declare the RDO versions they support.

Semantic-model changes are separate from editorial changes and MUST be identifiable as stated in Section 14.

## 22. Protocol Registry Considerations

RDO does not require an external IANA registry for this Draft 0.1.

The specification repository MAY maintain registries for extension names, semantic-model identifiers, relation types, event roles, or proposal numbers. A local registry entry MUST define ownership, versioning, compatibility, and whether the entry is normative.

## 23. Conformance

An implementation conforms to the RDO core when it:

1. supports stable, globally distinguishable RDO object identity;
2. represents objects through distinguishable assertions;
3. preserves assertion provenance;
4. supports properties and conflicting property assertions;
5. supports ObjectRelation and EventRelation semantics;
6. supports events and typed participant roles where applicable;
7. preserves required temporal metadata;
8. represents explicit semantic equivalence;
9. distinguishes entity resolution from state resolution;
10. represents traceable Resolutions and Current Resolution;
11. supports partial replication;
12. supports disconnected merge without requiring last-writer-wins;
13. preserves source history and does not treat replica absence as non-existence;
14. supports integrity and source-authentication checks required by its security policy;
15. rejects malformed or unauthorized data.

A federation-capable implementation additionally supports independent namespaces, cross-namespace equivalence, scoped authority, selective disclosure, and federated resolution.

A projection-capable implementation additionally supports projection identity, baseline, lineage, and separation from Current Resolution.

A simulation-boundary implementation additionally supports Resolution export, assumptions and injected events, independent simulation execution, Projection import, and simulation provenance.

## 24. Formal Invariants

The following invariants are normative:

1. An RDO Identifier identifies a record and is not silently replaced by a name or external identifier.
2. Matching names do not merge records.
3. Equivalence is explicit and provenance-bearing.
4. Source assertions and their conflicts remain distinguishable from a Resolution.
5. A Resolution does not alter the source assertions from which it was derived.
6. Current Resolution is local and traceable; it is not automatically global consensus.
7. Forwarding preserves origin provenance.
8. Arrival order is not event order.
9. Replica absence is not proof of non-existence.
10. Disconnected merge does not require last-writer-wins.
11. Projection and simulation output do not automatically modify Current Resolution.
12. RDO does not require a particular application, database, or simulation engine.
13. RDO does not require RPP, and RPP does not require RDO.
14. RPP is not used to represent semantic relationships.

## 25. Examples (Non-Normative)

### 25.1 Conflicting enterprise records

Two federated parties create separate RDO records for what may be the same real-world service. Their names differ and their external identifiers belong to different namespaces. A third party may publish an explicit equivalence assertion with a result such as `PROBABLE_MATCH` or `IDENTICAL`, while retaining both original records and their provenance. A local resolver may use that assertion without requiring central identifier migration.

### 25.2 Conflicting state

One source asserts that an object is active and another asserts that it is inactive for overlapping validity intervals. A Current Resolution may prefer one assertion for a stated scope because of source authority or temporal policy, while retaining both assertions and recording the unresolved or disputed conflict.

### 25.3 Projection

A node exports a Current Resolution as a simulation baseline, adds an assumption and an injected event, and receives a result stating that a service may become unavailable. The result is represented as a Projection with lineage to the baseline and assumptions. It does not become observed or current state automatically.

## 26. References

- RFC 2119, “Key words for use in RFCs to Indicate Requirement Levels”.
- RFC 8174, “Ambiguity of Uppercase vs Lowercase in RFC 2119 Key Words”.
- Reticulum Network Stack documentation and protocol source: https://reticulum.network/
- Reticulum project repository: https://github.com/markqvist/Reticulum
- Reticulum Package Protocol: https://github.com/FreeTAKTeam/rpp-spec

