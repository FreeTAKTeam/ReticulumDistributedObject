# Reticulum Distributed Objects (RDO)

Protocol: Reticulum Distributed Objects  
Acronym: RDO  
Status: Draft  
Version: 0.3-draft\
Transport: Reticulum Network Stack

## 1. Status of This Document

Normative **Draft 0.3** adds selective, durable replication using an optional LXMF carrier to Draft 0.2. The distributed M3, shared-Twin-Identity address binding and single-owner operation model remain unchanged. The accepted decision is recorded in [RDO-0001](proposals/RDO-0001-selective-lxmf-replication.md).

This draft specifies replication semantics, not a completed interoperable wire format. The shared-Identity binding and replication encoding still require Python/Rust interoperability tests. The bootstrap vocabulary in ontology/rdo-core.ttl is unchanged from Draft 0.2.

Capitalized MUST, MUST NOT, SHOULD, SHOULD NOT and MAY follow RFC 2119 and RFC 8174.

## 2. Purpose and Modeling Levels

RDO is a minimal, application-independent distributed M3. It defines Object, Class, Property, Relationship, types, identity, ownership and object operations. M2 ontologies such as RPP, DAF and R3AKT specialize these constructs. M1 consists of particular packages, applications, devices, properties and relationships; M0 denotes the external subjects they describe.

Every addressable RDO instance has exactly one canonical identifier: its Reticulum SINGLE Destination hash. Exactly one owning twin may commit changes. RDO does not decide factual truth, perform file transfer itself or require any M2 ontology.

## 3. Goals and Exclusions

Draft 0.3 MUST provide:

1. A minimal self-describing M3: rdo:Object, rdo:Class, rdo:Property and rdo:Relationship.
2. Independent, first-class Property and Relationship objects with their own Destinations, types, versions and owners, not embedded authoritative attributes or edges.
3. Exactly one owning twin per object. A twin Reticulum Identity is distinct from the many object Destinations it may control.
4. Reticulum-native hash addressing, owner-authorized operations and safe retries.
5. Independent potentially incompatible descriptions of an external entity or relationship, without implicit reconciliation.
6. M2 ontologies without embedding domain rules in the RDO kernel.
7. Separation of structural validity from epistemic truth.
8. Reticulum Resources for large bytes, without an RDO object per transfer chunk.

The optional replication profile in Sections 12.4-12.6 and 16.2-16.6 adds selected-peer distribution, persistent local replicas and delayed delivery through existing LXMF propagation nodes. Core RDO conformance does not require that profile or an LXMF dependency.

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

**Replica:** A non-owning copy of an RDO record. A copy selected for offline availability under the optional replication profile is persisted locally; it is not subject to ordinary transient-cache eviction.

**Device peer:** An application installation with its own recipient identity and LXMF delivery Destination. Several device peers may belong to the same person.

**Replication selection:** The records and content an application is authorized to send to a particular device peer. A selection may describe individual records or an application-defined set of records.

**Replication receipt:** An application acknowledgement identifying the record revisions durably accepted by a recipient. It is distinct from transport delivery, propagation custody and an owner Operation Result.

**Propagation node:** An LXMF service that stores encrypted messages for later delivery. Its message spool is separate from an application's local RDO replica store.

**RPP digest:** A content-derived identifier of an immutable file/package revision, not an RDO ID.

## 5. Ownership

Every Object, Property, Relationship and published Class definition MUST have exactly one owning twin. A twin may own thousands of independent Reticulum Destinations using a shared persistent Twin Identity and distinct Destination names.

Only an object's owner can commit a new authoritative Object Version or perform an operation that changes it. Nonowners may create their own descriptions or send operation requests. The owner of a Relationship need not own its source or destination; creating a relationship MUST NOT change either endpoint.

Replicas MUST NOT receive private Twin Identity keys during normal replication. Controlled migration of an Identity, Destination derivation data and state to another host MAY preserve the same addresses; automatic ownership transfer and competing active owners remain excluded.

## 6. Reticulum Identity, Destinations and Resolution

### 6.1 Canonical ID

The Destination hash is the sole canonical RDO ID. An external IRI, UUID, package digest, Object Version or host address MUST NOT replace it.

### 6.2 Shared-Identity profile

The shared-Identity naming convention introduced in Draft 0.2 and retained in Draft 0.3 is:

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

An RDF adapter MAY render the same RDO ID as urn:rdo:<destination-hash-hex>. Imported Dafrion or Sparx instance IRIs MAY be retained as attributed external aliases and MUST NOT silently imply object equivalence or another network address.

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

