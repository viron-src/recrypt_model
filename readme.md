**Viron Recrypt Model Repository**
=======

Public gRPC model used for interfacing with the Recrypt exchange.

The public native gRPC API is available at: https://api.recrypt.net:8443

Optionally you can use gRPC Web API at https://api.recrypt.net if running an integration from within the browser (not recommended)

Visit https://recrypt.net/api to manage your API keys. This page also documents how to authenticate in the next section.

## Authenticating

Use the API key generated at https://recrypt.net/api to authenticate by inserting gRPC metadata to the request with the key 'apikey' and value of the API key contents that you copied: https://grpc.io/docs/guides/metadata/

The gRPC protobuf models self-document which requests are available for use by API keys and whether it requires a readonly key or write-enabled key.

Attempting to use an IP restricted API key from an IP address that is not authorized will result in grpc error code UNAUTHENTICATED with message 'Session Key'.

API keys do not expire. You are responsible for rotating keys yourself based on your own security requirements.

## API Limits

There are currently no API limits, however please be aware of some edge cases when interacting with this API; see below.

NOTE: This will likely change in the future.

## Retry on failure

If you receive gRPC UNAVAILABLE error with "vgw-db" as the message then this error can be retried. This is the only error that should be retried.

This error can occasionally occur during maintenance. Implement exponential backoff upto 3 minutes; check the website for any system notifications if still occurring after this.

We guarantee no downtime on all days except on Tuesdays at 18:00 to 18:30 AM UTC - during this time this error may appear and should last no longer than 1 minute. If this is not acceptable clear your market orders before during this interval.

If there is no downtime notice banner on the website a few days prior this means there is no downtime scheduled for that week. You can additionally use Core::MaintenanceCheck RPC to automate this.

Any other gRPC error that is not documented below or on the request itself should be not possible when using the API correctly under normal operations.

## Idempotency

If you require idempotency on any request that involves the transfer or trade of currency then an 'idempotency_key' UUID/GUID field will be optionally available for that request as documented in the protobuf files.

Randomly generate this field for each request you want idempotent and re-use the field for the request if it failed with an undocumented or unexpected error. Once the first gRPC status OK or gRPC status ALREADY_EXISTS response is received this means the request/transaction has successfully occurred once and further requests with the same idempotency key UUID will return ALREADY_EXISTS - performing no further changes.

Changing the payload of the request but keeping the idempotency key field the same is undefined behaviour; there is no guarantee to know which payload was processed by the system. Make sure to change idempotency key field when changing any other field in the RPC.

The 'idempotency_key' field is global across the whole system and across all RPC's that support it, even between different accounts.

## Success failures

In extremely rare cases its possible a transaction is executed within the exchange system but a gRPC error (such as UNAVAILABLE with "vgw-db" as the message mentioned above or DEADLINE_EXCEEDED) is returned to your client.

Make use of idempotency fields mentioned above when interfacing with the system to provide highly available state synchronization between integration points.

## Internal timeouts

If you receive gRPC DEADLINE_EXCEEDED with "vgw-db" as the message then the transaction/request timed out from within the database itself and in most cases the transaction or request was rolled back.

This is different to DEADLINE_EXCEEDED with any other message mentioned above. DEADLINE_EXCEEDED should never occur under normal operations regardless, but it's worth being aware of this error code during extraordinary events.

## Streaming RPC's

It's possible for long-lived streaming RPCs to gracefully end with OK without requesting an end once every few days or longer due to load-balancing or system update events. Simply re-connect in this scenario.

## Backwards compatability

We will never make a change that breaks backwards compatability within the API. New fields may be added to existing RPCs.

If a pre-existing API becomes deprecated and then removed it will start returning NOT_FOUND rather than UNIMPLEMENTED.

## Compliance questioning

As part of regulatory compliance in rare cases if the system requires more user information before permitting a transaction then the UI will prompt for such information. In the case where an API is being used a gRPC error of ABORTED will be returned.

Log into the website and respond to the provided question to stop this error.

## Transaction limits

Attempting to perform a transaction that exceeds account transaction limits will result in gRPC error FAILED_PRECONDITION. Make a ticket to support to increase your limit.

## Enjoy

The .proto files should be self documenting for the API surface which is intended to be available to API keys.

## Notes

See https://grpc.github.io/grpc/core/md_doc_statuscodes.html for error codes mentioned above.

### Copyright © Viron Software ⨈