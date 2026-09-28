# Polygon RPC Getting Started

A practical guide to Polygon PoS JSON-RPC covering network verification, blocks, balances, contracts, logs, and receipts.

## RPC endpoint

```text
https://polygon.api.onfinality.io/public
```

Polygon PoS is EVM-compatible and uses chain ID `137`.

## 1. Verify Polygon PoS Mainnet

```bash
curl -s https://polygon.api.onfinality.io/public \
  -H 'content-type: application/json' \
  --data '{"jsonrpc":"2.0","id":1,"method":"eth_chainId","params":[]}'
```

Expected result: `0x89`.

## 2. Get the latest block

```bash
curl -s https://polygon.api.onfinality.io/public \
  -H 'content-type: application/json' \
  --data '{"jsonrpc":"2.0","id":1,"method":"eth_blockNumber","params":[]}'
```

## 3. Read a POL balance

```bash
curl -s https://polygon.api.onfinality.io/public \
  -H 'content-type: application/json' \
  --data '{
    "jsonrpc":"2.0",
    "id":1,
    "method":"eth_getBalance",
    "params":["0xYOUR_ADDRESS","latest"]
  }'
```

The RPC result is a hex-encoded wei value.

## 4. Read contract state

```bash
curl -s https://polygon.api.onfinality.io/public \
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

## 5. Query event logs

```bash
curl -s https://polygon.api.onfinality.io/public \
  -H 'content-type: application/json' \
  --data '{
    "jsonrpc":"2.0",
    "id":1,
    "method":"eth_getLogs",
    "params":[{
      "fromBlock":"0xSTART_BLOCK",
      "toBlock":"0xEND_BLOCK",
      "address":"0xCONTRACT_ADDRESS"
    }]
  }'
```

Keep ranges bounded. Smaller windows are easier to retry and less likely to hit execution limits.

## 6. Check a transaction receipt

```bash
curl -s https://polygon.api.onfinality.io/public \
  -H 'content-type: application/json' \
  --data '{
    "jsonrpc":"2.0",
    "id":1,
    "method":"eth_getTransactionReceipt",
    "params":["0xTRANSACTION_HASH"]
  }'
```

Useful fields include `status`, `blockNumber`, `gasUsed`, and `logs`.

## 7. JavaScript example

```js
const RPC_URL = 'https://polygon.api.onfinality.io/public';

async function rpc(method, params = []) {
  const res = await fetch(RPC_URL, {
    method: 'POST',
    headers: {'content-type': 'application/json'},
    body: JSON.stringify({jsonrpc: '2.0', id: 1, method, params}),
  });

  const body = await res.json();
  if (body.error) throw new Error(body.error.message);
  return body.result;
}

const chainId = Number.parseInt(await rpc('eth_chainId'), 16);
if (chainId !== 137) throw new Error(`Unexpected chain ID: ${chainId}`);
console.log(Number.parseInt(await rpc('eth_blockNumber'), 16));
```

## Polygon PoS execution vs consensus

Application-facing Ethereum JSON-RPC requests are served by the Polygon PoS execution layer. Normal dApps, wallets, and backends generally do not need to interact directly with validator/checkpoint coordination just to read state or send transactions.

## Troubleshooting

### `eth_getLogs` is slow

Shrink the block range and checkpoint progress.

### Address exists on Ethereum but not Polygon

Contract deployments and state are chain-specific.

### Historical state is unavailable

Old block/state queries can require archive access.

### MATIC vs POL

Current Polygon PoS documentation uses POL as the native token. Older documentation and tooling may still contain MATIC terminology.

## Mainnet settings

| Setting | Value |
| --- | --- |
| Network | Polygon PoS Mainnet |
| Chain ID | `137` |
| Native token | POL |
| RPC | `https://polygon.api.onfinality.io/public` |
| WebSocket | `wss://polygon.api.onfinality.io/public-ws` |
| Explorer | `https://polygonscan.com` |

## Resources

- [Polygon PoS documentation](https://docs.polygon.technology/pos/)
- [OnFinality Polygon RPC](https://onfinality.io/en/networks/polygon)
- [OnFinality RPC network directory](https://onfinality.io/en/networks)
