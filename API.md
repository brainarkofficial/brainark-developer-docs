# BrainArk RWA API — v1 reference

Base URL: `https://rwa.brainark.online` · Chain: BrainArk (1236) · Verified against the live system 2026-10-17.

## Authentication
Every `/api/v1/*` endpoint except `GET /api/v1/status` needs an API key:
```
X-API-Key: <your key>        or        Authorization: Bearer <your key>
```
Keys are created at https://rwa.brainark.online/developer after a one-time wallet sign-in (a free signature). A key is shown
once at creation and only its fingerprint is stored.

### Plans
| Plan | Price | Rate limit | Writes |
|---|---|---|---|
| Free | $0 | 10 / min · 5,000 / day | issue, transfer, marketplace, webhooks |
| Basic | $10 / month | 300 / min · 50,000 / day | Free + register asset types |
| Pro | $99 / month | 1,000 / min · 500,000 / day | all write scopes |
| Enterprise | $499 / month | 10,000 / min · 10,000,000 / day | per-organisation scopes and wallet |

**How writes are paid for:** each Free/Basic/Pro key gets its **own custodial wallet** on BrainArk; you fund it with BAK and
write calls are signed and paid from it. Enterprise organisations get one organisation wallet. BrainArk's own wallets never pay
for your writes.

## Read endpoints
| Method | Path | Notes |
|---|---|---|
| GET | `/api/v1/status` | **No key.** Health + live contract addresses |
| GET | `/api/v1/assets?owner=0x…` | Assets owned by an address |
| GET | `/api/v1/assets/:tokenId` | One asset |
| GET | `/api/v1/nft/:tokenId` | ERC-721 metadata JSON (name, description, image, attributes) |
| GET | `/api/v1/marketplace` | Active listings |
| GET | `/api/v1/marketplace/:id` | One listing |
| GET | `/api/v1/bak-price` | Current BAK price in USD |

```bash
curl https://rwa.brainark.online/api/v1/status
curl "https://rwa.brainark.online/api/v1/assets?owner=0xYourAddress" -H "X-API-Key: $KEY"
```

## Write endpoints
Signed and broadcast by the server from your key's (or organisation's) custodial wallet. Each needs the scope shown.
| Method | Path | Scope | Body |
|---|---|---|---|
| POST | `/api/v1/write/issue-asset` | `write:issue` | `assetType`, `name`, optional `description`, `image`, `externalUrl`, `ipfsHash`, `stakedAmount` (BAK, default/min 0.1), `recipientAddress`, `metadata` |
| POST | `/api/v1/write/batch-issue` | `write:issue` | `{ "assets": [ … up to 50 … ] }` — sent as separate on-chain transactions |
| POST | `/api/v1/write/transfer` | `write:transfer` | `tokenId`, `to` |
| POST | `/api/v1/write/marketplace/list` | `write:marketplace` | `tokenId`, `priceBAK`, `durationDays` (1–365, default 30) |
| POST | `/api/v1/write/marketplace/buy` | `write:marketplace` | `listingId`, optional `maxPriceBAK` (slippage guard) |
| POST | `/api/v1/write/marketplace/delist` | `write:marketplace` | `listingId` |
| POST | `/api/v1/write/register-asset-type` | `admin:types` | `typeName`, `schema` |

## Enterprise endpoints
`/api/enterprise/wallet` (organisation wallet + balance) · `/api/enterprise/keys` (list / create / revoke organisation keys) ·
`/api/enterprise/webhooks` (register / list / remove webhooks for `asset.issued`, `asset.transferred`, `asset.listed`,
`asset.sold`, `asset.delisted`, `asset.unstaked`; deliveries carry an `X-BrainArk-Signature` HMAC header).

## Using the contracts directly
Assets are issued through the RWA app or the write API (the registry only accepts issuance from its claim/extension contracts).
Trading directly on the marketplace:
```js
import { ethers } from 'ethers';
const MP = '0x5C1A47d5E813211025e6653099BAAc89E8DeF9C3', AR = '0xFf76Ea2EED588D0bFAF5466Bd0F20C3D980bd95D';
const mp = new ethers.Contract(MP, [
  'function listAsset(uint256 assetId, uint256 priceInBAK, uint256 durationSeconds) returns (uint256)',
  'function buyWithBAK(uint256 listingId) payable',
  'function cancelListing(uint256 listingId)',
  'function getListing(uint256) view returns (tuple(uint256 id,uint256 assetId,address seller,uint256 priceInBAK,uint256 priceInNative,uint256 startTime,uint256 endTime,bool active))',
], signer);
const ar = new ethers.Contract(AR, ['function approve(address to, uint256 tokenId)'], signer);

await ar.approve(MP, tokenId);                                   // seller: allow the marketplace to move the NFT
await mp.listAsset(tokenId, ethers.parseEther('100'), 30 * 86400);

const l = await mp.getListing(listingId);
await mp.buyWithBAK(listingId, { value: l.priceInBAK });         // buyer pays EXACTLY the price
```

## Fees (live values)
| Action | Fee |
|---|---|
| Marketplace sale | 2% of the price, taken from the seller's proceeds (plus any creator royalty); the buyer pays the listed price |
| Unstake an RWA asset | 3% of the staked BAK |
| Network gas | paid in BAK |

## Errors
| HTTP | Meaning |
|---|---|
| 400 | Missing or invalid fields |
| 401 | Missing/invalid API key — or, for key management, missing/expired wallet signature |
| 403 | Key lacks the required scope |
| 410 | Key reveal requested — keys are shown only once |
| 429 | Rate limit exceeded |
| 500 | On-chain write failed (e.g. reverted) |
