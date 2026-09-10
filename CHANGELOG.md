# Changelog

## 0.1 - Draft

Initial publication of the protocol specification.

Draft refinements:

- made the Reticulum `SINGLE` Destination hash the sole canonical RDO identifier;
- fixed the Draft 0.1 logical Object Destination name to `rdo.object` with a distinct Object Identity per RDO object;
- defined exactly one owning twin per RDO object;
- restricted authoritative changes to the owning twin;
- required federated twins to resolve and address objects through Reticulum and submit operations to the owner;
- removed the need for an IRI-to-owner or object-to-host directory as the primary federation mechanism;
- clarified that Dafrion/RDF IRIs are semantic or external identifiers, not a second RDO identity;
- explicitly deferred multi-writer state, distributed consensus, CRDT ownership, automatic ownership transfer, and cross-owner atomic transactions;
- added stale-request and duplicate-request protections for remote state-changing operations;
- clarified disconnected behavior: foreign state may be cached, but foreign authoritative writes wait for the owner.
