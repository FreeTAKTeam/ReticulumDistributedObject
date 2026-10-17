# Reticulum Distributed Objects (RDO)

Protocol: Reticulum Distributed Objects  
Acronym: RDO  
Status: Draft  
Version: 0.2-draft  
Transport: Reticulum Network Stack

## 1. Status of This Document

Normative **Draft 0.2** supersedes the untagged Draft 0.1 design. RDO is a distributed meta-metamodel (M3) based on Reticulum SINGLE Destinations. This draft remains subject to Python/Rust address-binding and wire interoperability tests.

Capitalized MUST, MUST NOT, SHOULD, SHOULD NOT and MAY follow RFC 2119 and RFC 8174.

## 2. Purpose and Modeling Levels

RDO is a minimal, application-independent distributed M3. It defines Object, Class, Property, Relationship, types, identity, ownership and object operations. M2 ontologies such as RPP, DAF and R3AKT specialize these constructs. M1 consists of particular packages, applications, devices, properties and relationships; M0 denotes the external subjects they describe.

Every addressable RDO instance has exactly one canonical identifier: its Reticulum SINGLE Destination hash. Exactly one owning twin may commit changes. RDO does not decide factual truth, perform file transfer itself or require any M2 ontology.

## 3. Goals and Exclusions

Draft 0.2 MUST provide:

1. A minimal self-describing M3: rdo:Object, rdo:Class, rdo:Property and rdo:Relationship.
2. Independent, first-class Property and Relationship objects with their own Destinations, types, versions and owners, not embedded authoritative attributes or edges.
3. Exactly one owning twin per object. A twin Reticulum Identity is distinct from the many object Destinations it may control.
4. Reticulum-native hash addressing, owner-authorized operations and safe retries.
5. Independent potentially incompatible descriptions of an external entity or relationship, without implicit reconciliation.
6. M2 ontologies without embedding domain rules in the RDO kernel.
7. Separation of structural validity from epistemic truth.
8. Reticulum Resources for large bytes, without an RDO object per transfer chunk.

Out of scope: distributed consensus, multi-writer merges, CRDT authority, automatic ownership transfer, atomic cross-twin transactions, global host directories, a general OWL reasoner, mandatory RDF storage and universal evidence/confidence algorithms.

## 4. Terminology

**RDO ID:** The Reticulum SINGLE Destination hash, represented as 16 bytes or an equivalent hex/IRI rendering.

**Object:** A separately addressed, owned, typed and versioned RDO record.

**Class:** An Object defining a reusable type.

**Property:** An Object with a source Object, property type and a typed literal or object reference as its value.

**Relationship:** An Object with a source Object, destination Object and relationship type.

**Twin Identity:** The persistent cryptographic Reticulum Identity controlled by one twin; it can derive multiple Object Destinations.

**Object Destination:** One independently addressed Reticulum SINGLE endpoint, identifying precisely one RDO record.

**Owning twin:** The sole authority allowed to commit changes to an Object it owns.

**M2 ontology:** A domain vocabulary defining specialized Classes, Properties and Relationships, such as RPP or DAF.

**Object Version:** An owner-issued revision marker, distinct from RDO ID.

**Operation Request/Result:** An authenticated request addressed to an Object Destination and the owner's recorded outcome.

**Epistemic assessment:** An evidence-based evaluation of claims, made outside RDO.

**Replica:** A non-owning cached copy of an RDO record.

**RPP digest:** A content-derived identifier of an immutable file/package revision, not an RDO ID.

## 5. Ownership

Every Object, Property, Relationship and published Class definition MUST have exactly one owning twin. A twin may own thousands of independent Reticulum Destinations using a shared persistent Twin Identity and distinct Destination names.

Only an object's owner can commit a new authoritative Object Version or perform an operation that changes it. Nonowners may create their own descriptions or send operation requests. The owner of a Relationship need not own its source or destination; creating a relationship MUST NOT change either endpoint.

Replicas MUST NOT receive private Twin Identity keys during normal replication. Controlled migration of an Identity, Destination derivation data and state to another host MAY preserve the same addresses; automatic ownership transfer and competing active owners remain excluded.

## 6. Reticulum Identity, Destinations and Resolution

### 6.1 Canonical ID

The Destination hash is the sole canonical RDO ID. An external IRI, UUID, package digest, Object Version or host address MUST NOT replace it.

### 6.2 Shared-Identity profile

The proposed Draft 0.2 naming convention is:

~~~text
Reticulum type:  SINGLE
App:             rdo
Aspects:         object.<token>
Identity:        owning twin's Reticulum Identity
RDO ID:          RNS.Destination.hash(twin_identity, "rdo", "object", token)
~~~

