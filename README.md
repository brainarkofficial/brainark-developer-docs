# BrainArk developer docs

Everything you need to build on BrainArk (chain ID **1236**, native coin **BAK**).

## 1. Connect
| | |
|---|---|
| Chain ID | `1236` (hex `0x4d4`) |
| RPC | `https://rpc.brainark.online` |
| Currency | BAK, 18 decimals |
| Explorer | https://explorer.brainark.online |

Add BrainArk to a browser wallet (BAKvault, MetaMask, …):
```js
await window.ethereum.request({
  method: 'wallet_addEthereumChain',
  params: [{ chainId: '0x4d4', chainName: 'BrainArk', nativeCurrency: { name: 'BrainArk', symbol: 'BAK', decimals: 18 },
             rpcUrls: ['https://rpc.brainark.online'], blockExplorerUrls: ['https://explorer.brainark.online'] }],
});
```

## 2. Read the chain
Use any EVM library, or [brainark-sdk](https://github.com/brainarkofficial/brainark-sdk) (viem-based: chain definition,
addresses, ABIs and helpers like `getActiveListings` / `getAsset`). Verified contract source:
[brainark-contracts](https://github.com/brainarkofficial/brainark-contracts).

## 3. RWA REST API
Base URL `https://rwa.brainark.online/api/v1` — full reference in **[API.md](API.md)**.
1. Open https://rwa.brainark.online/developer and connect your wallet.
2. **Sign in** — one free wallet signature (no transaction, no fee) proves you own the wallet; it lasts 12 hours.
3. Create a key. **It is shown only once** — store it safely. BrainArk keeps only a fingerprint and cannot show it again;
   if you lose it, create a new key and revoke the old one.
4. Send it as `X-API-Key: <key>` or `Authorization: Bearer <key>`.

## Security
Report vulnerabilities privately: **Security → Report a vulnerability** on
[brainark-contracts](https://github.com/brainarkofficial/brainark-contracts).
