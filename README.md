# Reticulum Distributed Objects (RDO)

**Status: Draft 0.3.** Distributed meta-metamodel (M3) with optional selective replication over LXMF.

RDO defines a small distributed ontology over the Reticulum Network Stack. Its foundational classes are **Object**, **Class**, **Property** and **Relationship**. Every independently published instance of these classes has its own Reticulum SINGLE Destination, one canonical identifier (the Destination hash), and one owning twin.

- Properties and relationships are full, separately addressable RDO objects, never embedded authoritative fields or edges.
- A twin's Reticulum Identity can own multiple object Destinations. The Draft 0.2 shared-Identity address binding defines signed derivation metadata, pending Python/Rust interoperability testing.
- A Relationship has explicit source, destination and semantic type, even when those endpoint objects belong to other twins.
- Several twins can independently describe the same external fact; truth and evidence assessment belong to an optional **epistemic layer**, not RDO.
- A foreign twin can reference, retain replicas and request operations but cannot commit changes to objects it does not own.
- RDO provides the M3 type/operation basis for M2 ontologies such as **RPP**, **DAF** and **R3AKT**.
- Bulk content transfer uses Reticulum Resources. RDO does not reinvent file transfer.

Read the [normative specification](SPECIFICATION.md) and the [Turtle M3 vocabulary](ontology/rdo-core.ttl).

[RPP](https://github.com/FreeTAKTeam/rpp-spec) is an M2 package/file ontology defined on top of RDO, with a Reticulum Resource-based binary transfer mechanism. RDO itself does not depend on RPP.

## Selective replication

The initial use case is one person accessing selected knowledge on Windows, Linux and Raspberry Pi OS computers on different networks. Each device has its own recipient identity and persistent local replica store. The user selects which records go to which peers; distribution need not include the whole graph.

The optional profile reuses LXMF direct delivery and propagation nodes for offline recipients. Messages carry complete revisions of selected records so that a recipient can read them after the authoring computer goes offline. Object, Property and Relationship records retain their separate RDO identities, owners and versions. A peer's LXMF inbox is a delivery address, not an object's canonical RDO ID.

Successful replication requires an application receipt after durable local acceptance. A propagation node holding a message is an earlier delivery milestone. Pending delivery and accepted local records survive application restarts. Replication preserves the existing single-owner write rule, including across devices belonging to the same user.

Core RDO and RPP remain usable without LXMF. The bootstrap ontology is unchanged from Draft 0.2. Read [RDO-0001](proposals/RDO-0001-selective-lxmf-replication.md) for the accepted decision and the three-computer acceptance scenario.

This repository defines the draft specification and vocabulary. Exact operation and replication encodings, the replication wire identifier, signature and version-succession rules, and shared-Identity interoperability tests remain open. Draft 0.3 records the semantic contract; it does not claim a tested replication implementation.

See [proposals/](proposals/) for the proposal process.