The token SHOULD be an immutable, randomly assigned 16-byte value rendered as 32 lowercase hex characters. It MUST be unique for Destinations under that Identity. It is *Destination derivation material*, not a competing object ID.

### 6.3 Hash-only resolution

RNS path requests resolve a Destination hash and signed public Identity, but its hash alone does not expose arbitrary Destination aspects. Therefore, each Object Destination MUST expose its immutable token and profile version in signed RNS announce/path-response application data. A client supplied only the RDO ID MUST verify the announce, retrieve the public Identity and token, recompute the Destination hash with the fixed app/aspects, and refuse any mismatch before addressing the object. Verified token/Identity bindings MAY be cached.

No IRI-to-host or object-to-owner directory is required. Announce application data MUST NOT contain a public semantic object catalog. Deployments SHOULD prefer on-demand paths over indiscriminate announces.

### 6.4 Profile verification

The prior Draft 0.1 profile used one Identity per object with a fixed name rdo.object. That is not automatically compatible with this shared-Identity binding. The new binding MUST be validated against Python RNS and LXMF-rs/RNS-rs before claiming network interoperability.

## 7. RDF Identity and Vocabulary

The bootstrap M3 is defined by ontology/rdo-core.ttl. Its stable ontology IRIs denote language terms; they are not an alternative to Destination hash identities for instantiated RDO objects.

An RDF adapter MAY render the same RDO ID as rdo:<destination-hash-hex>. Imported Dafrion or Sparx instance IRIs MAY be retained as attributed external aliases and MUST NOT silently imply object equivalence or another network address.

RDF triples MAY provide a query projection, but MUST NOT replace their independently owned Property and Relationship records. Domain ontology class and predicate IRIs, including DAF vocabulary terms, may remain unchanged.

## 8. Object State and Epistemic Separation

RDO validates structural type and reference rules, Destination identity, owner authority and state revision. It MUST NOT decide whether a Property or Relationship is factually true, whether competing descriptions denote the same real-world entity, or how persuasive their evidence is.

Independent twins MAY publish conflicting Property and Relationship objects. RDO MUST preserve their separate Destinations and owners; any belief, evidence fusion, equivalence or assessment belongs to the optional epistemic/domain layer.

A successful evidence submission MUST NOT be interpreted as acceptance of the claimed fact or as a change to unrelated domain objects. Projections and plans remain separate from accepted state.

## 9. Distributed M3 Ontology

### 9.1 Core classes

rdo:Object is the base. rdo:Class, rdo:Property and rdo:Relationship MUST inherit from rdo:Object. These four bootstrap terms are locally known; RDO MUST NOT require RPP or a network fetch to interpret its own M3. An independently published M2 type definition is a first-class Class object with one Destination and owner.

Each instance MUST identify a type. M2 ontologies may define subclasses and type restrictions without a mandatory general reasoning engine.

### 9.2 Property

An rdo:Property instance MUST be a separate RDO Object with its own Destination and owner. It MUST identify source (RDO ID), property type and one typed literal or RDO Object reference as value. Description and provenance are metadata of that Property object. The source Object does not contain the Property; a local index may expose it as though attached without changing authority.

### 9.3 Relationship

An rdo:Relationship instance MUST be a separate RDO Object with its own Destination and owner. It MUST identify source RDO ID, destination RDO ID and relationship type. Additional relationship attributes are separately addressed rdo:Property instances whose source is this Relationship object.

The source and destination may have distinct owners. The Relationship owner need not own either. Multiple twins may independently describe the same association (including reversed endpoints). RDO MUST NOT conflate or overwrite them.

### 9.4 M2 specialization

M2 ontologies define specialized types, permitted values, relation semantics and domain operations. RDO M3 MUST NOT contain RPP, DAF or R3AKT-specific types or decide which statements are epistemically valid.

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

## 15. RPP as M2, RNS Resources for Bytes

**RPP is an M2 ontology on RDO M3**, not a sibling ontology. Its Package, File and Manifest are specialized RDO Objects; ContentHash, MediaType and Size are Property types; Contains and Supersedes are Relationship types.

RPP control operations use RDO object Destinations and owner authorization. Its binary transfer implementation uses Reticulum Resources for manifests and file/chunk bytes, without requiring each chunk, acknowledgement or transfer window to be an RDO object.

An RPP manifest/package digest identifies immutable content and MUST NOT replace the RDO ID of the corresponding File/Package object. RDO remains usable without RPP; RPP's semantic model depends on RDO. See https://github.com/FreeTAKTeam/rpp-spec.

## 16. Reticulum Transport

Every independently addressable Object, Property, Relationship and published Class MUST have a unique Reticulum SINGLE Destination. The Draft 0.2 shared-Twin-Identity derivation and signed-token reconstruction are defined in Section 6.

