# Integration Guide Template

Use this template to help external developers integrate with your smart contracts. Focus on practical implementation rather than exhaustive reference documentation.

---

# Integrating with [Protocol Name]

This guide walks you through integrating [Protocol Name] into your application. By the end, you'll be able to [primary use case, e.g., "resolve names to addresses" or "mint tokens for your users"].

## Before You Start

### Prerequisites

- Basic understanding of Ethereum and smart contract interactions
- A web3 library installed ([ethers.js](https://docs.ethers.org/) or [viem](https://viem.sh/))
- An Ethereum RPC endpoint (Infura, Alchemy, or your own node)

### Contract Addresses

| Network | Contract | Address |
|---------|----------|---------|
| Mainnet | Controller | `0x1234...abcd` |
| Mainnet | Registry | `0x2345...bcde` |
| Sepolia | Controller | `0xaaaa...1111` |
| Sepolia | Registry | `0xbbbb...2222` |

### Installation

Install the official SDK (if available):

```bash
npm install @protocol/sdk
```

Or use the contract ABIs directly:

```bash
npm install @protocol/contracts
```

## Quick Start

### Option 1: Using the SDK

```javascript
import { ProtocolClient } from '@protocol/sdk';

const client = new ProtocolClient({
  provider: window.ethereum,
  network: 'mainnet'
});

// Example: Resolve a name
const address = await client.resolve('example.eth');
console.log(address); // 0x1234...abcd
```

### Option 2: Using ethers.js Directly

```javascript
import { ethers } from 'ethers';
import { RegistryABI } from '@protocol/contracts';

const provider = new ethers.BrowserProvider(window.ethereum);
const registry = new ethers.Contract(
  '0x2345...bcde', // Registry address
  RegistryABI,
  provider
);

// Example: Check name ownership
const owner = await registry.ownerOf(nameHash('example.eth'));
console.log(owner);
```

## Common Integration Patterns

### Pattern 1: Resolving Names to Addresses

The most common integration—convert a human-readable name to an Ethereum address.

```javascript
import { ethers } from 'ethers';
import { RegistryABI, ResolverABI } from '@protocol/contracts';

async function resolveName(name) {
  const provider = new ethers.JsonRpcProvider(process.env.RPC_URL);
  const node = ethers.namehash(name);
  
  // Step 1: Get the resolver for this name
  const registry = new ethers.Contract(REGISTRY_ADDRESS, RegistryABI, provider);
  const resolverAddress = await registry.resolver(node);
  
  if (resolverAddress === ethers.ZeroAddress) {
    throw new Error('Name not registered or no resolver set');
  }
  
  // Step 2: Query the resolver for the address
  const resolver = new ethers.Contract(resolverAddress, ResolverABI, provider);
  const address = await resolver.addr(node);
  
  return address;
}

// Usage
const address = await resolveName('vitalik.eth');
```

**Important considerations:**
- Always check if the resolver exists before querying
- Handle the case where a name is registered but has no address record
- Consider caching results to reduce RPC calls

### Pattern 2: Reverse Resolution (Address to Name)

Look up the primary name associated with an address.

```javascript
async function lookupAddress(address) {
  const provider = new ethers.JsonRpcProvider(process.env.RPC_URL);
  
  // Construct the reverse node
  const reverseNode = ethers.namehash(
    `${address.slice(2).toLowerCase()}.addr.reverse`
  );
  
  const registry = new ethers.Contract(REGISTRY_ADDRESS, RegistryABI, provider);
  const resolverAddress = await registry.resolver(reverseNode);
  
  if (resolverAddress === ethers.ZeroAddress) {
    return null; // No reverse record set
  }
  
  const resolver = new ethers.Contract(resolverAddress, ResolverABI, provider);
  const name = await resolver.name(reverseNode);
  
  // Important: Verify forward resolution matches
  const forwardAddress = await resolveName(name);
  if (forwardAddress.toLowerCase() !== address.toLowerCase()) {
    return null; // Forward/reverse mismatch
  }
  
  return name;
}
```

**Security note:** Always verify that forward resolution of the returned name matches the original address. This prevents spoofing attacks.

### Pattern 3: Checking Name Availability

Before allowing users to register, check if a name is available.

```javascript
async function isNameAvailable(name) {
  const provider = new ethers.JsonRpcProvider(process.env.RPC_URL);
  const controller = new ethers.Contract(
    CONTROLLER_ADDRESS,
    ControllerABI,
    provider
  );
  
  return await controller.available(name);
}
```

### Pattern 4: Registering a Name

Allow users to register names through your application.

```javascript
async function registerName(name, owner, duration, signer) {
  const controller = new ethers.Contract(
    CONTROLLER_ADDRESS,
    ControllerABI,
    signer
  );
  
  // Step 1: Get the registration price
  const price = await controller.rentPrice(name, duration);
  
  // Step 2: Generate a commitment (for front-running protection)
  const secret = ethers.randomBytes(32);
  const commitment = await controller.makeCommitment(
    name,
    owner,
    duration,
    secret,
    resolverAddress,
    [],  // Initial records
    false,
    0
  );
  
  // Step 3: Submit commitment
  const commitTx = await controller.commit(commitment);
  await commitTx.wait();
  
  // Step 4: Wait for commitment to mature (typically 60 seconds)
  console.log('Waiting for commitment to mature...');
  await new Promise(resolve => setTimeout(resolve, 60000));
  
  // Step 5: Register the name
  const registerTx = await controller.register(
    name,
    owner,
    duration,
    secret,
    resolverAddress,
    [],
    false,
    0,
    { value: price }
  );
  
  const receipt = await registerTx.wait();
  return receipt;
}
```

**Important:** The commit-reveal scheme prevents front-running. Always implement both steps.

## Reading Records

### Address Records

```javascript
// Get the ETH address
const ethAddress = await resolver.addr(node);

// Get address for other chains (using ENSIP-9 coin types)
const btcAddress = await resolver.addr(node, 0);    // Bitcoin
const solAddress = await resolver.addr(node, 501);  // Solana
```

### Text Records

```javascript
// Common text record keys
const twitter = await resolver.text(node, 'com.twitter');
const github = await resolver.text(node, 'com.github');
const email = await resolver.text(node, 'email');
const url = await resolver.text(node, 'url');
const avatar = await resolver.text(node, 'avatar');
const description = await resolver.text(node, 'description');
```

### Content Hash

```javascript
const contentHash = await resolver.contenthash(node);
// Returns IPFS, Swarm, or other content addresses
// Decode using @ensdomains/content-hash library
```

## Writing Records

Users can update their own records through the resolver.

```javascript
async function setAddress(name, newAddress, signer) {
  const node = ethers.namehash(name);
  
  // Get the resolver (must be connected to a signer)
  const registry = new ethers.Contract(REGISTRY_ADDRESS, RegistryABI, signer);
  const resolverAddress = await registry.resolver(node);
  const resolver = new ethers.Contract(resolverAddress, ResolverABI, signer);
  
  const tx = await resolver.setAddr(node, newAddress);
  await tx.wait();
}

async function setTextRecord(name, key, value, signer) {
  const node = ethers.namehash(name);
  const resolver = await getResolver(name, signer);
  
  const tx = await resolver.setText(node, key, value);
  await tx.wait();
}
```

## Batch Operations

For efficiency, use multicall to batch multiple reads or writes.

```javascript
import { Contract } from 'ethers';

async function batchResolve(names) {
  const provider = new ethers.JsonRpcProvider(process.env.RPC_URL);
  
  // Use a multicall contract or library
  const calls = names.map(name => ({
    target: RESOLVER_ADDRESS,
    callData: resolverInterface.encodeFunctionData('addr', [
      ethers.namehash(name)
    ])
  }));
  
  const results = await multicall.aggregate(calls);
  return results.map(r => resolverInterface.decodeFunctionResult('addr', r)[0]);
}
```

## Error Handling

### Common Errors

| Error | Cause | Solution |
|-------|-------|----------|
| `Name not registered` | Name doesn't exist in registry | Check availability before resolving |
| `No resolver set` | Name exists but has no resolver | Guide user to set a resolver |
| `Insufficient funds` | Not enough ETH for registration | Check price and user balance first |
| `Commitment too new` | Tried to register before commitment matured | Wait the required time (usually 60s) |
| `Name not available` | Name already registered | Check availability first |

### Defensive Coding

```javascript
async function safeResolve(name) {
  try {
    // Validate input
    if (!name || typeof name !== 'string') {
      throw new Error('Invalid name');
    }
    
    const address = await resolveName(name);
    
    // Validate output
    if (!ethers.isAddress(address)) {
      throw new Error('Invalid address returned');
    }
    
    return { success: true, address };
  } catch (error) {
    return { 
      success: false, 
      error: error.message,
      address: null 
    };
  }
}
```

## Testing Your Integration

### Local Development

Use a local fork of mainnet for testing:

```javascript
// hardhat.config.js
module.exports = {
  networks: {
    hardhat: {
      forking: {
        url: process.env.MAINNET_RPC_URL,
        blockNumber: 18500000 // Pin to a specific block
      }
    }
  }
};
```

### Testnet

Use Sepolia for testing with real (test) transactions:

```javascript
const provider = new ethers.JsonRpcProvider(
  'https://sepolia.infura.io/v3/YOUR_KEY'
);

// Testnet contract addresses
const REGISTRY_ADDRESS = '0xbbbb...2222';
```

## Production Checklist

Before going live, verify:

- [ ] Using mainnet contract addresses
- [ ] Error handling covers all edge cases
- [ ] Forward/reverse resolution verified (security)
- [ ] RPC endpoint is reliable and rate limits are handled
- [ ] User-facing errors are clear and actionable
- [ ] Caching implemented where appropriate
- [ ] Gas estimates shown before transactions

## Getting Help

- [Documentation](https://docs.example.com)
- [Discord](https://discord.example.com)
- [GitHub Issues](https://github.com/example/protocol/issues)

---

## Template Notes (delete this section when using)

**Customizing this template:**

- Replace generic examples with your actual contract interfaces
- Add integration patterns specific to your protocol
- Include SDK documentation if you have one
- Add framework-specific guides (React hooks, etc.) if relevant

**Checklist before publishing:**

- [ ] All code examples tested and working
- [ ] Contract addresses correct for each network
- [ ] Error handling examples match actual contract errors
- [ ] Security considerations clearly documented
