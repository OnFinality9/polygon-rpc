# Polygon Chain RPC: Connect Existing EVM Tooling and Handle POL Fees

Polygon's current developer documentation treats Polygon Chain as an EVM-compatible chain: existing Solidity contracts and tools such as Foundry, Hardhat, ethers, and web3.js can be pointed at a Polygon RPC endpoint without changing the contract language or JSON-RPC model.

The current mainnet settings are:

| Property | Value |
| --- | --- |
| Chain ID | `137` |
| Parent chain | Ethereum |
| Gas token | POL |
| Explorer | Polygonscan |

Polygon's official RPC-provider list includes OnFinality, so the examples below use:

```bash
export POLYGON_RPC=https://polygon.api.onfinality.io/public
```

## Quick connection test with raw JSON-RPC

```bash
curl -s "$POLYGON_RPC" \
  -H 'content-type: application/json' \
  --data '{"jsonrpc":"2.0","id":1,"method":"eth_chainId","params":[]}'
```

Expected result:

```json
{"jsonrpc":"2.0","id":1,"result":"0x89"}
```

`0x89` is decimal `137`.

## If you use Foundry, test the RPC with `cast`

```bash
cast chain-id --rpc-url "$POLYGON_RPC"
cast block-number --rpc-url "$POLYGON_RPC"
```

Read a native POL balance:

```bash
cast balance 0xYOUR_ADDRESS --rpc-url "$POLYGON_RPC"
```

Foundry is useful here because Polygon is EVM-compatible; there is no Polygon-specific transaction format to learn for ordinary contract deployment and calls.

## Point Hardhat at Polygon Chain

A minimal Hardhat network entry looks like this:

```js
export default {
  solidity: '0.8.28',
  networks: {
    polygon: {
      url: 'https://polygon.api.onfinality.io/public',
      chainId: 137,
      accounts: process.env.PRIVATE_KEY ? [process.env.PRIVATE_KEY] : [],
    },
  },
};
```

Do not commit the private key to the repository. Load it from a secret manager or environment variable.

With the RPC URL and chain ID configured, normal Ethereum deployment scripts work on Polygon Chain.

## Read a contract directly

At the JSON-RPC layer, a read-only contract call is `eth_call`:

```bash
curl -s "$POLYGON_RPC" \
  -H 'content-type: application/json' \
  --data '{
    "jsonrpc":"2.0",
    "id":1,
    "method":"eth_call",
    "params":[
      {"to":"0xCONTRACT_ADDRESS","data":"0xABI_ENCODED_CALLDATA"},
      "latest"
    ]
  }'
```

For ABI-heavy applications, let Foundry, viem, or ethers encode the calldata.

## Polygon has a dedicated Gas Station

Polygon's official Gas Station builds recommendations from recent `eth_feeHistory` data. For mainnet, query:

```bash
curl -s https://gasstation.polygon.technology/v2
```

The response contains `safeLow`, `standard`, and `fast` suggestions with `maxPriorityFee` and `maxFee` values.

Polygon's current documentation also specifies a minimum priority fee on mainnet, so copying fee assumptions from Ethereum Mainnet is not a good strategy.

If you prefer to stay entirely inside JSON-RPC, you can inspect fee history directly:

```bash
curl -s "$POLYGON_RPC" \
  -H 'content-type: application/json' \
  --data '{
    "jsonrpc":"2.0",
    "id":1,
    "method":"eth_feeHistory",
    "params":["0xF","latest",[10,25,50]]
  }'
```

## Build an event indexer in bounded windows

```bash
curl -s "$POLYGON_RPC" \
  -H 'content-type: application/json' \
  --data '{
    "jsonrpc":"2.0",
    "id":1,
    "method":"eth_getLogs",
    "params":[{
      "fromBlock":"0xSTART_BLOCK",
      "toBlock":"0xEND_BLOCK",
      "address":"0xCONTRACT_ADDRESS",
      "topics":[]
    }]
  }'
```

For production indexing:

1. choose a bounded block range;
2. process and persist the results;
3. save the last successful block;
4. continue from that checkpoint;
5. retry a smaller window when a provider rejects a large response.

This is much more robust than repeatedly querying a huge range ending at `latest`.

## MATIC vs POL

Older tutorials, contracts, dashboards, and wallet screens may still say **MATIC**. Polygon's current documentation uses **POL** as the native gas and staking token on Polygon Chain.

When migrating an older integration, check more than the RPC URL:

- UI labels and token symbols;
- fee accounting;
- bridge assumptions;
- environment variables that still use `MATIC_*` names;
- monitoring rules that search logs for old terminology.

The chain ID remains `137`, so token naming migrations can otherwise be easy to miss.

## Read a receipt after deployment or execution

```bash
curl -s "$POLYGON_RPC" \
  -H 'content-type: application/json' \
  --data '{
    "jsonrpc":"2.0",
    "id":1,
    "method":"eth_getTransactionReceipt",
    "params":["0xTRANSACTION_HASH"]
  }'
```

Useful fields are `status`, `contractAddress`, `gasUsed`, and `logs`. A successful broadcast is not the same as successful EVM execution; always inspect the receipt.

## References

- [Polygon Chain RPC endpoints](https://docs.polygon.technology/pos/reference/rpc-endpoints/)
- [Building on Polygon Chain](https://docs.polygon.technology/pos/get-started/building-on-polygon/)
- [Polygon Gas Station](https://docs.polygon.technology/tools/gas/polygon-gas-station/)
- [OnFinality Polygon RPC](https://onfinality.io/en/networks/polygon)
- [OnFinality network directory](https://onfinality.io/en/networks)
