# Security Template

Use this template to document security considerations, audit status, access controls, and known risks for your smart contracts.

---

# Security

This document describes the security model, audit history, and known risks for [Protocol Name].

## Audit Status

### Completed Audits

| Auditor | Date | Scope | Report |
|---------|------|-------|--------|
| Firm Name | January 2025 | Full protocol | [PDF](./audits/audit-report-jan-2025.pdf) |
| Other Firm | December 2024 | Controller, Registry | [PDF](./audits/audit-report-dec-2024.pdf) |

### Audit Findings Summary

| Severity | Found | Resolved | Acknowledged |
|----------|-------|----------|--------------|
| Critical | 0 | 0 | 0 |
| High | 1 | 1 | 0 |
| Medium | 3 | 3 | 0 |
| Low | 5 | 4 | 1 |
| Informational | 8 | 6 | 2 |

### Unresolved Findings

**[LOW-01] Centralized price oracle dependency**

- **Status:** Acknowledged
- **Description:** The protocol relies on a single Chainlink price feed. If the feed fails, registrations would use stale pricing.
- **Mitigation:** We monitor the price feed and can pause registrations if needed. A fallback oracle is planned for v2.

## Bug Bounty Program

We maintain an active bug bounty program to encourage responsible disclosure.

| Severity | Reward |
|----------|--------|
| Critical | Up to $100,000 |
| High | Up to $25,000 |
| Medium | Up to $5,000 |
| Low | Up to $1,000 |

