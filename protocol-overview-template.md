# Protocol Overview Template

Use this template to document the high-level architecture of a multi-contract system. This helps developers understand how contracts interact before diving into individual contract details.

---

# Protocol Name

A brief, one-paragraph description of what the protocol does and the problem it solves.

## Architecture Overview

Describe the high-level design of the protocol in 2-3 paragraphs. Explain the core concepts and how the contracts work together.

### Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                        User/dApp                            │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                      Controller                             │
│                   (Entry Point)                             │
└──────────┬─────────────────────────────────┬────────────────┘
           │                                 │
           ▼                                 ▼
┌─────────────────────┐         ┌─────────────────────────────┐
│      Registry       │         │         Resolver            │
│  (Name Storage)     │◄───────►│    (Data Resolution)        │
└─────────────────────┘         └─────────────────────────────┘
           │
           ▼
┌─────────────────────┐
│    PriceOracle      │
│  (Fee Calculation)  │
└─────────────────────┘
```

## Core Concepts

### Concept One

Explain a fundamental concept that users need to understand. For example, if documenting ENS, you might explain what a "node" is and how names are hashed.

### Concept Two

Another key concept. Keep explanations concise but complete.

### Concept Three

Continue with additional concepts as needed.

## Contract Overview

| Contract | Purpose | Mainnet Address |
|----------|---------|-----------------|
| Controller | Entry point for user interactions | `0x1234...abcd` |
| Registry | Stores name ownership records | `0x2345...bcde` |
| Resolver | Maps names to addresses and records | `0x3456...cdef` |
| PriceOracle | Calculates registration fees | `0x4567...def0` |

### Controller

The Controller is the main entry point for users. It handles registration, renewal, and coordinates interactions between other contracts.

**Key responsibilities:**
- Validate registration requests
- Calculate and collect fees
- Coordinate registry and resolver updates

**Interacts with:** Registry, Resolver, PriceOracle

[Full Controller Documentation →](./controller.md)

### Registry

The Registry maintains the authoritative record of name ownership. It tracks who owns each name and manages permissions.

**Key responsibilities:**
- Store name ownership records
- Manage operator approvals
- Emit transfer events

**Interacts with:** Controller

[Full Registry Documentation →](./registry.md)

### Resolver

The Resolver stores and returns records associated with names—addresses, content hashes, text records, and more.

**Key responsibilities:**
- Store address records
- Store text records
- Return records for lookups

**Interacts with:** Controller, external callers

[Full Resolver Documentation →](./resolver.md)

### PriceOracle

The PriceOracle determines registration and renewal fees based on name length, duration, and current ETH price.

**Key responsibilities:**
- Calculate registration fees
- Provide price quotes
- Integrate with external price feeds

**Interacts with:** Controller

[Full PriceOracle Documentation →](./price-oracle.md)

## User Flows

### Registering a Name

```
User                    Controller              Registry            Resolver
  │                          │                      │                   │
  │  1. register(name)       │                      │                   │
  │  ───────────────────────►│                      │                   │
  │                          │                      │                   │
  │                          │  2. checkAvailable() │                   │
  │                          │  ───────────────────►│                   │
  │                          │                      │                   │
  │                          │  3. available=true   │                   │
  │                          │  ◄───────────────────│                   │
  │                          │                      │                   │
  │                          │  4. register()       │                   │
  │                          │  ───────────────────►│                   │
  │                          │                      │                   │
  │                          │  5. setResolver()    │                   │
  │                          │  ────────────────────────────────────────►
  │                          │                      │                   │
  │  6. success + tokenId    │                      │                   │
  │  ◄───────────────────────│                      │                   │
```

**Steps:**
1. User calls `register()` on Controller with name and payment
2. Controller checks name availability in Registry
3. Registry confirms the name is available
4. Controller registers the name in Registry
5. Controller sets up default Resolver records
6. User receives confirmation and token ID

### Resolving a Name

```
User/dApp                Registry              Resolver
    │                        │                     │
    │  1. resolver(node)     │                     │
    │  ─────────────────────►│                     │
    │                        │                     │
    │  2. resolverAddress    │                     │
    │  ◄─────────────────────│                     │
    │                        │                     │
    │  3. addr(node)         │                     │
    │  ────────────────────────────────────────────►
    │                        │                     │
    │  4. 0x1234...abcd      │                     │
    │  ◄────────────────────────────────────────────
```

### Transferring Ownership

Describe another common user flow.

## Data Flow

### Name Registration Data

```
Input                           Stored                          Output
─────                           ──────                          ──────
name: "example"          ───►   Registry:                 ───►  tokenId: 12345
owner: 0xUser                     owner[node] = 0xUser          node: 0xabc...
duration: 1 year                  expiry[node] = timestamp
                                
                                Resolver:
                                  addr[node] = 0xUser
```

## Trust Model

### What the Protocol Controls

- Name registration and ownership records
- Resolver record storage
- Fee collection and distribution

### What Users Control

- Their registered names (as NFTs)
- Records associated with their names
- Resolver selection for their names

### Trust Assumptions

- The protocol owner can upgrade certain contracts (list which ones)
- Price oracle relies on [Chainlink/other] for ETH price data
- Users trust that contract upgrades won't steal their names

## Upgrade Mechanisms

| Contract | Upgradeable | Mechanism | Timelock |
|----------|-------------|-----------|----------|
| Controller | Yes | UUPS Proxy | 48 hours |
| Registry | No | Immutable | — |
| Resolver | Yes | New deployment | — |
| PriceOracle | Yes | Owner can replace | 24 hours |

### Upgrade Process

Describe how upgrades work, who can initiate them, and what safeguards exist.

## Network Deployments

### Mainnet

| Contract | Address | Deployment Date |
|----------|---------|-----------------|
| Controller | `0x1234...abcd` | 2024-01-15 |
| Registry | `0x2345...bcde` | 2024-01-15 |
| Resolver | `0x3456...cdef` | 2024-01-15 |

### Sepolia (Testnet)

| Contract | Address | Notes |
|----------|---------|-------|
| Controller | `0xaaaa...1111` | Mirrors mainnet |
| Registry | `0xbbbb...2222` | Mirrors mainnet |
| Resolver | `0xcccc...3333` | Mirrors mainnet |

## Further Reading

- [Integration Guide](./integration-guide.md) — How to integrate with this protocol
- [Security Documentation](./security.md) — Audit reports and known risks
- [Contract Reference](./contracts/) — Detailed documentation for each contract
- [GitHub Repository](https://github.com/example/protocol) — Source code

---

## Template Notes (delete this section when using)

**Customizing this template:**

- Adjust the architecture diagram to match your protocol
- Add or remove contracts from the overview
- Include additional user flows relevant to your protocol
- Update trust model based on your governance structure

**Checklist before publishing:**

- [ ] Architecture diagram is accurate
- [ ] All contracts listed with correct addresses
- [ ] User flows match actual contract behavior
- [ ] Trust assumptions are honest and complete
- [ ] Upgrade mechanisms clearly documented