An rdo:Property instance MUST be a separate RDO Object with its own Destination and owner. It MUST identify source (RDO ID), property type and one typed literal or RDO Object reference as value. Domain attributes such as description are independent rdo:Property objects referencing that Property as their source; only protocol metadata (including identity, type and version) is intrinsic. The source Object does not contain the Property; a local index may expose it as though attached without changing authority.

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

This is the primary disconnected-write model for Draft 0.3. The optional replication profile does not change it.

## 12. Replication and Synchronization

### 12.1 Owner-issued replication

An owner MAY publish or provide object state, revisions, assertions, relationships, events, or other records for replication.

A replica MUST preserve the RDO ID and owner attribution of replicated data.

Normal replication MUST NOT transfer ownership or private Twin Identity keys. Controlled migration remains a separate administrative procedure under Section 5.

### 12.2 No multi-writer merge requirement

Draft 0.3 does not require replicas to merge competing authoritative histories for the same RDO object because only one owner is permitted to create authoritative changes.

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

### 12.4 Selected peers and record sets

Sections 12.4-12.6 apply to implementations of the optional replication profile. The initial use case is one person using Windows, Linux and Raspberry Pi OS computers on different networks. Operating systems and user-interface choices do not change the protocol rules.

An application MUST support an explicit replication selection for each recipient peer. A selection need not include the whole graph. An active selection MUST track subsequent owner-issued revisions of its selected records and retain pending delivery state for each recipient until replication is confirmed, explicitly cancelled, or reported as failed or expired.

A user interface MAY select a visible node or a set of nodes. Its application mapping MUST state which independent Object, Property, Relationship and Class records are included. Selecting a node MUST NOT silently authorize copying all records reachable from it. References to unselected objects MAY remain unresolved locally and MUST NOT be treated as deleted objects.

The selection MUST identify whether document bodies, attachments and additional type definitions are included. An application MUST NOT report a selected node as fully available offline until its required display/read content is retained locally. Each included record MUST preserve its own RDO ID, owner and Object Version; grouping records for delivery does not make Properties or Relationships embedded authoritative fields.

Selecting a recipient or possessing a replica MUST NOT grant authority to republish records beyond their permitted recipients. Removing a record from a replication selection stops future distribution under that selection; it is not an owner-issued deletion and does not prove previously delivered copies have been erased.

### 12.5 Durable local reading

A recipient MUST persist accepted records selected for offline availability so that they remain readable after an application restart while the authoring twin, propagation node and network are unavailable. It MUST retain them until an explicit local removal or retention rule applies. Propagation-message expiry MUST NOT by itself remove an accepted local replica.

Local reads MUST expose the retained Object Version and available freshness information. They MUST NOT require contacting the owner merely to reopen already validated content, and MUST NOT present an offline copy as proof of the owner's current state. Missing records, excluded content and unavailable attachments MUST remain distinguishable from known absence.

A replica store and an LXMF propagation spool have separate lifetimes. A device MAY host both, but accepting a delivery into the spool does not make that knowledge a readable local replica.

### 12.6 Ownership across one user's devices

Device peers belonging to one person remain separate protocol participants. They MUST use distinct recipient identities for per-device delivery and completion tracking. Normal replication MUST NOT copy the owner's private Twin Identity keys, transfer ownership or cause a replica to announce itself as the owner's canonical Object Destination.

Each twin MAY create and modify objects it owns. A proposed edit to a foreign object remains subject to Sections 10 and 11.2, including when both twins belong to the same user. Independent authoritative editing of the same record on several devices is outside this profile.

## 13. Federation

### 13.1 Destination-based federation

Federation occurs by referencing RDO IDs and communicating with their Reticulum Destinations.

An optional replica-delivery carrier may address a peer inbox as specified in Section 16.2. That delivery address does not replace an object's canonical Destination or the resolution and owner-operation rules.

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

A conforming Draft 0.3 implementation MUST NOT require a global object-owner directory for normal object addressing.

The Reticulum Destination is itself the federation address.

### 13.3 Remote reads

A remote query MAY be served by the owner or, where application policy permits, by a replica.

A replica response SHOULD identify that it is a replica response and SHOULD expose its known Object Version or freshness information.

An operation that can change authoritative state MUST be submitted to the Object Destination controlled by the owner.

### 13.4 Cross-owner operations

An application workflow MAY require changes to objects with different owners. Draft 0.3 treats each owner operation independently.

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

The optional LXMF replication profile MUST NOT make RPP depend on LXMF or replace RPP's Reticulum Resource-based binary transfer. A reference to a file or package is not a locally available copy of its bytes. Applications promising offline access to an attachment MUST arrange its retained delivery and verify availability independently of receiving the reference.

## 16. Reticulum Transport

