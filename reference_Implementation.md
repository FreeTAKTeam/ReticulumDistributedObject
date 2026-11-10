# RDO Reference Implementation

**Status:** Reference application design; implementation and wire interoperability remain to be demonstrated.

**Decision date:** 2026-11-10.

This document records the application direction agreed during discovery: a Rust knowledge client with TheBrain/K-plex-style navigation, explicit peer selection inspired by REM, and selective RDO replication over LXMF. It describes UI and component responsibilities. The [RDO specification](SPECIFICATION.md) governs protocol semantics; [RDO-0001](proposals/RDO-0001-selective-lxmf-replication.md) records the accepted replication decision.

## 1. Purpose and agreed direction

The first use case is one person accessing selected knowledge on three computers, for example Windows, Linux and Raspberry Pi OS machines on different networks. A received node remains readable from local storage while its authoring computer is offline. Updates can reach a device that was unavailable when they were sent, including after the sender disconnects, provided an available intermediary retained the delivery.

The application also provides a place to exercise RDO, support R3AKT knowledge management and explore an alternative Dafrion client. These uses share the navigation and distribution approach; their domain types and permitted actions can differ.

| Area | Recorded direction |
| --- | --- |
| Navigation | Focus on one selected node and its immediate relationships, with smooth recentering. |
| Implementation language | Rust. Rune is a UI candidate; selection is still open. |
| Sharing | Choose known device peers and the knowledge each should receive. |
| Offline access | Persist accepted records and included content locally. |
| Delivery | Reuse LXMF direct delivery and propagation; define the RDO application payload and acceptance rules above it. |
| Authority | Preserve independent record identities and the existing single-owner write rule. |

## 2. UI style

### 2.1 Focused graph navigation

