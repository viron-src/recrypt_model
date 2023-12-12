**Viron Recrypt Model Repository**
=======

Public gRPC model used for interfacing with the Recrypt exchange.

The public native gRPC API is available at: https://api.recrypt.net:8443

Optionally you can use gRPC Web API at https://api.recrypt.net if running an integration from within the browser (not recommended)

Visit https://t.recrypt.net/api to manage your API keys. This page also documents how to authenticate.

## API Limits

There are currently no API limits, however please be aware of some edge cases when interacting with this API; see below.

## Retry on failure

If you receive gRPC INTERNAL error with "vgw-db" as the message then this error can be retried. This is the only error that should be retried.

This error can occasionally occur during maintenance. Implement exponential backoff upto 3 minutes; check the website for any system notifications if still occurring after this.

We guarantee no downtime on all days except on Tuesdays at 4:00 to 5:00 AM UTC - during this time this error may appear and should last no longer than 1 minute. If this is not acceptable clear your orders before this time.

Any other gRPC error that is not documented on the request itself should be not possible when using the API correctly under normal operations.

## Idempotency

If you require idempotency on any request that involves the transfer or trade of currency then an idempotency UUID field should be available for that request as documented in the protobuf files.

Randomly generate this field for each request you want idempotent and re-use the field for the request if it failed with an undocumented error. Once the first gRPC OK response is received this means the transaction has successfully occurred and further requests will return OK - performing no further changes.

Changing the payload of the request but keeping the idempotent field the same is undefined behaviour; there is no guarantee to know which payload was processed. Make sure to change idempotent field when changing any other field in the RPC. 

## Success failures

In extremely rare cases its possible a transaction is executed within the exchange system but a gRPC error (such as INTERNAL vgw-db mentioned above) is returned to your client.

Make use of idempotency fields mentioned above when interfacing with the system to provide highly available state synchronization between integration points.

## Timeouts

If you receive gRPC DEADLINE EXCEEDED with "vgw-db" as the message then the transaction/request timed out. This should never occur under normal operations, but it's worth being aware of this error code during extraordinary events.

## Streaming RPC's

It's possible for long-lived streaming RPCs to gracefully end with OK without requesting an end once every few days or longer. Simply re-connect in this scenario.

## Backwards compatability

We will never make a change that breaks backwards compatability within the API. New fields may be added to existing RPCs.

If a pre-existing API becomes deprecated and then removed it will start returning NOT_FOUND.

## Enjoy

The .proto files should be self documenting for the API surface which is intended to be available to API keys.

## Notes

See https://grpc.github.io/grpc/core/md_doc_statuscodes.html for error codes mentioned above.

### Copyright © Viron Software ⨈