### 16.1 Object addressing and operations

Every independently addressable Object, Property, Relationship and published Class MUST have a unique Reticulum SINGLE Destination. The Draft 0.2 shared-Twin-Identity derivation and signed-token reconstruction are defined in Section 6.

Small operations MAY use RNS packets/Links and MUST authenticate requesters where required. The owner MUST separately authorize every state-changing operation. Large byte transfers MAY use RNS Resources, without an extra fragmentation protocol.

A separate compact wire profile MUST define exact request/response encoding, signed application-data layout, limits and version negotiation. This semantic specification alone does not establish interoperable wire implementations.

### 16.2 Optional replica delivery over LXMF

The selected carrier for the initial replication application is LXMF. Implementations of this profile MUST reuse LXMF direct/propagated delivery and existing LXMF propagation nodes. This profile defines no new propagation network or relay protocol. Core RDO remains usable without LXMF.

A replication message is addressed to the receiving device's LXMF delivery Destination. The payload contains records identified by their existing RDO Destination hashes. An LXMF delivery Destination, message ID, transfer identifier or replication receipt MUST NOT replace a record's RDO ID.

The sender MUST maintain a separate delivery outcome for each selected peer. It MAY use direct delivery when a peer is reachable and propagated delivery when the peer is offline. An application relying on delayed delivery MUST configure an available propagation node and support retrieval of waiting messages. Sharing a human owner or a propagation node does not imply delivery to every device.

Authoritative mutations continue to use the Object Destination and owner authorization rules in Sections 6 and 10. Receiving a replication envelope only installs non-owning copies; it MUST NOT be treated as an instruction to mutate an authoritative object. Peer lists and replica indexes do not replace Reticulum destination-native resolution.

### 16.3 Snapshot contents and authority

Initial replication uses complete revisions of the selected records. A snapshot MUST carry the state needed to read those records offline, rather than only an object ID or a change notification requiring a later fetch from the owner. Implementations MAY batch independent records while preserving their separate identities, versions and owners. Delta encoding is not required by this profile.

The replication binding MUST represent, for each record:

- its canonical RDO ID, owner and Object Version;
- its type and complete record state, including the structural source/value or source/destination references for Property and Relationship records;
- sufficient verifiable origin and identity-binding information to validate that state without an online owner, either in the delivery or already retained by the recipient.

A recipient MUST check an incoming transfer against its configured sharing policy and validate the original owner's authority for every accepted record. A forwarding peer's signature alone is insufficient when that peer does not own the record. An authenticated LXMF origin may supply owner evidence only when the binding verifies that origin as the record's owner. A forwarding implementation MUST preserve the evidence required by that binding; a relay MUST NOT acquire ownership by carrying the bytes.

A snapshot MUST NOT convert independently owned Properties or Relationships into authoritative fields of another record. Packaging several records together does not establish an atomic snapshot across owners or a complete graph. If a deletion is distributed, it MUST be an explicit owner-authorized record; omission from a snapshot or selection is not deletion.

### 16.4 Persistent delivery and receipts

The sender MUST retain pending replication intent per recipient across application restarts and retain enough source data to retry safely. Delivery expiry, unavailable storage or exhausted retries MUST remain visible as incomplete replication. Handing data to a propagation node MUST NOT alone mark a peer's replica as complete.

An implementation MUST distinguish these milestones; the names below describe semantics and do not assign wire status codes:

| Milestone | Meaning |
| --- | --- |
| Queued locally | The sender has persisted the intended delivery. |
| Stored by propagation node | An intermediary has accepted custody of the message. |
| Received by destination | The recipient has obtained the message. |
| Stored as replica | The recipient has validated and durably committed the identified record revisions. |

A recipient MUST issue a successful replication receipt only after durable local acceptance. The receipt MUST identify its recipient and accepted record IDs and versions, with a correlation to the transfer being acknowledged. The sender MUST authenticate the receipt's origin. Transport receipts and local daemon custody acknowledgements MUST NOT be interpreted as successful graph application.

Receipt delivery MUST support LXMF propagation while the original sender is offline. Pending receipt intent, or durable acceptance data sufficient to reissue the same acknowledgement, MUST survive recipient restart. Local reading of accepted records MUST NOT wait for delivery of that receipt back to the sender. Receipt messages MUST NOT themselves require replication receipts.

A replication receipt MUST remain distinct from the `APPLIED` Operation Result in Section 10. Receiving state is not a new owner-authorized mutation. Per-record acceptance MUST NOT imply atomic acceptance of every record in a batch or synchronized versions across different owners.

