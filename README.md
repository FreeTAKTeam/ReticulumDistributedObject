# Reticulum Distributed Objects (RDO)

Reticulum Distributed Objects defines a protocol model for addressable distributed objects over the Reticulum Network Stack.

Draft 0.1 uses a deliberately simple ownership rule:

- every RDO object has exactly one canonical identifier: its Reticulum `SINGLE` Destination hash;
- every RDO object has exactly one owning twin;
- only the owning twin may commit authoritative changes;
- federated twins address the object's Reticulum Destination and submit operation requests to the owner;
- application identifiers such as Dafrion/RDF IRIs may be preserved as semantic or external identifiers, but they are not a second RDO identity and are not used as the primary routing mechanism.

This keeps location and cryptographic identity inside Reticulum and avoids requiring a separate object-to-owner or IRI-to-host directory.

Status: Draft, version 0.1.

Read the [normative specification](SPECIFICATION.md). This repository contains the protocol specification, not a particular implementation or runtime library.

Draft 0.1 intentionally excludes multi-writer authoritative state, distributed consensus, automatic ownership transfer, CRDT-based ownership, and atomic transactions spanning several owning twins. These may be considered after the single-owner model has been implemented and tested.

RDO and [RPP](https://github.com/FreeTAKTeam/rpp-spec) are sibling protocols. RDO MAY reference RPP for large external resources, but neither protocol requires the other.

Proposed changes are documented through the process described in [proposals/](proposals/).
