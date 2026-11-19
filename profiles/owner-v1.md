# Shared-Identity owner binding v1

**Candidate for RDO Draft 0.3.** An application given one 16-byte object Destination can discover its owner, authenticate a request and retain a verifiable revision for offline reading. This profile specializes §§6 and 10. The LXMF replication wire profile has its own qualification process.

## Derive and resolve a SINGLE Destination

The owner persists one Reticulum Identity and a unique random 16-byte token per object. Persist both before publication. The app name is `rdo` and the two aspects are `object` and the 32 lowercase ASCII hex digits of the token. Object type never enters this construction. Given the 64-byte RNS public key (X25519 public key followed by Ed25519 verification key), compute these leftmost byte slices:

```text
identity_hash = SHA256(public_key)[0:16]
name_hash     = SHA256(UTF8("rdo.object." + lowercase_hex(token)))[0:10]
destination   = SHA256(name_hash || identity_hash)[0:16]
announce_app_data = CBOR([1, token])    # exactly 19 bytes
```

Use the normal signed RNS announce or path response. In RNS the signature covers the concatenation of Destination, public key, name hash, 10-byte random/time field, optional flagged ratchet key, and the complete RNS app-data field on the wire, including any native stamp extension. The 19-byte RDO value above is the application payload exposed after RNS validates and processes its framing. Enforce the network’s admission policy before accepting the binding. The object type, inventory and current revision are private and absent here. Prefer a path response to a broadcast.

Starting with only the desired Destination, request its path. Validate the **full RNS announce** including Destination and signature before parsing app data. Require the exact array above, recompute both hashes from its signed token and public Identity, and compare with the advertised name hash and requested Destination. Establish the outbound SINGLE Link with that public Identity and those same aspects. A valid announce for another Destination is insufficient. Cache only a verified tuple. A path timeout is retryable. A hash-only API that discards signed app data cannot satisfy this profile.

## Revision bytes

All structures below use one RFC 8949 §4.2.1 deterministic CBOR item. Use shortest definite lengths, unsigned integers up to 2^64−1, byte strings, UTF-8 text, arrays, uint-key maps in increasing order, and null only where specified. Reject duplicate keys, trailing bytes, tags, indefinite lengths and unsupported versions. Limit nesting to 12, whole revision to 48 KiB, whole request/result to 64 KiB, and reject lengths above remaining input before allocating. These limits apply at transport ingress too.

A revision is the array `[1, object, parent, type, body, signature]`. Object is the 16-byte canonical Destination. Parent is null for genesis or a 32-byte revision digest. Type is the UTF-8 vocabulary IRI. Body is a uint-keyed map defined by the type. For the bootstrap Object/Class an empty map suffices. For Property, keys 0, 1, 2 are source Destination, property type IRI and value; encode a reference as [0, destination] and a typed literal as [1, datatype IRI, lexical text]. For Relationship, keys 0, 1, 2 are source Destination, relationship type IRI and endpoint Destination. Subclasses inherit the structural checks of their base. Fields describing another object carry references and confer no authority over it.

Sign `CBOR(["rdo.revision.v1", 1, object, parent, type, body])` with the resolved owner's Ed25519 private signing key. The revision ID is SHA-256 of the entire encoded revision, including its 64-byte signature. Verify the object, owner key, signature and structure before use. A successor must keep object and type and name the exact accepted predecessor. Retain old bytes. A second genesis, fork or missing predecessor cannot replace an accepted head. On first contact an authenticated owner READ may supply a current signed snapshot as a checkpoint. Record that provenance. A replica retains the verified binding, signed revision and validation time for disconnected reads. It cannot advance the authoritative head.

## Owner operations

On an established Link to the reconstructed object, identify the initiator with RNS Link identification. Use native Link request path `/rdo/1/op`. Request data and response data are CBOR **byte strings at the RNS API boundary** (pass the encoded bytes once). Service checks the authenticated initiator and actual Link destination. A claimed principal in input cannot authorize a request.

| Structure | Ordered CBOR fields |
|---|---|
| Request | [1, id, object, op, expected, args, expires] |
| Result | [1, id, status, version, value] |

