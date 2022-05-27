# Sidetree

## Protocol

Sidetree is a protocol for creating scalable [Decentralized Identifier](https://w3c.github.io/did-core/) networks 
that can run atop any existing decentralized anchoring system (e.g. Bitcoin, Ethereum, distributed ledgers, 
witness-based approaches). The protocol allows users to create globally unique, user-controlled identifiers 
and manage their associated PKI metadata, all without the need for centralized authorities or trusted third parties. 
The syntax of the identifier and accompanying data model used by the protocol is conformant with the 
[W3C Decentralized Identifiers](https://w3c.github.io/did-core/) specification. 
Implementations of the protocol can be codified as their own distinct DID Methods and registered in the 
[W3C DID Method Registry](https://w3c.github.io/did-spec-registries/#did-methods).

See the [latest spec](https://identity.foundation/sidetree/spec/) <span></span> for the full Sidetree specification.

## Sidetree Interactions

![Sidetree Interactions Diagram](../../_static/orb/sidetree-interactions.png)

Orb node will validate each Sidetree operation as per specification. Valid operation will then be added 
to the batch writer queue and unpublished operations store. Batch writer will then batch multiple Sidetree 
operations together and store them in Sidetree batch files as per 
[Sidetree file structure spec](https://identity.foundation/sidetree/spec/#file-structures). 

Next, Sidetree batch file information will be stored into anchor index and anchor index will be witnessed 
as per witness policy. Observer will process witnessed anchor index and Sidetree batches as per 
[Sidetree transaction operation processing](https://identity.foundation/sidetree/spec/#transaction-operation-processing) 
and store DID operations into operations store.  It will delete observed DID operation from unpublished operation store.
DID will be resolved from stored DID operations.

## DID Operations

Sidetree supports Create, Update, Recover and Deactivate(CRUD) operations. 
Check our [Sidetree did operations](https://identity.foundation/sidetree/spec/#did-operations) and 
[Sidetree API spec](https://identity.foundation/sidetree/api/) to learn how to construct JSON for Sidetree CRUD operations.

## DID Resolution

Upon invocation of resolution, DID resolver will first retrieve all published (observed) and unpublished operations 
for the DID Unique Suffix of the DID URI being resolved.

DID document processing starts by iterating over all Create operations (ordered by anchoring time) and applying 
first valid Create operation. 
Next step in the processing is applying Recover and Deactivate operations based on previous operation Next Recovery Commitment. 
Once Recover and Deactivate operations have been applied to the document (or if no Recovery and Deactivate operations are present) 
the resolver will proceed to Update operation processing based on previous operation Next Update Commitment only if 
the Next Update Commitment value is present and no Deactivate operations were successfully processed earlier.

The resolver will check each operation for validity (including validating signature for update, recover and deactivate operations) 
before it applies that operation patch and commitments to DID Document.

See [Sidetree resolution spec](https://identity.foundation/sidetree/spec/#resolution) for more details about resolution.

## REST Endpoints

Sidetree resolution endpoint:

/sidetree/v1/identifiers

Sidetree operations endpoint:

/sidetree/v1/operations

See the [API spec](https://identity.foundation/sidetree/api/) for the full API specification to interact with a Sidetree node.

## Versioning

## Sidetree References

See the [latest spec](https://identity.foundation/sidetree/spec/) for the full Sidetree specification.

See the [API spec](https://identity.foundation/sidetree/api/) for the full API specification to interact with a Sidetree node.

See the [reference implementation document](https://github.com/decentralized-identity/sidetree/blob/master/docs/core.md) for description of the reference implementation.

See the [reference implementation](https://github.com/decentralized-identity/sidetree) for Sidetree reference implementation.


## Operation/Resolution Caching

Orb node provides two unique caching features for resolving DIDs:
1. resolve DIDs using unpublished operations
2. resolve DIDs using published/unpublished operations from anchor origin

### Operation/Resolution Caching Parameters

The following section enumerates the startup parameters that have to be configured in order to enable
operation caching and resolution from anchor origin domain.

Parameter [enable-unpublished-operation-store](../parameters.html#enable-unpublished-operation-store)
should be set to true to enable un-published operation store. If set to true Orb node will
upon validating Sidetree operation insert this operation into unpublished operations store
and use it immediately for DID resolution. Hence, Orb clients will be able to immediately
see their change (without waiting for anchoring/observing process to complete).
This unpublished Sidetree operation will be removed from unpublished operation store
when Orb node is done observing Sidetree operation. At this point Sidetree operation has been
stored into published operation store and will be used for DID resolution.


Parameter [unpublished-operation-store-operation-types](../parameters.html#unpublished-operation-store-operation-types)
determines the operation types that will be caches. Default value is "create,update" which enables storing unpublished 'create' and 'update' operations into
unpublished store and using those unpublished 'create' and 'update' operations for resolving document.


Parameter [resolve-from-anchor-origin](../parameters.html#resolve-from-anchor-origin)
should be set to "true" in order to resolve DID from anchor origin.
Orb node will resolve DID document locally,
determine DID's anchor origin and then resolve DID document from anchor origin Orb node.
If anchor origin Orb node has additional published/unpublished operations then Orb node will
add those operations for DID resolution.
This feature allows non origin servers to immediately resolve updated DIDs.

Parameter [include-unpublished-operations-in-metadata](../parameters.html#include-unpublished-operations-in-metadata)
should be set to "true" to include unpublished operations in metadata. This setting is required
on anchor origin node if non anchor origin nodes wish to resolve from anchor origin.

Parameter [include-published-operations-in-metadata](../parameters.html#include-published-operations-in-metadata)
should be set to "true" to include published operations in metadata. This setting is required
on anchor origin node if non anchor origin nodes wish to resolve from anchor origin.

### Operation Update Parameters

In order to ensure consistent operation caching, Orb client will normally issue
document update against anchor origin Orb node. However, if Orb node receives
an update operation for different anchor origin it will resolve DID against
anchor origin Orb node and make sure it has processed the latest operations
from anchor origin Orb node before accepting an update operation.

Parameter [verify-latest-from-anchor-origin](../parameters.html#verify-latest-from-anchor-origin)
should be set to "true" to verify if Orb node has already processed the latest operations
from anchor origin before accepting an update operation. If Orb node is behind in
processing DID operations (comparing to anchor origin) then Sidetree operation
will be rejected.
This feature also guards against DID document branching. 


