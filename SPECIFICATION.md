# Reticulum Distributed Objects (RDO)

Protocol: Reticulum Distributed Objects  
Acronym: RDO  
Status: Draft  
Version: 0.1  
Transport: Reticulum Network Stack

## 1. Status of This Document

This document is the normative Draft 0.1 specification for Reticulum Distributed Objects (RDO).

Draft 0.1 deliberately uses a constrained ownership model. Every RDO object has one Reticulum Destination as its canonical identifier and one owning twin. Only the owning twin determines and commits authoritative changes to that object. Multi-writer ownership, distributed consensus, automatic ownership transfer, and cross-owner transactions are outside Draft 0.1.

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **NOT RECOMMENDED**, **MAY**, and **OPTIONAL** in this document are to be interpreted as described in RFC 2119 and RFC 8174 when, and only when, they appear in all capitals.

Text marked as **Non-normative** explains intent or gives implementation guidance. It does not add protocol requirements.

## 2. Purpose and Scope

RDO defines a protocol model for addressable distributed objects over the Reticulum Network Stack.

An RDO object is simultaneously:

- a logical distributed object;
- an addressable Reticulum Destination;
- a unit of ownership and authorization;
- a target for read and operation requests.

RDO is intended for federated and intermittently connected systems in which objects may be referenced and cached by many participants, while exactly one owning twin remains responsible for determining authoritative changes to each object.

A federated twin that needs an authoritative operation on a foreign object MUST address the object's Reticulum Destination and submit the operation to its owner. It MUST NOT independently commit a competing authoritative state for that RDO object.

RDO is application-neutral. Dafrion is an intended reference consumer, but RDO MUST remain independent of any enterprise-architecture vocabulary, database, programming language, user interface, or simulation engine.

## 3. Design Principles and Non-Goals

### 3.1 Design principles

Draft 0.1 follows these principles:

1. **Destination-native identity.** The canonical identifier of an RDO object is its Reticulum Destination hash.
2. **One RDO identifier.** Application identifiers, including RDF IRIs, MAY be retained as aliases or external identifiers, but MUST NOT become a second authoritative RDO identity.
3. **Single ownership.** Every RDO object has exactly one owning twin.
4. **Owner-controlled change.** Only the owning twin may determine and commit authoritative changes to the object.
5. **Reticulum-native resolution.** Location and cryptographic identity are resolved through Reticulum. RDO MUST NOT require an IRI-to-owner or object-to-host directory as its primary resolution mechanism.
6. **Read-only federation of foreign state.** A non-owning twin MAY cache, query, reference, or project foreign state, but MUST submit authoritative operations to the owner.
7. **Safe operation requests.** Requests MUST be attributable, replay-safe for their declared semantics, and able to detect stale preconditions when a caller supplies them.
8. **Evidence remains distinguishable from accepted state.** Sources may disagree; the owner decides how its application resolves those inputs.
9. **Relations remain explicit.** Applications MAY represent relationships as independently governed RDO objects when relationship identity, properties, or provenance are required.
10. **Projection is not mutation.** Plans, forecasts, scenarios, and simulations do not change authoritative object state unless the owner accepts them through an operation.
11. **Large content stays outside the object protocol.** RDO MAY reference RPP or another content mechanism for large binary artifacts.

### 3.2 Non-goals for Draft 0.1

Draft 0.1 does not define:

- multi-writer authoritative objects;
- distributed consensus between owners;
- CRDT-based authoritative state;
- automatic conflict merging between competing writers;
- automatic object-ownership transfer;
- automatic failover that transfers ownership to a replica;
- atomic transactions spanning several owners;
- a global naming service;
- an IRI-to-owner routing registry;
- a universal truth, confidence, or source-fusion algorithm;
- a particular database or persistence technology;
- a package or bulk-resource transfer protocol.

These capabilities MAY be considered by later specifications or extensions after the single-owner model has been implemented and tested.

## 4. Terminology

**RDO object**: A distributed logical object identified and addressed by one canonical Reticulum Destination hash.

