---
sidebar_position: 10
---

# Debug EVM Transactions

Geth's debug APIs and OpenEthereum's trace module provide non-standard RPC methods for getting a deeper insight into transaction processing.

> Thanks to the PureStake team, the Polkadot ecosystem has tracing capabilities similar to that of Geth, and OpenEthereum. Astar Network implements the same approach for Astar EVM tracing, due to it being the best solution we have at the moment, for the Polkadot ecosystem.

## Debugging RPC Methods

The following RPC methods are available:

### [debug_traceTransaction](https://geth.ethereum.org/docs/interacting-with-geth/rpc/ns-debug#debugtracetransaction)

* This call will attempt to run the transaction in the exact same manner as it was executed on the network. It will replay any transaction that may have been executed prior to this one before it will finally attempt to execute the transaction that corresponds to the given hash.
* Optional params:
* `disableStorage(boolean)` - (default: false) setting this to true disables storage capture
* `disableMemory(boolean)` - (default: false) setting this to true disables memory capture
* `disableStack(boolean)` - (default: false) setting this to true disables stack capture

### [debug_traceBlock](https://geth.ethereum.org/docs/interacting-with-geth/rpc/ns-debug#debugtraceblock)

* This call will return a full stack trace of all invoked opcodes of all transaction that were included in this block.
* variants to `debug_traceBlockByHash` and `debug_traceBlockByNumber` are also available

### [debug_traceCall](https://geth.ethereum.org/docs/interacting-with-geth/rpc/ns-debug#debugtracecall)

* This call will run an _eth-call-like_ execution within the context of the given block execution using the final state of parent block as the base.

### [trace_filter](https://openethereum.github.io/JSONRPC-trace-module#trace_filter)

* Optional params:
  * `fromBlock(uint blockNumber)` - either block number (hex), earliest which is the genesis block or latest (default) best block available. Trace starting block
  * `toBlock(uint blockNumber)` - either block number (hex), earliest which is the genesis block or latest best block available. Trace ending block
  * `fromAddress(array addresses)` - filter transactions done from these addresses only. If an empty array is provided, no filtering is done with this field
  * `toAddress(array addresses)` - filter transactions done from these addresses only. If an empty array is provided, no filtering is done with this field
  * `after(uint offset)` - default offset is 0. Trace offset (or starting) number
  * `count(uint numberOfTraces)` - number of traces to display in a batch

There are some default values that you should be aware of:

* The maximum number of trace entries a single request of `trace_filter` is allowed to return is `500`. A request exceeding this limit will return an error
* Blocks processed by requests are temporarily stored in cache for `300` seconds, after which they are deleted.

To change the default values you can add CLI flags when spinning up your tracing node.

## Run a Debugging Node

:::caution
EVM tracing features available from Astar 5.1 release.
:::

To use the supported RPC methods, you need to run a node in debug mode, which is slightly different than running a full node. Additional flags will also need to be used to tell the node which of the non-standard features to support.

Spinning up a debug or tracing node is similar to running a full node. However, there are some additional flags that you may want to enable specific tracing features:

* `--ethapi=debug` - optional flag that enables `debug_traceTransaction`
* `--ethapi=trace` - optional flag that enables `trace_filter`
* `--ethapi=txpool` - optional flag that enables `txpool_content`, `txpool_inspect`, `txpool_status`
* `--wasm-runtime-overrides=<path/to/overrides>` - required flag for tracing that specifies the path where the local Wasm runtimes are stored
* `--runtime-cache-size 64` - required flag that configures the number of different runtime versions preserved in the in-memory cache to 64
* `--ethapi-trace-max-count <uint>` - sets the maximum number of trace entries to be returned by the node. _The default maximum number of trace entries a single request of trace_filter returns is_ **500**
* `--ethapi-trace-cache-duration <uint>` - sets the duration (in seconds) after which the cache of `trace_filter`, for a given block, is discarded. _The default amount of time blocks are stored in the cache is **300** seconds_

:::info
EVM tracing node installation manual available on [this page](/docs/build/nodes/evm-tracing-node).
:::

### Using the Debug/Tracing API

Once you have a running tracing node, you can open your terminal to run curl commands and start to call any of the available JSON RPC methods.

For example, for the `debug_traceTransaction` method, you can make the following JSON RPC request in your terminal:

:::caution
`--ethapi=debug` flag as tracing node argument required to expose this API.
:::

```Bash
curl http://127.0.0.1:9944 -H "Content-Type:application/json;charset=utf-8" -d \
  '{
    "jsonrpc":"2.0",
    "id":1,
    "method":"debug_traceTransaction",
    "params": ["0xc74f3219cf6b9763ee5037bab4aa8ebe5eafe85122b00a64c2ce82912c7d3960"]
  }'
```

The node responds with the step-by-step replayed transaction information.

For the `trace_filter` call, you can make the following JSON RPC request in your terminal (in this case, the filter is from block 20000 to 25000, only for transactions where the recipient is 0x4E0078423a39EfBC1F8B5104540aC2650a756577, it will start with a zero offset and provide the first 20 traces):

```Bash
curl http://127.0.0.1:9944 -H "Content-Type:application/json;charset=utf-8" -d   '{
    "jsonrpc":"2.0",
    "id":1,
    "method":"trace_filter","params":[{"fromBlock":"4142700","toBlock":"4142800","toAddress":["0xb1dD8BABf551cD058F3B253846EB6FA2a5cabc50"],"after":0,"count":20}]
  }'
```

The node responds with the trace information corresponding to the filter.

### Using transaction pool API

Let's get pool status using `curl` HTTP POST request.

:::caution
`--ethapi=txpool` flag as tracing node argument required to expose this API.
:::

```Bash
curl http://127.0.0.1:9944 -H "Content-Type:application/json;charset=utf-8" -d \
  '{
    "jsonrpc":"2.0",
    "id":1,
    "method":"txpool_status", "params":[]
  }'
```