Small operations MAY use RNS packets/Links and MUST authenticate requesters where required. The owner MUST separately authorize every state-changing operation. Large byte transfers MAY use RNS Resources, without an extra fragmentation protocol.

A separate compact wire profile MUST define exact request/response encoding, signed application-data layout, limits and version negotiation. This semantic specification alone does not establish interoperable wire implementations.

## 17. Security and Trust

An owning twin MUST securely persist its Reticulum private Identity and MUST NOT disclose it to replicas. A compromised shared Identity may jeopardize all associated Object Destinations.

Clients MUST verify signed announce data and the reconstructed Destination hash before authenticated operations. A valid Reticulum connection does not grant permission to mutate an object. The owner MUST enforce authorization independently.

Owning a description establishes who may update that description, not whether it is factually true. Avoid public object inventories and unbounded object/path queries.

## 18. Dafrion Example (Non-Normative)

Dafrion uses DAF as an M2 ontology over RDO M3. Twin A owns Object X, Property P1(source=X) and Relationship R1(source=X,destination=Y). Twin B owns Object Y, Property P2(source=Y) and Relationship R2(source=Y,destination=X).

P1, P2, R1 and R2 are separate RDO Objects, not embedded fields or edges. R1 and R2 may describe one real-world association but are independently owned. The epistemic layer evaluates such descriptions; RDO does not.

Dafrion's validation, governance, evidence and approval controls remain above RDO and MUST NOT be bypassed by remote operations.

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

M2 ontologies MAY add domain-specific subclasses, property types, relationship types and authorized operations. Their versions SHOULD be identifiable.

An extension MUST NOT replace Destination hashes with another canonical RDO ID, reintroduce embedded authoritative Property/Relationship state, or grant nonowners write authority while claiming Draft 0.2 conformance. Other address profiles and consensus models require explicit future specifications.

## 21. Conformance

A core Draft 0.2 implementation MUST support the four M3 classes and their inheritance, give each addressable instance one Reticulum SINGLE Destination hash, maintain one owning twin, and model Property and Relationship as separately owned/addressed Objects.

It MUST support owner-authorized operations with version preconditions and retained duplicate-request protection, independent M2 ontologies, and structural checks without epistemic truth decisions. It MUST NOT require RPP or assign RDO identities to every binary chunk.

Shared-Identity binding implementations MUST demonstrate hash-only discovery with signed token retrieval, correct outbound Destination reconstruction and authorized requests interoperating with Python Reticulum before claiming wire conformance.

## 22. Invariants

1. One addressable object has one canonical RDO ID: its Reticulum Destination hash.
2. One owning Twin Identity may control many separate Destinations.
3. Object, Class, Property and Relationship are first-class RDO Objects; the latter three inherit Object.
4. Property and Relationship instances are not embedded in source or endpoint Objects.
5. A Property has source, type and value; a Relationship has source, destination and type.
6. Different twins may author independent descriptions of the same real-world association.
7. Only the owner may commit changes; references grant no authority to mutate endpoints.
8. RDO validates structural/protocol correctness, never factual truth.
9. RPP is an M2 ontology over RDO, with bulk bytes handled by Reticulum Resources.
10. RDO identity is not an imported IRI, content digest, object version or host address.
11. Timeout is not rejection; replay safety and stale-version detection remain required.
12. No cross-owner atomicity or automatic ownership transfer is implied.

## 23. Example (Non-Normative)

~~~text
Twin A / Reticulum Identity A:
  X  = rdo:Object
  P1 = rdo:Property(source=X, type=Name, value="X")
  R1 = rdo:Relationship(source=X, destination=Y, type=dependsOn)

Twin B / Reticulum Identity B:
  Y  = rdo:Object
  P2 = rdo:Property(source=Y, type=Status, value="active")
  R2 = rdo:Relationship(source=Y, destination=X, type=supports)
~~~

Each named instance above has its own Destination and RDO ID, even when several share a Twin Identity. No object contains a relationship. Whether R1 and R2 refer to the same association is for an epistemic assessment.

RPP File F similarly owns no embedded ContentHash Property: a separate Property instance refers to F. File bytes are supplied using RPP over an RNS Resource; their content digest is not F's RDO ID.

## 24. References

- RFC 2119 and RFC 8174
- Reticulum Network Stack: https://reticulum.network/
- Reticulum reference: https://github.com/markqvist/Reticulum
- RDO ontology: ontology/rdo-core.ttl (this repository)
- RPP M2: https://github.com/FreeTAKTeam/rpp-spec
- RDF Schema: https://www.w3.org/TR/rdf-schema/