**RDO Identifier (RDO ID)**: The Reticulum Destination hash of an RDO object. The RDO ID is the object's canonical protocol identity and network address.

**Object Destination**: The Reticulum `SINGLE` Destination representing one RDO object.

**Object Identity**: The Reticulum Identity used to construct and control an Object Destination.

**Owning twin**: The twin that holds the private cryptographic material for an object's Object Identity and is responsible for authoritative operations on that object.

**Federated twin**: A twin that references, reads, caches, observes, or requests operations on objects owned by another twin.

**Replica**: A retained copy of owner-issued information about an RDO object. A replica is not an owner and does not receive ownership merely by possessing data.

**Operation Request**: An authenticated request submitted to an Object Destination asking the owner to perform an operation.

**Operation Result**: The owner's response to an Operation Request, including its status and, when applicable, the resulting object version or returned value.

**Object Version**: An owner-issued value that identifies a committed state of an RDO object. The exact representation is defined by the application binding or wire profile.

**Assertion**: A separately attributable statement or observation supplied as evidence about an RDO object or another subject.

**Current Resolution**: The authoritative state currently issued by the owning twin for an RDO object after applying its application rules to accepted information.

**Projection**: A possible, planned, forecast, hypothetical, counterfactual, or simulated state that has not been committed as authoritative object state.

**External identifier**: A non-RDO identifier associated with an object, such as an imported database key, RDF IRI, serial number, asset identifier, or application-specific URI.

## 5. Core Ownership Model

### 5.1 Exactly one owner

Every RDO object MUST have exactly one owning twin.

For Draft 0.1, ownership is established by control of the private Reticulum Identity material used by the object's Object Destination. A non-owning twin MUST NOT possess the private object identity merely to provide a replica.

The owning twin MAY host many RDO objects and therefore MAY control many Object Identities and Object Destinations.

### 5.2 Owner authority

Only the owning twin MAY commit a new authoritative state, withdraw authoritative state, or execute an application operation that changes the object.

A remote participant MAY provide evidence, observations, proposals, commands, or requested changes. These inputs become authoritative only if the owning twin accepts them according to its application rules.

Ownership establishes responsibility for the RDO object's accepted state. It does not establish that every claim made by the owner is objectively true.

### 5.3 Ownership transfer

Automatic ownership transfer is outside Draft 0.1.

An implementation MAY support administrative migration by explicitly moving the private Object Identity and its state to another host under controlled procedures, provided the canonical RDO ID remains unchanged. Such migration is an operational deployment procedure, not a federated consensus mechanism.

## 6. Destination-Native Object Identity

### 6.1 The RDO ID

The canonical RDO Identifier MUST be the Reticulum Destination hash of the Object Destination.

Conceptually:

```text
RDO ID = Reticulum Object Destination hash
```

There MUST NOT be a separate mandatory UUID, application URI, database key, or owner-directory key used as the protocol identity of the same RDO object.

A textual hexadecimal representation of the Destination hash is only a representation of the same RDO ID, not another identifier.

### 6.2 Object Destination construction

The Draft 0.1 Reticulum binding uses:

```text
Destination type: SINGLE
Application name: rdo
Aspect: object
Identity: unique Object Identity for the RDO object
```

A conforming Draft 0.1 Object Destination therefore uses the fixed logical destination name:

```text
rdo.object
```

with an Object Identity unique to that RDO object.

Using a fixed application name and aspect allows a peer that knows only the RDO ID to request a Reticulum path, recover the destination's public identity through normal Reticulum mechanisms, and reconstruct the outbound `rdo.object` Destination without requiring a second object-specific routing name.

Object-specific derivation data that is not the Reticulum Identity MUST NOT be required as an additional federation identifier in Draft 0.1.

### 6.3 Stability

An RDO ID MUST remain stable for the lifetime of the object.

Changing object name, properties, relationships, application state, hosting interface, IP address, Reticulum path, or physical node MUST NOT change the RDO ID.

