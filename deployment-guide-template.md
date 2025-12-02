# Deployment Guide Template

Use this template to document how to deploy your smart contracts. Clear deployment docs help teams deploy consistently and reduce errors.

---

# Deployment Guide

This guide covers deploying [Protocol Name] contracts to Ethereum networks.

## Prerequisites

### Required Tools

- [Node.js](https://nodejs.org/) v18 or later
- [Foundry](https://book.getfoundry.sh/) or [Hardhat](https://hardhat.org/)
- Git

### Required Accounts & Keys

- Deployer wallet with sufficient ETH for gas
- RPC endpoint for target network (Infura, Alchemy, etc.)
- Etherscan API key (for verification)

### Environment Setup

Clone the repository and install dependencies:

```bash
git clone https://github.com/example/protocol.git
cd protocol
npm install
```

Create a `.env` file:

```bash
# .env
PRIVATE_KEY=your_deployer_private_key
MAINNET_RPC_URL=https://mainnet.infura.io/v3/YOUR_KEY
SEPOLIA_RPC_URL=https://sepolia.infura.io/v3/YOUR_KEY
ETHERSCAN_API_KEY=your_etherscan_api_key
```

> ⚠️ **Never commit `.env` files to version control.** Add `.env` to your `.gitignore`.

## Deployment Overview

### Contract Deployment Order

Contracts must be deployed in a specific order due to dependencies:

```
1. Registry (no dependencies)
      │
      ▼
2. PriceOracle (no dependencies)
      │
      ▼
3. Resolver (depends on Registry)
      │
      ▼
4. Controller (depends on Registry, PriceOracle, Resolver)
      │
      ▼
5. Configuration (set permissions, link contracts)
```

### Deployment Checklist

Before deploying:

- [ ] Contracts audited and issues resolved
- [ ] Deployer wallet funded with ETH
- [ ] RPC endpoints tested and working
- [ ] Constructor parameters finalized
- [ ] Multisig addresses confirmed (if applicable)
- [ ] Team notified of deployment window

## Network-Specific Instructions

### Deploying to Sepolia (Testnet)

Use testnet for initial deployment testing.

```bash
# Run the deployment script
npx hardhat run scripts/deploy.js --network sepolia
```

Or with Foundry:

```bash
forge script script/Deploy.s.sol:DeployScript \
  --rpc-url $SEPOLIA_RPC_URL \
  --broadcast \
  --verify
```

### Deploying to Mainnet

Mainnet deployments require extra caution.

**Pre-deployment:**

1. Test deployment on Sepolia first
2. Verify all constructor parameters
3. Ensure sufficient ETH (estimate: ~0.5 ETH for full deployment)
4. Schedule deployment during low gas periods

**Deploy:**

```bash
# Dry run first (no broadcast)
forge script script/Deploy.s.sol:DeployScript \
  --rpc-url $MAINNET_RPC_URL

# If dry run succeeds, broadcast
forge script script/Deploy.s.sol:DeployScript \
  --rpc-url $MAINNET_RPC_URL \
  --broadcast \
  --verify \
  --slow  # Wait for confirmations between transactions
```

## Step-by-Step Deployment

### Step 1: Deploy Registry

The Registry has no dependencies and should be deployed first.

**Constructor parameters:**

| Parameter | Type | Description | Example |
|-----------|------|-------------|---------|
| `owner` | `address` | Initial contract owner | `0x1234...abcd` |

**Deployment:**

```solidity
// script/01_DeployRegistry.s.sol
pragma solidity ^0.8.19;

import "forge-std/Script.sol";
import "../src/Registry.sol";

contract DeployRegistry is Script {
    function run() external {
        uint256 deployerPrivateKey = vm.envUint("PRIVATE_KEY");
        address owner = vm.envAddress("OWNER_ADDRESS");
        
        vm.startBroadcast(deployerPrivateKey);
        
        Registry registry = new Registry(owner);
        
        console.log("Registry deployed to:", address(registry));
        
        vm.stopBroadcast();
    }
}
```

**Verification:**

```bash
forge verify-contract \
  --chain-id 1 \
  --constructor-args $(cast abi-encode "constructor(address)" 0x1234...abcd) \
  0xDEPLOYED_ADDRESS \
  src/Registry.sol:Registry
```

### Step 2: Deploy PriceOracle

**Constructor parameters:**

| Parameter | Type | Description | Example |
|-----------|------|-------------|---------|
| `priceFeed` | `address` | Chainlink ETH/USD feed | `0x5f4eC3...` |
| `basePrices` | `uint256[]` | Base prices by name length | `[500, 100, 25, 5]` |

**Deployment:**

```solidity
// script/02_DeployPriceOracle.s.sol
contract DeployPriceOracle is Script {
    function run() external {
        uint256 deployerPrivateKey = vm.envUint("PRIVATE_KEY");
        
        // Chainlink ETH/USD on mainnet
        address priceFeed = 0x5f4eC3Df9cbd43714FE2740f5E3616155c5b8419;
        
        // Base prices in USD (with 18 decimals)
        uint256[] memory basePrices = new uint256[](4);
        basePrices[0] = 500 ether;  // 5+ char names
        basePrices[1] = 100 ether;  // 4 char names
        basePrices[2] = 25 ether;   // 3 char names
        basePrices[3] = 5 ether;    // Special pricing
        
        vm.startBroadcast(deployerPrivateKey);
        
        PriceOracle oracle = new PriceOracle(priceFeed, basePrices);
        
        console.log("PriceOracle deployed to:", address(oracle));
        
        vm.stopBroadcast();
    }
}
```

### Step 3: Deploy Resolver

**Constructor parameters:**

| Parameter | Type | Description | Example |
|-----------|------|-------------|---------|
| `registry` | `address` | Registry contract address | From Step 1 |

### Step 4: Deploy Controller

**Constructor parameters:**

| Parameter | Type | Description | Example |
|-----------|------|-------------|---------|
| `registry` | `address` | Registry address | From Step 1 |
| `priceOracle` | `address` | PriceOracle address | From Step 2 |
| `resolver` | `address` | Default resolver | From Step 3 |
| `minCommitAge` | `uint256` | Commitment wait time | `60` (seconds) |
| `maxCommitAge` | `uint256` | Commitment expiry | `86400` (24 hours) |

### Step 5: Configure Permissions

After all contracts are deployed, configure permissions:

```solidity
// script/05_Configure.s.sol
contract Configure is Script {
    function run() external {
        uint256 deployerPrivateKey = vm.envUint("PRIVATE_KEY");
        
        Registry registry = Registry(REGISTRY_ADDRESS);
        
        vm.startBroadcast(deployerPrivateKey);
        
        // Grant Controller permission to register names
        registry.setController(CONTROLLER_ADDRESS, true);
        
        // Transfer ownership to multisig
        registry.transferOwnership(MULTISIG_ADDRESS);
        
        vm.stopBroadcast();
    }
}
```

## Verification

### Verify on Etherscan

Verify each contract after deployment:

```bash
# Using Foundry
forge verify-contract \
  --chain-id 1 \
  --constructor-args $(cast abi-encode "constructor(address,address)" arg1 arg2) \
  DEPLOYED_ADDRESS \
  src/Contract.sol:ContractName

# Using Hardhat
npx hardhat verify --network mainnet DEPLOYED_ADDRESS "arg1" "arg2"
```

### Post-Deployment Verification

After deployment, verify everything works:

```bash
# Run integration tests against deployed contracts
DEPLOYED_REGISTRY=0x... npx hardhat test test/integration/ --network sepolia
```

**Manual checks:**

- [ ] All contracts verified on Etherscan
- [ ] Ownership transferred to correct addresses
- [ ] Permissions configured correctly
- [ ] Test registration works end-to-end
- [ ] Events emitting correctly

## Deployed Addresses

Record all deployed addresses:

### Mainnet

| Contract | Address | Tx Hash | Block |
|----------|---------|---------|-------|
| Registry | `0x...` | `0x...` | 12345678 |
| PriceOracle | `0x...` | `0x...` | 12345679 |
| Resolver | `0x...` | `0x...` | 12345680 |
| Controller | `0x...` | `0x...` | 12345681 |

### Sepolia

| Contract | Address | Tx Hash | Block |
|----------|---------|---------|-------|
| Registry | `0x...` | `0x...` | 1234567 |
| ... | ... | ... | ... |

## Troubleshooting

### Transaction Reverted

- Check constructor parameters are correct
- Verify deployer has sufficient ETH
- Check gas limit isn't too low

### Verification Failed

- Ensure compiler version matches exactly
- Check constructor arguments are correctly encoded
- Verify optimizer settings match deployment

### Permission Denied

- Confirm you're using the correct deployer account
- Check if ownership was already transferred

## Rollback Procedures

If something goes wrong:

1. **Do not panic** — most issues can be resolved
2. **Pause affected contracts** (if pause functionality exists)
3. **Document the issue** — block number, transaction hash, error
4. **Assess impact** — are user funds at risk?
5. **Deploy fix** — either patch or redeploy

For immutable contracts, a new deployment may be required. Update all references and notify integrators.

---

## Template Notes (delete this section when using)

**Customizing this template:**

- Add your actual deployment scripts
- Include network-specific parameters (chain IDs, price feeds)
- Add your team's deployment approval process
- Include runbook for common deployment issues

**Checklist before publishing:**

- [ ] Deployment order matches actual dependencies
- [ ] Constructor parameters documented for each contract
- [ ] Verification commands tested
- [ ] Post-deployment checks are comprehensive
