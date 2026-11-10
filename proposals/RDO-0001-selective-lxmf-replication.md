# RDO-0001: Selective replication over LXMF

Status: Accepted architectural decision\
Decision date: 2026-11-10\
Specification revision: 0.3-draft\
Wire binding: Pending

## Problem

One person needs to access the same selected knowledge on three computers, for example Windows, Linux and Raspberry Pi OS machines on different networks. A received node must remain readable locally while its authoring computer is offline. A recipient that is offline when an update is published must be able to obtain that update later, including after the authoring computer disconnects.

Distribution is selective. The user chooses peers to receive particular nodes or sets of nodes, following the peer-selection interaction used in REM. Selecting a peer does not expose the entire graph.

The application may use TheBrain/K-plex-style navigation and a Rust implementation. Rune remains a UI candidate. These application choices do not add dependencies or requirements to RDO. R3AKT knowledge management and an alternative Dafrion client are later consumers of the same replication capability.

## Decision

Use existing LXMF propagation for delayed delivery. Add an optional RDO replication profile that carries owner-issued record snapshots inside LXMF application messages. The normative requirements are in [SPECIFICATION.md](../SPECIFICATION.md), Sections 12.4-12.6 and 16.2-16.6.

Each device is a separate recipient peer with persistent local storage. The application keeps a selection and delivery state for each peer. Selected records and required read content are copied locally; an object ID or update notification alone does not satisfy offline reading.

Replicated Object, Property, Relationship and Class records retain their independent RDO IDs, owners and versions. A grouped transfer is a delivery convenience. It does not turn first-class records into authoritative embedded fields or grant the receiving device ownership.

Initial transfers carry complete selected record revisions. Each recipient confirms durable acceptance at the application level. Delivery tracking distinguishes the sender's queue, propagation custody, receipt by the device and acceptance into its local replica store. Pending delivery survives restart, and repeated delivery does not create duplicate records.

The current single-owner rule continues to govern authoritative writes. This decision does not authorize independent offline editing of the same record on several machines. Proposed changes to foreign objects continue through the owner-operation model.

## Responsibilities

| Part | Responsibility |
| --- | --- |
| Reticulum | Destination-native identity resolution and network transport. |
| LXMF | Recipient-addressed messages, direct delivery and propagation for offline recipients. |
| RDO replication module | Selection enforcement, owner/version validation, persistent local records, retry tracking and replica receipts. |
| Application | Peer selection, node-to-record mapping, content availability and local presentation. |

Core RDO remains usable without LXMF. RPP remains an M2 ontology over RDO with its existing Reticulum Resource-based binary transfer and no mandatory LXMF dependency.

## Three-computer acceptance scenario

This is an acceptance scenario for a future implementation, not a report of an executed test.

1. Windows creates a knowledge node with its display properties and a selected relationship. It selects Linux and Raspberry Pi as recipient peers. Another unrelated node remains unshared.
2. Linux is offline. Windows persists the intended deliveries and restarts. The recipient selections and pending deliveries are recovered.
3. Windows sends Linux's snapshot through an available LXMF propagation node on the Pi. The Pi retains the encrypted delivery; Windows then goes offline. If the Pi is also a reader, it receives its own recipient-addressed copy separately.
4. Linux comes online later and retrieves its delivery while Windows remains offline. It validates the original owner and object bindings, accepts the selected records into its local database and queues or propagates a replication receipt for Windows. Pending receipt intent survives restart, and local reading does not wait for Windows to receive it.
5. Linux disconnects and restarts. The node and included content remain readable locally, with retained versions and freshness information. Excluded neighbours or attachments remain visibly unavailable, and the unrelated node has not been copied.
6. The same revision is delivered again. Linux creates no duplicate node or relationship and can confirm its existing durable acceptance. An older or unorderable revision does not silently overwrite accepted state.
7. Linux sends its receipt through the Pi and disconnects. Windows reconnects while Linux is offline, retrieves and authenticates the receipt, and closes only the pending delivery for the acknowledged Linux records. The receipt does not generate another replication receipt.

The source and recipient need not be online together. Some intermediary must already hold the payload, and its retention must last long enough for retrieval. A failure or expiry before destination acceptance remains incomplete delivery.

## Compatibility and security effects

The M3 ontology and the shared-Twin-Identity Destination derivation remain unchanged from Draft 0.2. Core-only implementations do not need LXMF. Implementations opting into replication need the additional semantics and a future agreed wire binding.

A device's LXMF inbox is not a substitute for an object's canonical Reticulum SINGLE Destination. Replica deliveries do not change owner-operation addressing, and replicas do not announce themselves as the authoritative object endpoint. A peer list is not a global object-owner directory.

Each accepted record needs verifiable owner origin even when it arrived through another peer. Ordinary replication does not distribute private Twin Identity keys. Sharing controls cover recipients and selected content; possession of a record or a reference does not authorize broader disclosure.

Propagation storage is bounded. It is not the application's permanent replica store. Attachment references also do not prove offline availability of attachment bytes. Removing a sharing selection is distinct from deleting a record, and does not guarantee deletion of copies already delivered.

## Details left open

The architectural decision does not fix:

- The exact LXMF discriminator, field assignments, binary encoding or negotiation. `rdo.replication.v1` is an illustrative name.
- Signature coverage, canonical bytes and the representation of owner/identity-binding evidence.
- Object-version succession and deletion-record encoding or retention.
- Batch sizes, attachment policy, retry intervals, storage limits or relay selection defaults.
- Automatic expansion from a visible node to dependent records, beyond an explicit application mapping and sharing selection.

Multi-writer authority, automatic ownership transfer and cross-owner atomic commits remain outside the initial RDO model. RDO does not assess the factual truth of replicated statements.

## Existing implementation building blocks

These references identify reusable APIs and design constraints, not proof of a working RDO replication application:

- [LXMF propagation and message fields](https://github.com/markqvist/LXMF).
- [LXMF custom application field definitions](https://github.com/markqvist/LXMF/blob/master/LXMF/LXMF.py).
- [LXMF-rs application custody classification](https://github.com/FreeTAKTeam/LXMF-rs/blob/main/docs/migrations/propagation-application-kind.md).
- [LXMF-rs direct/propagated delivery and fallback options](https://github.com/FreeTAKTeam/LXMF-rs/blob/main/docs/sdk/quickstart.md).
- [LXMF-rs propagation API](https://github.com/FreeTAKTeam/LXMF-rs/blob/main/crates/libs/lxmf-sdk/src/backend/zmq_pipeline/propagation.rs).
- [LXMF-rs durable custody configuration](https://github.com/FreeTAKTeam/LXMF-rs/blob/main/docs/sdk/README.md) and [delivery-state semantics](https://github.com/FreeTAKTeam/LXMF-rs/blob/main/docs/sdk/delivery-states.md).
- [RPP LXMF independence](https://github.com/FreeTAKTeam/rpp-spec/blob/main/SPECIFICATION.md), Section 5.3.