Loss or replacement of the private Object Identity would produce a different Reticulum Destination and therefore a different RDO object identity. Implementations MUST persist Object Identities accordingly.

### 6.4 Reticulum is the resolution mechanism

A participant that holds an RDO ID MUST use Reticulum path and destination resolution to reach the object.

RDO MUST NOT require the following sequence as its primary routing mechanism:

```text
RDO ID -> application directory -> owner host -> network address
```

The required model is:

```text
RDO ID / Destination hash
        |
        v
Reticulum path discovery
        |
        v
Object Destination
        |
        v
Owning twin
```

An implementation MAY cache Reticulum path knowledge according to RNS behavior. Such cache is not an RDO naming directory.

### 6.5 Announces

An Object Destination MAY announce when appropriate for the deployment. Draft 0.1 does not require periodic announces for every object.

Deployments with many objects SHOULD avoid unnecessary announce traffic and SHOULD prefer Reticulum's normal on-demand path discovery and path-response behavior where practical.

Announce application data MUST NOT be required to carry a second RDO identifier.

## 7. Application Identifiers and Dafrion IRIs

### 7.1 External identifiers

An RDO object MAY carry any number of external identifiers. Each external identifier SHOULD include enough namespace information to interpret it correctly.

Examples include:

- a Dafrion or DAF IRI;
- a CMDB identifier;
- a device serial number;
- an imported Sparx identifier;
- a source-system URL.

External identifiers MUST NOT replace the RDO ID and MUST NOT be required to locate the owning twin.

### 7.2 One identity, several representations

Applications MAY render the RDO ID in application-compatible forms. For example, an RDF adapter MAY expose a deterministic IRI form such as:

```text
rdo:<destination-hash-hex>
```

This notation is a serialization of the same RDO ID. It MUST NOT introduce another resolution system.

An imported Dafrion IRI MAY be retained as an alias or external identifier attached to the RDO object. The imported IRI MAY remain important semantic data, but network federation MUST use the RDO ID.

### 7.3 Vocabulary IRIs

RDO object identity MUST NOT be confused with semantic vocabulary identity.

DAF class IRIs, property IRIs, ontology terms, schema identifiers, and other semantic-definition IRIs MAY remain unchanged. The single-ID rule applies to distributed RDO object instances, not to the vocabularies used to describe them.

## 8. Object State and Evidence

### 8.1 Authoritative state

The owning twin maintains the Current Resolution for its object.

The exact internal representation is application-specific. An owner MAY use mutable records, immutable revisions, RDF graphs, event sourcing, assertions, relational storage, or another persistence mechanism.

RDO does not require every implementation to adopt the same internal state model.

### 8.2 Assertions and observations

Applications MAY preserve source assertions and observations separately from Current Resolution.

When several sources describe the same object differently, RDO does not require a distributed agreement algorithm. The owning twin determines how those inputs affect the Current Resolution.

A source assertion SHOULD preserve, when applicable:

- source identity;
- observation or occurrence time;
- receipt time;
- asserted value or relation;
- confidence or qualification supplied by that source;
- supporting reference.

A confidence value MUST NOT be interpreted as owner acceptance unless the owning application's policy explicitly defines that meaning.

### 8.3 Foreign observations

A federated twin that observes information about a foreign object MAY submit that observation to the owner through an Operation Request.

The owner MAY:

- accept it as evidence;
- reject it;
- retain it without changing Current Resolution;
- require validation or human approval;
- derive a new Current Resolution.

The remote observer MUST NOT change the authoritative foreign object merely because it generated the observation.

### 8.4 Same real-world subject, different RDO objects

Two owners MAY independently create two RDO objects that later appear to describe the same real-world subject. They remain distinct RDO objects because their RDO IDs differ.

Applications MAY record an equivalence or correlation between them. Such a correlation MUST NOT merge their Reticulum identities or transfer ownership.

Entity-resolution algorithms are outside Draft 0.1.

## 9. Relations

A relation between objects MAY be represented as application data or as an independently addressable RDO object.