The receiver MUST retain enough acceptance state to recognize retries after restart and reissue a receipt for records already accepted. Duplicate delivery of the same record revision MUST NOT create duplicate objects or relationships. Reusing an RDO ID and version for different authenticated state MUST be rejected as inconsistent.

Object Versions remain owner-issued markers. Arrival order, timestamps and lexical or numeric comparison of an unspecified marker MUST NOT be used to silently replace a known newer record. The wire binding MUST define how successor versions are established. If their order cannot be established, the recipient MUST preserve its accepted state and report or retain the uncertain revision separately.

### 16.5 Offline recipients and storage limits

A recipient MAY retrieve and accept a propagated snapshot after its authoring computer has gone offline. This requires that another available node already retained the payload and that the recipient retrieves it before the intermediary discards it. If no intermediary is available, the sender MUST retain its queue until contact is possible or delivery is explicitly ended as incomplete.

Propagation custody is bounded by node retention, capacity and delivery policies. Applications MUST NOT rely on a propagation spool as permanent knowledge storage. They MUST account for expiry and payload limits, preserve delivery intent or report failure, and reconcile missing revisions when communication resumes. A copied reference or an update notification cannot substitute for retained content when the owner is offline.

Large attachments and transfer fragments remain subject to the chosen content-transfer mechanism and its limits. This profile does not introduce a second fragmentation protocol or assign RDO identities to transport chunks.

### 16.6 Wire binding still to be specified

LXMF provides custom application fields, including `FIELD_CUSTOM_TYPE` and `FIELD_CUSTOM_DATA`. They are the intended extension mechanism for this application. The name `rdo.replication.v1` is illustrative and is not yet an assigned or interoperable wire identifier.

Before claiming wire interoperability, a binding MUST specify field assignments and discriminator negotiation, record and receipt encoding, signature coverage and identity-binding verification, version succession, deletion representation, batching and payload limits, and error handling. It MUST include verified vectors and the delayed-delivery acceptance scenario in RDO-0001. These details have not been fixed by the architectural decision in this draft.

Implementation note (non-normative): the initial Rust application can reuse the existing LXMF-rs application-payload and propagation APIs. Its persistence and custody options need to be enabled and verified for the claimed guarantees. SDK support alone does not establish RDO replication conformance.

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

An extension MUST NOT replace Destination hashes with another canonical RDO ID, reintroduce embedded authoritative Property/Relationship state, or grant nonowners write authority while claiming Draft 0.3 conformance. Other address profiles and consensus models require explicit future specifications.

## 21. Conformance

A core Draft 0.3 implementation MUST support the four M3 classes and their inheritance, give each addressable instance one Reticulum SINGLE Destination hash, maintain one owning twin, and model Property and Relationship as separately owned/addressed Objects.

It MUST support owner-authorized operations with version preconditions and retained duplicate-request protection, independent M2 ontologies, and structural checks without epistemic truth decisions. It MUST NOT require RPP or assign RDO identities to every binary chunk.

Shared-Identity binding implementations MUST demonstrate hash-only discovery with signed token retrieval, correct outbound Destination reconstruction and authorized requests interoperating with Python Reticulum before claiming wire conformance.

Core conformance MUST NOT require LXMF. An implementation claiming the optional replication profile MUST also satisfy Sections 12.4-12.6 and 16.2-16.6, including selected-peer scope, durable offline reading, preserved owner authority, restart-safe delivery state and application receipts. A semantic-profile implementation MUST NOT claim an interoperable wire protocol until the binding and tests required by Section 16.6 exist.

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
13. Optional replication sends selected owner-issued records to distinct device peers; receiving them grants no write authority.
14. A peer's LXMF inbox and an RDO Object Destination have different roles and are not interchangeable identifiers.
15. Propagation custody, destination delivery and durable local replica acceptance are distinct events.
16. Core RDO and RPP remain usable without LXMF.

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
- Accepted selective replication decision: [RDO-0001](proposals/RDO-0001-selective-lxmf-replication.md)
- LXMF message fields and propagation: https://github.com/markqvist/LXMF
- LXMF custom application field definitions: https://github.com/markqvist/LXMF/blob/master/LXMF/LXMF.py
- LXMF-rs application custody: https://github.com/FreeTAKTeam/LXMF-rs/blob/main/docs/migrations/propagation-application-kind.md
- LXMF-rs delivery choices: https://github.com/FreeTAKTeam/LXMF-rs/blob/main/docs/sdk/quickstart.md
- LXMF-rs durable custody and receipt semantics: https://github.com/FreeTAKTeam/LXMF-rs/blob/main/docs/sdk/README.md and https://github.com/FreeTAKTeam/LXMF-rs/blob/main/docs/sdk/delivery-states.md

