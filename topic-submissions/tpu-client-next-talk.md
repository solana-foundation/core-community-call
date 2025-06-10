#  Topic Submission

## Presenter
- Name: Kirill Lykov
- Affiliation: Anza
- Role: Software Engineer

## Topic Details
### Summary

[tpu-client-next](https://github.com/anza-xyz/agave/tree/master/tpu-client-next) is a new crate in the Agave repository.
It implements the client-side logic for the TPU protocol and is designed around an agent-based architecture.

### Technical Overview

Tpu-client-next is a crate which is currently used as a default client-side code for the agave validator and replaces ConnectionCache in this role.
This crate might be used to implement different client-side implementations for TPU traffic.
It is well-documented and provides a rich configurability allowing to fine tune client code for your specific needs.
API-wise, to interact with this client code one should use only one tokio channel which makes it a perfect fit for agent-based code architectures.
It has been tested on private cluster and testnet for forwarding and RPC.
Beside of that this client code is used for in-house `transaction-bench` tool we internally use for stress testing validator.

`tpu-client-next` is the default client-side implementation used by the Agave validator 2.3, replacing the older `ConnectionCache`.
It enables flexible and modular implementations of TPU traffic clients and is designed with rich configurability to allow fine-tuning for specific use cases.

The crate is well-documented and exposes a simple interface: interaction with the client happens via a single Tokio channel, making it an ideal choice for agent-based systems.

It has been tested for both forwarding and RPC on private clusters and testnet environments.
Beside of that, it powers our internal transaction-bench tool, which is used for validator stress testing.


## Date's available (Calls are typically on the 3rd Friday of each month)
June 20th

## Time Required
15-20min

## Additional Notes
[Any other relevant information for the call organizers]