The interaction takes inspiration from [TheBrain's navigation](https://www.thebrain.com/blog/navigating-beyond-the-plex) and [K-plex](https://github.com/zsviczian/kplex): the selected node occupies the center, and choosing a neighbor makes that node the new focus.

The reference design includes:

- A structured immediate neighborhood, with stable regions for different relationship roles. Parents above and children below provide the initial hierarchical convention; side regions show associated or other typed relationships.
- Animated recentering that helps the user follow the selected relationship.
- Back/forward navigation, history, pins and search.
- A content and properties panel alongside the graph, with inspection of relationship type, source, owner and version.
- Visible availability information for local records, pending transfers and references whose content has not been received.

A domain profile maps relationship meanings to regions, labels and visual styles. DAF or R3AKT relationship types retain their exact semantics beneath that presentation. The spatial convention does not replace the stored relationship type. A filtered or partially replicated neighborhood must not suggest that the displayed graph is complete.

### 2.2 A visible node can represent several RDO records

The UI may present a knowledge item, its name, description and selected relationships as one navigable node. The domain adapter resolves that presentation into separate Object, Property, Relationship and, where applicable, Class records. Each retains its own canonical RDO ID, owner and version, as required by the [RDO data model](SPECIFICATION.md).

For example, sharing a note can include the note Object, its name and body Properties, and one Relationship to another item. That relationship can remain visible as a reference even if its target was excluded from sharing. Neither the presentation nor a grouped transfer turns these records into embedded authoritative fields.

Reading uses the local store. A node does not require a connection to its owner merely to display an accepted revision. An attachment is marked available offline only when the required bytes are stored locally; receiving its reference is insufficient.

### 2.3 Existing Dafrion work and the Rust UI choice

Dafrion's existing [impact visualization](https://github.com/FreeTAKTeam/dafrion-desktop/blob/5942491b8733bb0a5d2cb34a99852cf679c1302f/packages/desktop/apps/electron/src/renderer/components/product/dafrion/DafrionImpactVisualization.tsx) supplies a practical reference for focused neighborhoods, selecting a neighbor as the new focus, relationship inspection and animated layout changes. Its [Core client](https://github.com/FreeTAKTeam/dafrion-desktop/blob/5942491b8733bb0a5d2cb34a99852cf679c1302f/packages/desktop/apps/electron/src/renderer/components/product/dafrion/core-client.ts) is an integration reference for domain queries and governed operations.

This existing work informs behavior and service integration. A Rust client still needs its own UI implementation; direct reuse of the React/TypeScript renderer has not been selected. A future Dafrion adapter must preserve the applicable Core validation and operation flow.

[Rune](https://github.com/anomalyco/rune) is a candidate for the Rust UI. Before selecting it, a small prototype needs to demonstrate graph rendering, text editing, hit testing and acceptable performance on the intended Windows, Linux and Raspberry Pi OS devices. Navigation state, layout rules and selection logic should remain ordinary Rust modules independent of the renderer. Exact component and crate boundaries remain open.

## 3. REM approach to peers

### 3.1 Explicitly saved peers

REM already supports discovery or import of peers, explicit saving, labels, filtering and connection controls. Discovered peers are not automatically saved, and the saved list persists locally. Its save implementation prefers a discovered peer's LXMF delivery destination. See the inspected [REM mobile README](https://github.com/FreeTAKTeam/reticulum_mobile_emergency_management/blob/82d3d527b0dde9f9859c0a5302f6e513808b3cc5/apps/mobile/README.md) and [peer actions](https://github.com/FreeTAKTeam/reticulum_mobile_emergency_management/blob/82d3d527b0dde9f9859c0a5302f6e513808b3cc5/apps/mobile/src/stores/nodeActionsController.ts).

The reference application adopts that explicit peer-management approach. A user can recognize devices by labels such as “Office Windows”, “Linux laptop” and “Home Pi”, while inspecting their addresses and availability when needed. Saving a peer does not by itself share knowledge or authorize incoming records. REM's team-specific policies are not requirements of this application.

### 3.2 Select knowledge, then recipients

The intended workflow is:

1. Select a node or a set of nodes and choose **Share**.
2. Choose saved device peers that should receive them.
3. Inspect the records and content included by the domain profile, including any referenced neighbors or attachments left out.
4. Persist that selection per peer and enqueue the selected record revisions.
5. Keep those selections updated as new owner-issued revisions become available, showing delivery and local availability separately.

Distribution is selective. Sharing one node does not share the entire graph or recursively disclose every relationship target. The adapter declares which display properties, relationship records and type information are included; the replication module enforces the resulting record set. Automatic dependency expansion beyond this explicit mapping remains an open design choice.

Removing a record from a sharing selection stops future distribution under that selection. It is not an owner-issued deletion or proof that an earlier recipient erased its copy. These rules follow [RDO Sections 12.3–12.6](SPECIFICATION.md#123-partial-replication).

### 3.3 Keep three roles distinct

| Role | Meaning |
| --- | --- |
| Device peer | An application installation with its own recipient identity and LXMF inbox. It can retain readable RDO replicas and issue application receipts. |
| Propagation node | An LXMF service that holds encrypted messages for later retrieval. Selecting it chooses delivery infrastructure; it does not make it a reader of the knowledge. |
| RDO object | An independently identified and owned record. Its canonical ID remains its RDO Destination hash, regardless of which peer stores a replica. |

Several device peers may belong to the same person. Replication does not copy private owner keys or make all those devices co-owners of existing records. The peer inbox is a delivery address; owner operations still use the canonical RDO object destination.

A Pi can serve as both a reader and a propagation node. These roles use different stores: a readable application database and an encrypted delivery spool. If the Pi should read the selected knowledge, it receives its own addressed copy in addition to carrying messages for another device. The Pi is an example deployment, not a required central server.

### 3.4 Delivery while another device is unavailable

Use [LXMF](https://github.com/markqvist/LXMF) direct delivery when available and existing propagation facilities for deferred delivery. If no suitable propagation node is reachable, the sender retains its durable pending deliveries and retries. Sending a notification that requires an immediate fetch from the owner does not meet the offline-reading use case; selected complete record revisions are the initial transfer unit.

Track the following milestones per recipient:

| Milestone | What it establishes |
| --- | --- |
| Queued locally | The sender has persisted the delivery intent. |
| Stored by a propagation node | An intermediary has accepted custody, subject to its retention and capacity limits. |
| Received by the destination | The recipient's messaging endpoint has received the delivery. |
| Accepted as a replica | The application has validated and durably stored the identified record revisions. |

The sender confirms the final milestone only after receiving an authenticated application receipt. An LXMF delivery signal or daemon custody acknowledgement does not establish durable acceptance into the knowledge store. Receipt delivery also supports propagation while the original sender is offline; pending receipt state survives restart, and receipt messages do not require another replication receipt. Local reading does not wait for the original sender to receive that acknowledgement.

These are the [RDO transport and receipt rules](SPECIFICATION.md#16-reticulum-transport). Propagation provides deferred delivery within storage limits; it does not guarantee permanent backup or delivery after every possible outage.

## 4. Division of responsibilities

The following assignments are logical boundaries for the reference design. They do not prescribe a crate layout, database product or deployment topology.

| Part | Owns | Boundary |
| --- | --- | --- |
| Desktop UI | Focus, graph layout and rendering, history, search, content inspection, user commands, recipient selection and availability display. | Screen position and user selection do not confer write authority or define stored relationship semantics. |
| Domain profile / application adapter | Map domain records to visible nodes, relationship regions, editors and permitted actions; resolve sharing selections into explicit record sets. | Preserve exact domain types. A Dafrion adapter uses its applicable Core services; a basic knowledge or R3AKT client does not acquire a mandatory DAF dependency. |
| RDO semantic core | Canonical IDs and identity bindings; first-class records; ownership, versions, structural validation and owner-authorized operations. | Does not define the UI, decide factual truth or implement a separate propagation service. |
| Optional RDO replication module | Per-peer selections; packaging revisions; checking sharing policy and owner evidence; revision acceptance and deduplication; pending deliveries, receipts and restart recovery. | Uses LXMF for delivery. Preserves original ownership when forwarding and distinguishes transport success from replica acceptance. |
| Local store and indexes | Persist owned records, non-owning replicas, included content, peer selections, pending deliveries and acceptance state; support local queries. | A stored replica remains non-owning. Durable commit precedes a successful application receipt. Storage technology remains open. |
| LXMF-rs | Message creation, recipient-addressed delivery, direct/propagated modes, propagation retrieval and transport/custody signals. | Does not interpret graph semantics or determine that an RDO revision has been accepted by the application. |
| Reticulum | Cryptographic identities, destinations, path discovery, links and Resource transport. | Network authentication alone does not authorize an RDO mutation or create a persistent readable replica. |
| RPP, when needed | Package/file semantics and binary content transfer over Reticulum Resources. | Optional for simple knowledge records. Neither RDO core nor RPP requires LXMF; bulk transfer need not turn every chunk into an RDO object. |

The transport assignments use existing [LXMF-rs delivery options](https://github.com/FreeTAKTeam/LXMF-rs/blob/main/docs/sdk/quickstart.md), [application custody classification](https://github.com/FreeTAKTeam/LXMF-rs/blob/main/docs/migrations/propagation-application-kind.md) and [delivery-state semantics](https://github.com/FreeTAKTeam/LXMF-rs/blob/main/docs/sdk/delivery-states.md). File semantics follow the [RPP specification](https://github.com/FreeTAKTeam/rpp-spec/blob/main/SPECIFICATION.md). Their availability is a starting point for integration, not evidence that this RDO client already exists.

### 4.1 Reading, writing and sharing

- **Read:** the UI queries the local store through the domain adapter. Excluded or unavailable neighbors remain references; they do not prevent reading an accepted local node.
- **Write:** the UI submits an intention through the adapter and the relevant owner operation path. A device can commit changes to records it owns. Edits to another twin's records remain requests until that owner accepts them; being the same human user does not bypass this rule.
- **Share:** the adapter resolves a visible selection into record IDs and content. The replication module validates the selection, persists deliveries and uses LXMF. The recipient validates original owner evidence, accepts revisions durably and returns its application receipt.

Duplicate delivery must not duplicate nodes or relationships. Revision handling must not treat arrival time or an assumed numeric maximum as an ordering rule. An older, conflicting or unorderable revision must not silently replace accepted state. Exact version-succession and wire rules remain pending in the specification.

## 5. First implementation slice

The first slice should demonstrate the [three-computer acceptance scenario in RDO-0001](proposals/RDO-0001-selective-lxmf-replication.md#three-computer-acceptance-scenario), with a small focused graph and an explicit peer list:

1. Create a node with display Properties and a selected Relationship; share it with chosen devices while leaving an unrelated node unshared.
2. Recover selected peers and pending deliveries after restarting the sender.
3. Deliver through a propagation node while the recipient is offline; disconnect the authoring device before the recipient retrieves the message.
4. Read the accepted records locally after disconnecting and restarting the recipient, showing excluded content accurately.
5. Handle duplicate delivery without duplicate records and preserve owner authority during attempted edits.
6. Return the durable acceptance receipt through propagation while the original sender is offline, then complete its pending delivery when it reconnects.

This is a proposed implementation and verification sequence. This document does not report completed application tests.

## 6. Choices still open

- Rune or another Rust UI choice, following tests on the target devices.
- Exact relationship-region mappings, layout algorithms and editing interactions for each domain.
- Peer discovery and onboarding details beyond the explicit saved-peer approach.
- Database, indexing, module/crate boundaries and embedded-runtime versus service integration.
- Dependency expansion, attachment inclusion and transfer coordination.
- Replication message encoding, wire identifier, signature/binding representation, version succession, batching and size limits.
- The extent of Dafrion and REM code reuse, as distinct from reusing their interaction and integration patterns.

These choices can be resolved through the reference implementation without changing the agreed division between application behavior, RDO semantics, LXMF delivery and Reticulum transport.
