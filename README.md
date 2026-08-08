# Reticulum Distributed Objects (RDO)

Reticulum Distributed Objects defines a generic protocol model for representing, exchanging, resolving, and synchronizing distributed objects, assertions, relations, events, provenance, and projections over systems using the Reticulum Network Stack. It is independent of any particular application, database, or simulation engine.

Status: Draft, version 0.1.

Read the [normative specification](SPECIFICATION.md). This repository contains the protocol specification, not a particular implementation or runtime library.

Protocol versions use semantic versioning where practical. Git tags identify published specification versions; tagged specifications are immutable. Incompatible wire-level changes require a major version, compatible additions require a minor version, and editorial corrections may use a patch version. Implementations should declare the RDO versions they support.

Proposed changes are documented through the process described in [proposals/](proposals/).

RDO and [RPP](https://github.com/FreeTAKTeam/rpp-spec) are sibling protocols. RDO MAY reference RPP for large external resources, but neither protocol requires the other.

