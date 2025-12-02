# Glossary Template

Use this template to define protocol-specific terminology. A good glossary reduces confusion and ensures consistent language across your documentation.

---

# Glossary

This glossary defines terms used throughout the [Protocol Name] documentation.

## Protocol Terms

### Controller

The main entry-point contract for user interactions. The Controller handles registration, renewal, and coordinates actions across other protocol contracts.

See: [Controller Documentation](./controller.md)

### Node

A unique identifier for a name in the protocol, created by hashing the name. For hierarchical names, the node is calculated recursively.

**Example:**
```
node("example.eth") = keccak256(node("eth"), keccak256("example"))
```

See: [ENSIP-1: Name Hash](https://docs.ens.domains/ensip/1)

### Registry

The core contract that stores name ownership. The Registry tracks which address owns each name and which resolver is set for each name.

See: [Registry Documentation](./registry.md)

### Resolver

A contract that stores and returns records associated with a name—addresses, text records, content hashes, etc. Different resolvers can support different record types.

See: [Resolver Documentation](./resolver.md)

### Name

A human-readable identifier (like `example.eth`) that can be resolved to an address or other records.

### Label

A single component of a name. For `subdomain.example.eth`, the labels are `subdomain`, `example`, and `eth`.

### Subname / Subdomain

A name created under another name. For `sub.example.eth`, `sub` is a subname of `example.eth`. The owner of `example.eth` controls who can create subnames.

### Grace Period

A window of time after a name expires during which the previous owner can still renew without competing with others. Typically 90 days.

### Premium Period

A window after the grace period during which expired names are available but at a declining premium price. This prevents sniping of valuable expired names.

## Technical Terms

### ABI (Application Binary Interface)

A specification for how to encode function calls and decode responses when interacting with smart contracts. Required for calling contract functions from applications.

### Commit-Reveal

A two-step process that prevents front-running. Users first submit a hash of their intended action (commit), wait a required time, then reveal and execute the action.

**Process:**
1. Submit `hash(name + secret)` → wait 60 seconds
2. Submit actual `name` and `secret` → registration proceeds

### Content Hash

A pointer to off-chain content stored on decentralized networks like IPFS or Swarm. Allows names to resolve to websites or other content.

### EIP (Ethereum Improvement Proposal)

A design document proposing new features or standards for Ethereum. Relevant EIPs for this protocol include EIP-137 (ENS) and EIP-721 (NFTs).

### ENSIP (ENS Improvement Proposal)

A design document specific to the Ethereum Name Service protocol. ENSIPs define standards for resolvers, record types, and protocol extensions.

### Gas

The unit of computation on Ethereum. Users pay gas fees to execute transactions. Complex operations (like registration) require more gas.

### Namehash

The algorithm used to convert human-readable names into fixed-length identifiers (nodes). Defined in EIP-137.

```javascript
namehash("") = 0x0
namehash("eth") = keccak256(namehash(""), keccak256("eth"))
namehash("example.eth") = keccak256(namehash("eth"), keccak256("example"))
```

### NFT (Non-Fungible Token)

A unique token on the blockchain. Registered names are represented as NFTs, allowing them to be transferred and traded like other tokens.

### Primary Name / Reverse Record

The name associated with an address for display purposes. When an app shows `vitalik.eth` instead of `0xd8dA...`, it's using the reverse record.

### RPC (Remote Procedure Call)

The interface for communicating with Ethereum nodes. Applications use RPC endpoints to read blockchain data and submit transactions.

### TTL (Time To Live)

How long a piece of data should be cached before refreshing. For resolver records, the TTL indicates how long applications should cache resolved values.

### UUPS (Universal Upgradeable Proxy Standard)

A pattern for creating upgradeable smart contracts. The upgrade logic lives in the implementation contract, making proxies simpler and cheaper.

## Record Types

### Address Record (addr)

The Ethereum address a name resolves to. This is the most common record type.

```javascript
// Set
resolver.setAddr(node, "0x1234...")

// Get  
const address = await resolver.addr(node);
```

### Multicoin Address

Addresses for non-Ethereum blockchains, identified by [SLIP-44](https://github.com/satoshilabs/slips/blob/master/slip-0044.md) coin types.

| Chain | Coin Type |
|-------|-----------|
| Bitcoin | 0 |
| Ethereum | 60 |
| Solana | 501 |

### Text Record

Arbitrary key-value strings. Common keys include:

| Key | Description |
|-----|-------------|
| `avatar` | URL or NFT reference for profile image |
| `url` | Website URL |
| `description` | Profile description |
| `com.twitter` | Twitter handle |
| `com.github` | GitHub username |
| `email` | Email address |

### Content Hash

Pointer to decentralized content:

| Protocol | Prefix |
|----------|--------|
| IPFS | `ipfs://` |
| Swarm | `bzz://` |
| Onion | `onion://` |
| Arweave | `ar://` |

## Roles

### Owner

The address that owns a name. Owners can transfer the name, set resolvers, create subnames, and update records.

### Operator

An address approved to manage a name on behalf of the owner. Operators can do everything owners can except transfer ownership.

### Controller (Role)

A contract authorized to make changes to the registry. The protocol's Controller contract has this role.

### Registrant

The address that registered a name. Initially the same as owner, but ownership can be transferred.

## Units

### Wei

The smallest unit of ETH. 1 ETH = 10^18 wei.

### Gwei

A common unit for gas prices. 1 gwei = 10^9 wei.

### Registration Duration

Measured in seconds. Common durations:

| Duration | Seconds |
|----------|---------|
| 1 year | 31,536,000 |
| 2 years | 63,072,000 |
| 5 years | 157,680,000 |

---

## Template Notes (delete this section when using)

**Building a good glossary:**

- Define terms on first use in documentation, then link to glossary
- Keep definitions concise—link to detailed docs for more info
- Include code examples where helpful
- Organize alphabetically within categories

**Checklist before publishing:**

- [ ] All protocol-specific terms defined
- [ ] Technical terms explained for non-experts
- [ ] Cross-references link to relevant documentation
- [ ] Examples are accurate and tested
