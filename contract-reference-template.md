# Contract Reference Template

Use this template to document a single smart contract. It covers the contract's purpose, interface, and usage.

---

# ContractName

Brief description of what this contract does and its role in the protocol.

## Overview

| Property | Value |
|----------|-------|
| Contract | `ContractName.sol` |
| Solidity Version | `^0.8.19` |
| License | MIT |
| Deployed Address (Mainnet) | `0x1234...abcd` |
| Deployed Address (Sepolia) | `0xabcd...5678` |

### Inheritance

```
ContractName
├── Ownable
├── ReentrancyGuard
└── IERC721
```

### Key Features

- Feature one: brief description
- Feature two: brief description
- Feature three: brief description

## State Variables

### Public Variables

| Variable | Type | Description |
|----------|------|-------------|
| `owner` | `address` | The contract owner with admin privileges |
| `totalSupply` | `uint256` | Total number of tokens minted |
| `baseURI` | `string` | Base URI for token metadata |

### Mappings

| Mapping | Type | Description |
|---------|------|-------------|
| `balanceOf` | `mapping(address => uint256)` | Number of tokens owned by each address |
| `tokenApprovals` | `mapping(uint256 => address)` | Approved operator for each token |

## Functions

### Read Functions

#### `balanceOf`

Returns the number of tokens owned by an address.

```solidity
function balanceOf(address owner) external view returns (uint256)
```

**Parameters:**

| Name | Type | Description |
|------|------|-------------|
| `owner` | `address` | The address to query |

**Returns:**

| Type | Description |
|------|-------------|
| `uint256` | Number of tokens owned by the address |

**Example:**

```javascript
const balance = await contract.balanceOf("0x1234...abcd");
console.log(`Balance: ${balance}`);
```

---

#### `ownerOf`

Returns the owner of a specific token.

```solidity
function ownerOf(uint256 tokenId) external view returns (address)
```

**Parameters:**

| Name | Type | Description |
|------|------|-------------|
| `tokenId` | `uint256` | The token ID to query |

**Returns:**

| Type | Description |
|------|-------------|
| `address` | Address of the token owner |

**Reverts:**

- `TokenDoesNotExist()` — if the token has not been minted

---

### Write Functions

#### `mint`

Mints a new token to the specified address.

```solidity
function mint(address to) external payable returns (uint256)
```

**Parameters:**

| Name | Type | Description |
|------|------|-------------|
| `to` | `address` | Recipient address for the new token |

**Returns:**

| Type | Description |
|------|-------------|
| `uint256` | The ID of the newly minted token |

**Requirements:**

- `msg.value` must be at least `mintPrice`
- `to` cannot be the zero address
- Total supply must not exceed `maxSupply`

**Reverts:**

- `InsufficientPayment()` — if `msg.value` is below `mintPrice`
- `MaxSupplyReached()` — if minting would exceed `maxSupply`
- `InvalidRecipient()` — if `to` is the zero address

**Example:**

```javascript
const tx = await contract.mint(recipientAddress, {
  value: ethers.parseEther("0.08")
});
const receipt = await tx.wait();

// Get the minted token ID from the Transfer event
const event = receipt.logs.find(log => log.fragment.name === "Transfer");
const tokenId = event.args.tokenId;
```

---

#### `transfer`

Transfers a token from one address to another.

```solidity
function transfer(address from, address to, uint256 tokenId) external
```

**Parameters:**

| Name | Type | Description |
|------|------|-------------|
| `from` | `address` | Current owner of the token |
| `to` | `address` | Recipient address |
| `tokenId` | `uint256` | ID of the token to transfer |

**Requirements:**

- Caller must be the owner or an approved operator
- `from` must be the current owner
- `to` cannot be the zero address

**Reverts:**

- `NotAuthorized()` — if caller is not owner or approved
- `InvalidRecipient()` — if `to` is the zero address

---

### Admin Functions

#### `setBaseURI`

Updates the base URI for token metadata. Restricted to contract owner.

```solidity
function setBaseURI(string calldata newBaseURI) external onlyOwner
```

**Parameters:**

| Name | Type | Description |
|------|------|-------------|
| `newBaseURI` | `string` | New base URI for metadata |

**Access Control:** Owner only

---

## Events

### `Transfer`

Emitted when a token is transferred.

```solidity
event Transfer(address indexed from, address indexed to, uint256 indexed tokenId)
```

| Parameter | Type | Indexed | Description |
|-----------|------|---------|-------------|
| `from` | `address` | Yes | Previous owner (zero address for mints) |
| `to` | `address` | Yes | New owner (zero address for burns) |
| `tokenId` | `uint256` | Yes | ID of the transferred token |

**Listening for events:**

```javascript
contract.on("Transfer", (from, to, tokenId) => {
  console.log(`Token ${tokenId} transferred from ${from} to ${to}`);
});
```

---

### `Approval`

Emitted when an approval is granted or revoked.

```solidity
event Approval(address indexed owner, address indexed approved, uint256 indexed tokenId)
```

| Parameter | Type | Indexed | Description |
|-----------|------|---------|-------------|
| `owner` | `address` | Yes | Token owner granting approval |
| `approved` | `address` | Yes | Address being approved (zero to revoke) |
| `tokenId` | `uint256` | Yes | Token ID being approved |

---

## Errors

| Error | Parameters | Description |
|-------|------------|-------------|
| `NotAuthorized()` | — | Caller is not authorized for this action |
| `TokenDoesNotExist()` | — | The specified token has not been minted |
| `InvalidRecipient()` | — | Recipient address is invalid (zero address) |
| `InsufficientPayment()` | — | ETH sent is less than required |
| `MaxSupplyReached()` | — | Minting would exceed maximum supply |

## Access Control

| Role | Address/Mechanism | Capabilities |
|------|-------------------|--------------|
| Owner | `0x1234...abcd` | Set base URI, withdraw funds, pause contract |
| Minter | Any address | Mint tokens (with payment) |
| Token Holder | Token owners | Transfer, approve, burn owned tokens |

## Security Considerations

- This contract uses `ReentrancyGuard` to prevent reentrancy attacks on mint and transfer functions
- Admin functions are restricted to the contract owner via `Ownable`
- See [Security Documentation](./security.md) for audit status and known risks

## Related Contracts

- [RegistryContract](./registry.md) — Manages contract registration
- [ResolverContract](./resolver.md) — Handles name resolution

---

## Template Notes (delete this section when using)

**Customizing this template:**

- Add or remove function sections based on your contract's interface
- Include gas estimates if relevant for expensive operations
- Add diagrams for complex state transitions
- Link to Etherscan for deployed contracts

**Checklist before publishing:**

- [ ] All public/external functions documented
- [ ] All events documented
- [ ] All custom errors documented
- [ ] Deployed addresses are correct
- [ ] Code examples tested against actual contract
- [ ] Access control clearly explained