A relation SHOULD be represented as its own RDO object when it requires independent:

- identity;
- ownership;
- properties;
- provenance;
- lifecycle;
- authorization.

If represented as an RDO object, the relation follows the same single-ID and single-owner rules as every other RDO object.

A relationship connecting objects owned by different twins does not change ownership of either endpoint.

Creating or modifying a relation MUST NOT implicitly mutate both endpoint objects unless each required mutation is separately submitted to and accepted by its owner.

Atomic cross-owner relationship transactions are outside Draft 0.1.

## 10. Operation Model

### 10.1 General rule

All authoritative actions on an RDO object MUST be executed by its owning twin.

A federated twin performs a foreign operation as follows:

```text
RDO ID
  |
  v
Reticulum path resolution
  |
  v
Authenticated communication with Object Destination
  |
  v
Operation Request
  |
  v
Owner application logic
  |
  +--> APPLIED
  +--> REJECTED
  +--> PENDING
  +--> STALE
  +--> NOT_AUTHORIZED
```

### 10.2 Minimum Operation Request semantics

An Operation Request MUST identify:

- a request identifier;
- the requested operation;
- its arguments or payload.

A state-changing request SHOULD also identify the object version on which the requester based the operation when stale-state detection matters.

The authenticated requester identity SHOULD be obtained from the authenticated Reticulum session rather than trusted solely from an identity value inside the application payload.

### 10.3 Operation Result

An Operation Result SHOULD identify:

- the request identifier;
- the status;
- the current or resulting Object Version, when applicable;
- a returned value, when applicable;
- a rejection, authorization, validation, or stale-state reason, when applicable.

Initial status values are:

```text
APPLIED
REJECTED
PENDING
STALE
NOT_AUTHORIZED
NOT_FOUND
ERROR
```

`PENDING` permits an owner application such as Dafrion to retain the request for validation, governance, or human approval without falsely reporting that the requested change has already occurred.

### 10.4 Stale requests

If a request declares an expected Object Version and the owner's current version does not satisfy that precondition, the owner MUST NOT silently apply the operation as though the expected version were current.

It SHOULD return `STALE` with enough version information for the requester to reconsider the request.

### 10.5 Duplicate requests and retries

Networks may lose responses even after an operation has been applied. A requester therefore cannot infer from a timeout that the operation did not execute.

Owners MUST support duplicate detection for state-changing requests within an implementation-defined retention period. Repeating the same request identifier with the same operation payload MUST NOT apply the same logical change twice during that period.

If the same request identifier is reused with a different payload, the owner MUST reject it.

An implementation SHOULD persist the duplicate-detection record atomically with the state change when local storage permits it.

## 11. Disconnected Operation

### 11.1 Foreign objects

A federated twin MAY read and use a cached copy of a foreign RDO object while disconnected.

Such a read SHOULD expose, when available:

- the RDO ID;
- the cached Object Version;
- the time or synchronization point at which that state was obtained.

A cached copy MUST NOT be presented as proof that it is the owner's current state.

### 11.2 Foreign changes

If the owning twin is unreachable, a federated twin MUST NOT commit an authoritative change to the foreign RDO object.

It MAY store a pending Operation Request locally and submit it when the owner becomes reachable. Until the owner returns `APPLIED`, the requested change MUST remain distinguishable from authoritative state.

### 11.3 Locally owned objects

A twin MAY continue operating on objects it owns while disconnected from the rest of the federation. When connectivity returns, replicas can receive the owner-issued resulting state or history.

This is the primary disconnected-write model for Draft 0.1.

## 12. Replication and Synchronization

### 12.1 Owner-issued replication

An owner MAY publish or provide object state, revisions, assertions, relationships, events, or other records for replication.

A replica MUST preserve the RDO ID and owner attribution of replicated data.

Replication MUST NOT transfer ownership unless the private Object Identity is explicitly migrated through an administrative procedure outside normal synchronization.

### 12.2 No multi-writer merge requirement