**Scope:** All deployed contracts listed in [Deployments](#deployed-contracts)

**Submission:** security@example.com or via [Immunefi](https://immunefi.com/bounty/example)

**Rules:**
- Do not exploit vulnerabilities on mainnet
- Provide clear reproduction steps
- Allow reasonable time for fixes before disclosure

## Access Control

### Roles and Permissions

| Role | Held By | Capabilities |
|------|---------|--------------|
| Owner | Multisig `0x1234...` | Upgrade contracts, change parameters, pause |
| Controller | Controller contract | Register names, update registry |
| Operator | Approved per-user | Manage names on behalf of owner |

### Owner Capabilities

The protocol owner (a 3-of-5 multisig) can:

- ✅ Upgrade the Controller contract (with 48-hour timelock)
- ✅ Update pricing parameters
- ✅ Pause/unpause registrations
- ✅ Add or remove approved controllers
- ❌ **Cannot** transfer user-owned names
- ❌ **Cannot** modify existing resolver records
- ❌ **Cannot** access user funds

### Multisig Configuration

| Property | Value |
|----------|-------|
| Address | `0x1234567890abcdef...` |
| Type | Gnosis Safe |
| Threshold | 3 of 5 signers |
| Timelock | 48 hours for upgrades |

**Signers:**
- Signer 1: Team Lead (`0xaaa...`)
- Signer 2: CTO (`0xbbb...`)
- Signer 3: Security Lead (`0xccc...`)
- Signer 4: External Advisor (`0xddd...`)
- Signer 5: Community Representative (`0xeee...`)

## Trust Assumptions

Users of this protocol trust that:

1. **The multisig acts honestly** — Signers will not collude to perform malicious upgrades
2. **Timelocks provide exit opportunity** — 48-hour delay allows users to exit before harmful changes take effect
3. **Audits found major issues** — The codebase has been reviewed by reputable auditors
4. **Price oracle is reliable** — Chainlink feeds provide accurate ETH/USD pricing

### What This Protocol Does NOT Protect Against

- Compromised user private keys
- Phishing attacks targeting users
- Frontend compromises (always verify contract addresses)
- Ethereum network failures or reorgs
- All 3+ multisig signers colluding maliciously

## Contract Security Features

### Reentrancy Protection

All state-changing functions that interact with external contracts use the `ReentrancyGuard` modifier:

```solidity
function mint() external payable nonReentrant {
    // Safe from reentrancy
}
```

**Protected functions:**
- `Controller.register()`
- `Controller.renew()`
- `Registry.transfer()`

### Access Control Modifiers

```solidity
modifier onlyOwner() {
    require(msg.sender == owner, "Not owner");
    _;
}

modifier onlyController() {
    require(controllers[msg.sender], "Not controller");
    _;
}

modifier onlyTokenOwner(uint256 tokenId) {
    require(ownerOf(tokenId) == msg.sender, "Not token owner");
    _;
}
```

### Pausability

The Controller can be paused in emergencies:

```solidity
function pause() external onlyOwner {
    _pause();
}

function unpause() external onlyOwner {
    _unpause();
}
```

When paused:
- ❌ New registrations blocked
- ❌ Renewals blocked
- ✅ Transfers still work (users can exit)
- ✅ Resolution still works

### Upgrade Safety

Upgradeable contracts use the UUPS pattern with safety checks:

```solidity
function _authorizeUpgrade(address newImplementation) 
    internal 
    override 
    onlyOwner 
    timelocked 
{
    // Additional validation
}
```

## Known Risks

### Protocol-Specific Risks

**Name Expiration**

Names that are not renewed expire and become available for re-registration. Users may lose their names if they fail to renew.

*Mitigation:* We send reminder emails (if email is set) and provide a grace period after expiration.

**Front-Running**

Without the commit-reveal scheme, name registrations could be front-run by MEV bots.

*Mitigation:* The two-step commit-reveal process prevents front-running of specific name registrations.

**Resolver Trust**

Users can set any resolver for their names. Malicious resolvers could return incorrect data.

*Mitigation:* We provide a default trusted resolver. Users who set custom resolvers do so at their own risk.

### Ethereum-Level Risks

**Chain Reorgs**

Deep chain reorganizations could revert recent registrations.

*Mitigation:* Wait for sufficient confirmations (12+ blocks) before considering registrations final.

**Gas Price Spikes**

High gas prices may make registrations expensive.

*Mitigation:* Users can wait for lower gas or use L2 deployments (coming soon).

## Incident Response

### Severity Levels

| Level | Description | Response Time |
|-------|-------------|---------------|
| Critical | Active exploit, funds at risk | Immediate |
| High | Vulnerability discovered, no active exploit | 24 hours |
| Medium | Non-critical bug affecting functionality | 72 hours |
| Low | Minor issue, no security impact | 1 week |

### Response Procedures

**Critical Incident:**

1. **Pause** affected contracts immediately
2. **Assess** scope and impact
3. **Communicate** via official channels (Twitter, Discord)
4. **Fix** and deploy patch
5. **Postmortem** within 7 days

### Contact

- **Security Email:** security@example.com
- **Emergency Telegram:** @example_security
- **PGP Key:** [pubkey.asc](./pubkey.asc)

## Deployed Contracts

Verify you are interacting with official contracts:

### Mainnet

| Contract | Address | Verified |
|----------|---------|----------|
| Registry | `0x1234...abcd` | [Etherscan](https://etherscan.io/address/0x1234) |
| Controller | `0x2345...bcde` | [Etherscan](https://etherscan.io/address/0x2345) |
| Resolver | `0x3456...cdef` | [Etherscan](https://etherscan.io/address/0x3456) |

### How to Verify

Always verify contract addresses through multiple sources:

1. Official documentation (this page)
2. Official app UI
3. Etherscan verification status
4. Cross-reference with announcements

## Security Resources

- [Audit Reports](./audits/)
- [Bug Bounty Program](https://immunefi.com/bounty/example)
- [Security Contact](mailto:security@example.com)
- [Incident History](./incidents/)

---

## Template Notes (delete this section when using)

**Customizing this template:**

- Update with your actual audit reports and findings
- Configure bug bounty amounts for your budget
- Document your actual multisig setup
- Add protocol-specific risks

**Checklist before publishing:**

- [ ] All audits linked with findings summary
- [ ] Bug bounty program details accurate
- [ ] Access control documentation complete
- [ ] Known risks honestly documented
- [ ] Incident response contacts current
- [ ] All contract addresses verified
