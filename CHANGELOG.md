# Changelog

## 0.3 - Draft - 2026-11-10

- Accepted [RDO-0001](proposals/RDO-0001-selective-lxmf-replication.md): selective replication for one user's computers on different networks.
- Added per-peer record selections and persistent local copies that remain readable while the authoring twin is offline.
- Selected existing LXMF direct/propagated delivery and propagation nodes as the initial application's carrier; added no new propagation protocol.
- Kept device LXMF delivery addresses separate from canonical RDO Object Destinations and preserved each record's identity, ownership and version.
- Required complete selected record revisions, per-recipient delivery state, safe retries and application receipts after durable acceptance.
- Distinguished local queues, propagation custody, recipient delivery and local replica acceptance; retained incomplete-delivery reporting and explicit attachment availability.
- Preserved single-owner authoritative writes, first-class Property/Relationship records, the Draft 0.2 address binding and LXMF-independent core RDO/RPP.
- Corrected stale operative Draft 0.1 references and the obsolete per-object private-Identity migration wording.

The M3 vocabulary is unchanged. Exact replication encoding, discriminator, authentication binding, version succession and interoperability vectors remain pending; `rdo.replication.v1` is only an illustrative identifier.

## 0.2 - Draft

- Recast RDO as a distributed M3 over Reticulum; added ontology/rdo-core.ttl.
- Defined rdo:Object, rdo:Class, rdo:Property and rdo:Relationship, with the latter three inheriting Object.
- Made each Property and Relationship instance a separate owned/addressable object instead of an embedded field or edge.
- Distinguished one owning Twin Identity from multiple Reticulum Object Destinations.
- Introduced a shared-Identity Destination naming profile with signed aspect-token recovery; Python/Rust interoperability testing is still required.
- Kept one Destination hash as the sole canonical RDO ID and retained single-owner mutation.
- Separated structural correctness from epistemic truth assessment.
- Reclassified RPP as an optional M2 ontology atop RDO; bulk bytes remain on RNS Resources.
- Preserved nonowner request routing, duplicate handling, stale-version protection and offline behavior.

Draft 0.2 changes the proposed Reticulum binding; it does not claim compatibility with the untagged Draft 0.1 profile.

## 0.1 - Draft

Initial published draft. Later refined single-owner rules, Reticulum destination-hash identity, remote requests and disconnected behavior. The original draft used a distinct Reticulum Identity per object and fixed destination name rdo.object.