Draft 0.1 does not require replicas to merge competing authoritative histories for the same RDO object because only one owner is permitted to create authoritative changes.

Synchronization MAY still need to detect:

- missing owner-issued versions;
- stale replicas;
- duplicate records;
- partial replication;
- interrupted transfers.

These are synchronization problems, not ownership conflicts.

### 12.3 Partial replication

A twin MAY retain only selected objects or selected data about them.

Absence from a replica MUST NOT imply non-existence.

Applications performing queries over a partial replica SHOULD expose the scope or freshness limits when those limits could materially change the answer.

## 13. Federation

### 13.1 Destination-based federation

Federation occurs by referencing RDO IDs and communicating with their Reticulum Destinations.

The canonical flow is:

```text
Foreign RDO reference
        |
        v
RDO Destination hash
        |
        v
Reticulum resolves path and destination identity
        |
        v
Owning twin receives query or operation
```

A deployment MAY maintain application indexes for search or discovery, but those indexes MUST NOT replace the Object Destination as the canonical identity and routing target.

### 13.2 No owner directory requirement

A conforming Draft 0.1 implementation MUST NOT require a global object-owner directory for normal object addressing.

The Reticulum Destination is itself the federation address.

### 13.3 Remote reads

A remote query MAY be served by the owner or, where application policy permits, by a replica.

A replica response SHOULD identify that it is a replica response and SHOULD expose its known Object Version or freshness information.

An operation that can change authoritative state MUST be submitted to the Object Destination controlled by the owner.

### 13.4 Cross-owner operations

An application workflow MAY require changes to objects with different owners. Draft 0.1 treats each owner operation independently.

The protocol does not provide an atomic commit across several Object Destinations. Applications requiring such a workflow MUST coordinate the requests at the application layer and handle partial acceptance explicitly.

## 14. Projection and Simulation

A Projection represents planned, forecast, hypothetical, counterfactual, or simulated state.

A federated twin MAY create local projections involving foreign objects without changing those objects.

A Projection SHOULD identify the RDO IDs and Object Versions used as its baseline.

Simulation or projection output MUST NOT become authoritative object state merely because it was computed successfully.

To change a foreign object based on a projection, the application MUST submit an Operation Request to that object's owner. The owner then applies its normal validation and authorization process.

## 15. External Resources and Relationship to RPP

Large binary content, snapshots, datasets, repositories, documents, models, images, or other external resources SHOULD normally be referenced rather than embedded directly in RDO state.

RDO and RPP are sibling protocols:

```text
Reticulum Network Stack
|
+-- RDO  -> distributed object identity and operations
|
+-- RPP  -> package/resource distribution
```

An RDO object MAY reference an RPP package or another externally verifiable resource.

RDO MUST NOT require RPP. RPP MUST NOT be used to replace RDO object identity or semantic relationships.

## 16. Reticulum Transport Binding

### 16.1 Required Destination type

Each RDO object MUST be represented by a Reticulum `SINGLE` Object Destination using the fixed Draft 0.1 application/aspect name `rdo.object` and a distinct Object Identity.

The Object Destination's hash is the RDO ID.

### 16.2 Path discovery

If a peer does not have a usable path to an RDO ID, it SHOULD use normal Reticulum path discovery.

Path discovery, path changes, transport hops, and interface changes MUST NOT change the RDO ID.

### 16.3 Identity recovery

A peer SHOULD use Reticulum's normal announce/path-response identity mechanisms to obtain or recall the public Object Identity required for encrypted communication with the Object Destination.

Because Draft 0.1 fixes the logical destination name to `rdo.object`, no object-specific application naming token is required in addition to the RDO ID.

### 16.4 Links and requester identification

State-changing operations SHOULD be carried over an authenticated Reticulum Link when the selected RNS mechanism supports the operation requirements.

When the owner must authorize the requester, the requester MUST identify itself through the Reticulum authentication mechanism required by the binding or application policy.

### 16.5 Requests, packets, and resources