A client creates a random 16-byte id per logical request. Object must equal the receiving Destination. Expires is UTC Unix seconds, with `now < expires <= now + 300` at admission. Expected and version are 32-byte revision digests or null. Result value is null unless specified. Status assignments in order from 0 through 6 are APPLIED, REJECTED, PENDING, STALE, NOT_AUTHORIZED, NOT_FOUND and ERROR. For denial suppress the version and value. Drop undecodable/unidentified traffic. A well-formed unauthorized request gets NOT_AUTHORIZED before existence, version or data are disclosed.

| Op | Preconditions and result |
|---|---|
| 0 READ | Expected and args null. Check read policy. APPLIED returns current digest and exact signed revision bytes as a bstr. |
| 1 PROPOSE | Expected is the current digest. Args is the proposed replacement body map. Owner authorizes the caller and validates the body against the existing type. Only the owner may serialize, sign and commit the successor. APPLIED returns its digest and revision bytes. STALE returns the current digest to an authorized caller. |
| Other | Reject unassigned operations. M2 profiles may register their own Link request paths with typed operations and owner policy. |

Genesis is a local owner publication. Serialize compare-and-swap of the current head, the new immutable revision and the duplicate-request record in one durable transaction. Index requests by (authenticated Identity hash, object, id), store the exact request bytes and result through expiry. After authentication and current authorization, identical retries return the retained result. Different bytes with the same id are REJECTED before execution. Check expiry on retries and refuse new work if replay state cannot be retained. A pending decision may advance once under the same checks. After expiry or an unknown outcome, READ before issuing a fresh proposal. In a race on the same expected head, only one distinct proposal may commit. Offline replicas can queue an intent for later owner authorization.

## Qualification

The [vectors](../vectors/owner-v1.json) use public synthetic keys. Reconstruct every address and signed binding from the public inputs, including independent Property and Relationship addresses and three example M2 objects. JSON diagnostic values use decimal keys for uint-keyed maps and strings of the form `h'00ff'` for bytes. The other strings are text. `announce_packet_hex` contains a complete unstamped, unratcheted RNS packet, including header and Destination. The authenticated editor and reader are distinct from the owner. Each operation case starts afresh with the named head and any indicated retained result. Check the semantic request, response and revision bytes as well as the rejection cases. The fixture labels do not enter the address. A consumer can use the following harness contract in either language:

1. Feed raw announce bytes and a separately supplied expected Destination to the native RNS validator. Reconstruct an outbound Destination with the recovered public Identity and token.
2. Start an owner with the supplied genesis head and policy. Process each case from its specified initial state. Check status, digest, signature and the number of committed revisions. Repeated or rejected requests must preserve the expected head.
3. For **network qualification**, repeat over a clean two-process link in both Python-owner/Rust-client and Rust-owner/Python-client directions. Start each client with only the Destination and no cached path. Capture signed app-data retrieval, identified READ, authorized proposal, foreign denial, stale write, lost-response retry after restart and offline replica read after stopping the owner. Repeat with stamped path requests, announces and Link establishment against an enforcing peer using the RNS matrix in [Fabric #768](https://github.com/FreeTAKTeam/LXMF-rs/issues/768). Record software revisions and results. Packet and byte agreement alone covers only the offline portion of this gate.

### Offline Rust harness entry point

For each lookup vector, pass the decoded packet and the separately supplied 16-byte wanted address to the following check. `rns_core` is Fabric’s `reticulum-rs-core` library. This is the offline part of the harness; the two-process test above also has to retrieve the signed bytes from a path response.

```rust
use rns_core::destination::{new_out, DestinationAnnounce};
use rns_core::packet::Packet;

fn matches_object(raw: &[u8], wanted: &[u8]) -> bool {
    let Ok(packet) = Packet::from_bytes(raw) else { return false };
    let Ok(info) = DestinationAnnounce::validate(&packet) else { return false };
    let app = info.app_data;
    if app.len() != 19 || app[..3] != [0x82, 1, 0x50] {
        return false;
    }
    let aspect = format!("object.{}", hex::encode(&app[3..]));
    let recovered = new_out(info.destination.desc.identity, "rdo", &aspect);
    recovered.desc.name.as_name_hash_slice() == info.destination.desc.name.as_name_hash_slice()
        && recovered.desc.address_hash.as_slice() == wanted
        && packet.destination.as_slice() == wanted
}
```

[RDO #1](https://github.com/FreeTAKTeam/ReticulumDistributedObject/issues/1) owns this gate. [RPP #2](https://github.com/FreeTAKTeam/rpp-spec/issues/2) consumes the binding. The application requesting a file supplies its own access policy.
