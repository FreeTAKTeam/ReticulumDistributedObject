# Changelog

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