A concrete implementation MAY use Reticulum requests, packets, links, channels, or resources according to payload size and operation needs.

Large state transfer or replication data SHOULD use Reticulum Resources rather than defining a second fragmentation protocol inside RDO.

The exact Draft 0.1 binary operation envelope MAY be defined in a separate wire-profile document. Such a profile MUST preserve the ownership, destination, request, version, duplicate-detection, and projection rules in this specification.

## 17. Security and Trust

The private Object Identity is security-critical because control of it establishes control of the object's Reticulum Destination in Draft 0.1.

An owning twin MUST protect and persist Object Identity private material.

A replica MUST NOT receive private Object Identity material as part of ordinary replication.

An authenticated network path does not itself authorize every application operation. The owner MUST apply its application authorization policy after identifying the requester.

Object ownership and source trust are separate. An owner can receive untrusted or disputed evidence without relinquishing ownership, and ownership does not make an observation true by definition.

Implementations SHOULD minimize metadata in announces and discovery traffic, especially where object names, types, relationships, or enterprise structure are sensitive.

## 18. Dafrion Binding Guidance (Non-Normative)

Dafrion is a primary intended consumer of RDO, but this section is not part of the application-neutral core.

### 18.1 Distributed Dafrion resources

When a Dafrion resource becomes an RDO-managed distributed object, its RDO ID should become its canonical distributed identity.

A Dafrion adapter can represent that destination hash as a deterministic RDF-compatible IRI form while preserving imported or historical Dafrion IRIs as external identifiers.

The important rule is:

```text
one distributed object
        =
one canonical RDO ID
        =
one Reticulum Object Destination hash
```

DAF vocabulary IRIs remain unaffected.

### 18.2 Dafrion ownership

A Dafrion twin owns the RDO objects for which it controls the Object Identities.

A second Dafrion twin can reference those objects in its local graph and can cache their state, but any operation that changes them is sent to their Object Destinations.

The owner then executes the request through the same semantic validation, governance, approval, persistence, and domain rules that would govern a local operation.

RDO should therefore sit below the Dafrion semantic and governance services, not bypass them.

### 18.3 Example

```text
Dafrion Twin A                        Dafrion Twin B
owns Application X                   owns Technology Y

RDO X destination                    RDO Y destination
       |                                     |
       |---- relationship/reference ---------|
       |                                     |
       +-- request change to Y -------------->
                                             |
                                      Twin B validates
                                      and decides
                                             |
       <----------- Operation Result --------+
```

Twin A never becomes an authoritative writer for Technology Y. Twin B does not need Twin A to discover an IP address, URL, server name, or Dafrion IRI for Y. The RDO ID is the Reticulum Destination used to reach it.

### 18.4 Evidence and live synchronization

Operational systems may submit observations to the owning Dafrion twin. The owner decides whether the observation:

- becomes evidence only;
- changes an observed operational view;
- requires validation;
- requires human approval;
- changes the accepted architectural state.

This keeps RDO transport and ownership simple while allowing Dafrion to implement richer evidence and digital-twin behavior above it.

## 19. Error Handling

Temporary transport failure MUST NOT be interpreted as object deletion, rejection, or proof that an object does not exist.

A timeout after a state-changing request MUST be treated as an unknown outcome until the requester can query or safely retry the request identifier.

A node SHOULD distinguish at least:

- no Reticulum path;
- authentication failure;
- authorization failure;
- stale version;
- application validation rejection;
- pending decision;
- malformed request;
- internal owner error.

## 20. Extensibility

Extensions MAY add new operation types, replication methods, semantic metadata, subscription behavior, or wire encodings.

An extension MUST NOT redefine the Draft 0.1 RDO ID as something other than the Object Destination hash while claiming Draft 0.1 identity conformance.

Future specifications MAY define multi-owner or transferable-ownership models, but they MUST explicitly state how they preserve or migrate canonical Reticulum object identity.

## 21. Conformance

A Draft 0.1 core implementation conforms when it:

1. uses a Reticulum `SINGLE` Destination as each RDO object's canonical identity;
2. uses the Object Destination hash as the sole RDO ID;
3. uses the fixed Draft 0.1 destination name `rdo.object` with a distinct Object Identity per object;
4. assigns exactly one owning twin to every object;
5. permits only the owner to commit authoritative changes;
6. uses Reticulum destination/path resolution rather than requiring an external owner-routing directory;
7. treats external identifiers and application IRIs as non-routing aliases or semantic identifiers;
8. allows foreign objects to be referenced and cached without granting write ownership;
9. routes foreign authoritative operations to the owning Object Destination;
10. distinguishes pending requests and projections from applied authoritative state;
11. supports stale-request detection when a request supplies a version precondition;
12. prevents duplicate application of the same retained state-changing request identifier;
13. does not require multi-writer merge or distributed consensus;
14. keeps RPP optional and separate from RDO object identity.

## 22. Formal Invariants

The following invariants are normative for Draft 0.1:

1. **One object, one RDO ID.** An RDO object has exactly one canonical RDO ID.
2. **The RDO ID is a Reticulum Destination.** The RDO ID is the Object Destination hash, not a UUID, IRI, host address, or directory key.
3. **One object, one owner.** Exactly one owning twin controls authoritative changes to an RDO object.
4. **Ownership follows Object Identity control.** Ordinary replication does not transfer the private Object Identity.
5. **Reticulum resolves the object.** RDO does not require an IRI-to-owner routing directory.
6. **IRIs are not a second RDO identity.** Application and source IRIs may be retained but do not replace or route around the RDO ID.
7. **Foreign replicas are not writers.** A cached foreign copy does not grant authority to change the object.
8. **Foreign changes are requests.** A non-owner submits an operation to the owner and waits for the owner's decision.
9. **Timeout is not rejection.** Loss of an operation response does not prove that the operation was not applied.
10. **Projection is not state.** Simulation, planning, and hypothetical output do not alter authoritative state without owner acceptance.
11. **No cross-owner atomicity is implied.** Operations on different owners are independently accepted or rejected.
12. **Semantic equivalence does not merge RDO IDs.** Two RDO objects may be judged to describe the same external subject while remaining separately owned RDO objects.
13. **RDO does not require a particular application or database.**
14. **RDO and RPP remain separate sibling protocols.**

## 23. Examples (Non-Normative)

### 23.1 Foreign operation

Twin A references an object whose RDO ID resolves through Reticulum to Twin B. Twin A wants to change one of the object's accepted properties. Twin A submits an Operation Request to the Object Destination. Twin B validates authorization and application rules, commits the change, and returns the resulting Object Version.

Twin A never writes a competing authoritative copy.

### 23.2 Disconnected federation

Twin A has a cached copy of an object owned by Twin B. Connectivity to Twin B is lost. Twin A may continue querying its cached copy and may prepare a pending requested change, but it cannot report that change as authoritative. When connectivity returns, the request is submitted to the object's Reticulum Destination.

### 23.3 Two records for one real-world entity

Twin A owns one RDO object and Twin B owns another. Later evidence suggests both describe the same physical transmitter. The application may create an explicit correlation between them. The objects retain separate RDO IDs and separate owners until an explicit future ownership or consolidation procedure says otherwise.

### 23.4 Dafrion IRI mapping

A legacy Dafrion resource has an imported IRI. When exposed as an RDO object, it receives an Object Destination whose hash becomes the canonical distributed RDO ID. The original IRI remains available as an external semantic identifier. Federated requests use the RDO ID, not an IRI lookup service.

## 24. References

- RFC 2119, "Key words for use in RFCs to Indicate Requirement Levels".
- RFC 8174, "Ambiguity of Uppercase vs Lowercase in RFC 2119 Key Words".
- Reticulum Network Stack documentation and protocol source: https://reticulum.network/
- Reticulum project repository: https://github.com/markqvist/Reticulum
- Reticulum Package Protocol: https://github.com/FreeTAKTeam/rpp-spec
