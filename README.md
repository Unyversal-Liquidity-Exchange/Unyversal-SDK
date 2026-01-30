# Unyversal SDK

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![NPM Version](https://img.shields.io/npm/v/@unyversal/sdk.svg?style=flat-square)](https://www.npmjs.com/package/@unyversal/sdk)
[![NPM Downloads](https://img.shields.io/npm/dm/@unyversal/sdk.svg?style=flat-square)](https://www.npmjs.com/package/@unyversal/sdk)
[![Discord](https://img.shields.io/discord/1234567890?color=7289da&label=Discord&logo=discord&logoColor=white)](https://discord.gg/ule-ecosystem)
[![Twitter](https://img.shields.io/twitter/follow/uleprotocol?style=social)](https://twitter.com/uleprotocol)

**Unyversal-SDK** is the official **multichain developer toolkit** and **universal translator** for the **Unyversal Liquidity Exchange (ULE)** ecosystem. It abstracts away the complexity of cross-chain interactions, wallet connections, gas management, bridging, and protocol calls --- allowing developers to build once and reach users across Ethereum, Solana, Base, Polygon, ULE mainnet, and more, with a single, clean API surface.

Powered by the **ULE-Protocol** liquidity engine, Unyversal-SDK makes building **NFT marketplaces**, **DeFi integrations**, **instant liquidity tools**, and **full-stack dApps** feel like developing for a single chain --- while automatically handling chain detection, token swaps, gas zaps, synthetic asset mirroring, and zero-fee listing signatures.

Whether you're integrating ULE into an existing app or building natively on the ULE Network, Unyversal-SDK is your one-stop library for seamless multichain liquidity and user experience.

## Table of Contents

- [Features](#features)
- [Installation](#installation)
- [Quick Start](#quick-start)
- [Core Concepts](#core-concepts)
- [API Reference](#api-reference)
  - [Provider & Wallet](#provider--wallet)
  - [Gas Zap](#gas-zap)
  - [NFT & Token Operations](#nft--token-operations)
  - [Cross-Chain Bridging](#cross-chain-bridging)
  - [ULE-Protocol Hooks](#ule-protocol-hooks)
- [Examples](#examples)
- [Configuration](#configuration)
- [Local Development & Testing](#local-development--testing)
- [Supported Chains](#supported-chains)
- [Contributing](#contributing)
- [License](#license)

## Features

- **Automatic Chain Detection** --- Detects user's current chain and suggests/wallets switches
- **Gas-Zap** --- One-click token → $ULE conversion across chains (via bridges & DEXs)
- **Zero-Gas Listing Signatures** --- Create ULE-Protocol listings off-chain
- **Instant Exit / Buyback** --- Sell NFTs instantly to protocol pools
- **Synthetic Asset Mirroring** --- Lock on origin chain → trade mirrored asset on ULE
- **Unified NFT & Token API** --- ERC-721/1155, SPL, native ULE assets in one interface
- **Wallet Abstraction** --- Works with MetaMask, WalletConnect, Phantom, Osmium, etc.
- **ULE Local Mode** --- Seamless integration with UNYTE for local testing
- **TypeScript-First** --- Full types, autocompletion, and strong safety
- **Modular** --- Tree-shakeable; import only what you need

## Installation

```bash
# Using npm
npm install @unyversal/sdk

# Using yarn
yarn add @unyversal/sdk

# Using pnpm
pnpm add @unyversal/sdk
```

Peer dependencies (install if needed):

Bash

```
npm install viem wagmi ethers@6 @tanstack/react-query
```

Quick Start
-----------

tsx

```
import { createUnyversalClient, gasZap } from '@unyversal/sdk'
import { mainnet, ule } from '@unyversal/chains'
import { http } from 'viem'
import { createConfig } from 'wagmi'

const config = createConfig({
  chains: [mainnet, ule],
  transports: {
    [mainnet.id]: http(),
    [ule.id]: http('https://rpc.ule.network'),
  },
})

const client = createUnyversalClient({
  wagmiConfig: config,
  defaultChainId: ule.id, // preferred target chain
})

// Example: Buy an NFT with automatic gas zap if user has only ETH
async function buyNFT() {
  await gasZap({
    fromToken: 'ETH',
    toToken: '$ULE',
    amount: '0.05',
    onSuccess: async () => {
      const tx = await client.nft.buy({
        collection: '0xBC4CA0EdA7647A8aB7C2061c2E118A18a936f13D', // Bored Ape
        tokenId: 1234n,
        maxPrice: 12n * 10n ** 18n, // 12 ETH equivalent in $ULE
      })
      console.log('Purchased!', tx.hash)
    },
  })
}
```

Core Concepts
-------------

-   **Client** --- Central instance (createUnyversalClient) that manages providers, chains, and protocol state.
-   **Gas Zap** --- Automatically bridges/swaps tokens to $ULE for ULE transactions.
-   **Signature Listings** --- Off-chain signed messages for zero-gas ULE listings.
-   **Synthetic Mirrors** --- 1:1 wrapped assets from other chains tradable on ULE.
-   **Instant Exit** --- Sell to protocol-owned liquidity pools at oracle price.

API Reference
-------------

### Provider & Wallet

TypeScript

```
const client = createUnyversalClient({ wagmiConfig })

// Get current chain & switch if needed
const chain = await client.getCurrentChain()
await client.switchTo(ule)

// Connect wallet
await client.connect()
```

### Gas Zap

TypeScript

```
await gasZap({
  fromToken: 'ETH' | 'USDC' | 'SOL' /* etc */,
  toToken: '$ULE',
  amount: '1.5',
  slippage: 0.5, // %
  onProgress?: (step: string) => void,
  onSuccess?: () => Promise<void>,
})
```

### NFT & Token Operations

TypeScript

```
// List NFT with zero gas (signature only)
const signature = await client.nft.signListing({
  collection: '0x...',
  tokenId: 5678n,
  price: 10n * 10n ** 18n,
})

// Buy
await client.nft.buy({ collection, tokenId, maxPrice })

// Instant sell to protocol pool
const proceeds = await client.nft.instantExit({
  collection: '0x...',
  tokenId: 999n,
})
```

### Cross-Chain Bridging

TypeScript

```
// Lock Pudgy on Ethereum → get mirror on ULE
await client.bridge.lockAndMirror({
  sourceChain: mainnet,
  collection: '0xBd3531dA...',
  tokenId: 420n,
})

// Trade mirror on ULE, then bridge back if desired
```

Examples
--------

See the /examples folder or live demos:

-   Basic NFT viewer & buy button
-   Multichain wallet dashboard
-   Instant sell widget
-   Gas-zap checkout flow

Configuration
-------------

TypeScript

```
createUnyversalClient({
  wagmiConfig,
  defaultChainId: ule.id,
  rpcUrls: {
    [ule.id]: 'https://rpc.ule.network',
  },
  oracleProvider: 'chainlink', // or custom
  slippageTolerance: 1.0,
  enableTestMode: process.env.NODE_ENV === 'development',
})
```

Local Development & Testing
---------------------------

Use **UNYTE** for local simulation:

Bash

```
# Terminal 1: Local ULE node
npx unyte node

# Terminal 2: Your app
npm run dev
```

The SDK auto-detects localhost:8545 and switches to test mode with fake $ULE.

Supported Chains
----------------

| Chain | Status | Notes |
| --- | --- | --- |
| ULE Mainnet | Native | $ULE gas, full protocol support |
| Ethereum | Full | Bridging & mirroring |
| Polygon | Full | Low-cost alternative |
| Base | Full | Coinbase ecosystem |
| Solana | Beta | SPL → ULE synthetic |
| Arbitrum | Planned | Coming soon |

Contributing
------------

We welcome PRs, bug reports, and feature ideas!

1.  Fork the repo
2.  Create your feature branch (git checkout -b feat/amazing-feature)
3.  Commit your changes (git commit -m 'Add amazing feature')
4.  Push to the branch (git push origin feat/amazing-feature)
5.  Open a Pull Request

Join the [Discord](https://discord.gg/ule-ecosystem) for real-time discussion.

License
-------

MIT License --- see LICENSE

* * * * *

**Build once. Reach everywhere.** **Unyversal-SDK --- Liquidity without borders.**
