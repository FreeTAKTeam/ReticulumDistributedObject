# Reticulum Distributed Objects (RDO)

**Status: Draft 0.2 — distributed meta-metamodel (M3).**

RDO defines a small distributed ontology over the Reticulum Network Stack. Its foundational classes are **Object**, **Class**, **Property** and **Relationship**. Every independently published instance of these classes has its own Reticulum SINGLE Destination, one canonical identifier (the Destination hash), and one owning twin.

- Properties and relationships are full, separately addressable RDO objects, never embedded authoritative fields or edges.
- A twin's Reticulum Identity can own multiple object Destinations. The Draft 0.2 shared-Identity address binding defines signed derivation metadata, pending Python/Rust interoperability testing.
- A Relationship has explicit source, destination and semantic type, even when those endpoint objects belong to other twins.
- Several twins can independently describe the same external fact; truth and evidence assessment belong to an optional **epistemic layer**, not RDO.
- A foreign twin can reference, cache and request operations but cannot commit changes to objects it does not own.
- RDO provides the M3 type/operation basis for M2 ontologies such as **RPP**, **DAF** and **R3AKT**.
- Bulk content transfer uses Reticulum Resources. RDO does not reinvent file transfer.

Read the [normative specification](SPECIFICATION.md) and the [Turtle M3 vocabulary](ontology/rdo-core.ttl).

[RPP](https://github.com/FreeTAKTeam/rpp-spec) is an M2 package/file ontology defined on top of RDO, with a Reticulum Resource-based binary transfer mechanism. RDO itself does not depend on RPP.

This repository defines the draft specification and vocabulary, not a production implementation. A compact operation wire profile and shared-Identity address interoperability tests remain open.

See [proposals/](proposals/) for the proposal process.
