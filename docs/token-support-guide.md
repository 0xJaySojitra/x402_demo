# Token & Facilitator Guide

This guide explains which tokens are supported by x402, how to configure custom tokens, and how to set up different facilitator modes.

## Supported Tokens

### Built-in Token Support

The x402 protocol primarily uses USDC (USD Coin) across multiple blockchain networks:

| Network | Token | Contract Address | Chain ID | Use Case |
|---------|-------|------------------|----------|----------|
| **Base** | USDC | `0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913` | 8453 | Production (low fees) |
| **Base Sepolia** | USDC | `0x036CbD53842c5426634e7929541eC2318f3dCF7e` | 84532 | Testing |
| **Polygon** | USDC | `0x3c499c542cEF5E3811e1192ce70d8cC03d5c3359` | 137 | Production (very low fees) |
| **Polygon Amoy** | USDC | `0x41E94Eb019C0762f9Bfcf9Fb1E58725BfB0e7582` | 80002 | Testing |
| **Avalanche** | USDC | `0xB97EF9Ef8734C71904D8002F8b6Bc66Dd9c48a6E` | 43114 | Production |
| **Avalanche Fuji** | USDC | `0x5425890298aed601595a70AB815c96711a31Bc65` | 43113 | Testing |
| **IoTeX** | Bridged USDC | `0xcdf79194c6c285077a58da47641d4dbe51f63542` | 4689 | Production |
| **Sei** | USDC | `0xe15fc38f6d8c56af07bbcbe3baf5708a2bf42392` | 1329 | Production |
| **Sei Testnet** | USDC | `0x4fcf1784b31630811181f670aea7a7bef803eaed` | 1328 | Testing |
| **Peaq** | USDC | `0xbbA60da06c2c5424f03f7434542280FCAd453d10` | 3338 | Production |
| **Solana** | USDC | `EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v` | - | Production |
| **Solana Devnet** | USDC | `4zMMC9srt5Ri5X14GAgXhaHii3GnPAEERYPJgZJDncDU` | - | Testing |

### Token Requirements

For a token to work with x402, it must implement **EIP-3009** (transferWithAuthorization):

```solidity
function transferWithAuthorization(
    address from,
    address to, 
    uint256 value,
    uint256 validAfter,
    uint256 validBefore,
    bytes32 nonce,
    uint8 v,
    bytes32 r,
    bytes32 s
) external returns (bool);
```

### Why EIP-3009?

EIP-3009 enables **gasless transfers** where:
- Users sign authorization off-chain (no gas needed)
- Service providers execute the transfer (they pay gas)
- No pre-approval required (unlike regular ERC-20)
- Single-use nonces prevent replay attacks

## Using Built-in Tokens

### Configuration

For built-in networks, minimal configuration is needed:

```env
# Use built-in Base Sepolia USDC
NETWORK=base-sepolia
PAY_TO_ADDRESS=0xYourWalletAddress

# That's it! Asset address and chain ID are automatic
```

### Available Networks

```env
# Mainnets
NETWORK=base           # Base mainnet
NETWORK=polygon        # Polygon mainnet  
NETWORK=avalanche      # Avalanche mainnet
NETWORK=iotex          # IoTeX mainnet
NETWORK=sei            # Sei mainnet
NETWORK=peaq           # Peaq mainnet
NETWORK=solana         # Solana mainnet

# Testnets  
NETWORK=base-sepolia   # Base testnet (recommended)
NETWORK=polygon-amoy   # Polygon testnet
NETWORK=avalanche-fuji # Avalanche testnet
NETWORK=sei-testnet    # Sei testnet
NETWORK=solana-devnet  # Solana devnet
```

### Getting Testnet Tokens

#### Base Sepolia
- **ETH Faucet:** https://www.alchemy.com/faucets/base-sepolia
- **USDC:** Swap testnet ETH for USDC on testnet DEXs

#### Polygon Amoy
- **MATIC Faucet:** https://faucet.polygon.technology/
- **USDC:** Use testnet bridges or DEXs

#### Avalanche Fuji
- **AVAX Faucet:** https://faucet.avax.network/
- **USDC:** Use testnet bridges

## Facilitator Modes

The x402 implementation supports three settlement modes for handling payments:

### Mode Comparison

| Mode | Setup | Control | Gas Fees | Custom Tokens | Complexity |
|------|-------|---------|----------|---------------|------------|
| **Default Facilitator** | None | Low | Facilitator pays | ❌ USDC only | Low |
| **Local Settlement** | Keys + RPC | Full | You pay | ✅ Any EIP-3009 | Medium |
| **Custom Facilitator** | Deploy service | Full | You pay | ✅ Configurable | High |

### Mode 1: Default Facilitator (Recommended for USDC)

**No configuration needed!** Uses `https://x402.org/facilitator` by default.

```env
# No additional config needed - uses built-in USDC support
PAY_TO_ADDRESS=0xYourWalletAddress
```

**Pros:**
- Zero setup required
- Handles gas fees for you
- Reliable and maintained
- Works with all supported networks

**Cons:**
- Only supports official USDC contracts
- No custom token support
- Less control over settlement

### Mode 2: Local Settlement (Recommended for Custom Tokens)

Handle settlement directly in your server without external facilitator.

```env
SETTLEMENT_MODE=local
PRIVATE_KEY=0xYourPrivateKey
RPC_URL=https://sepolia.base.org
CHAIN_ID=84532

# For custom tokens
ASSET_ADDRESS=0xYourTokenAddress
ASSET_NAME=Your Token Name
ASSET_VERSION=2
```

**Pros:**
- Full control over settlement
- Support for any EIP-3009 token
- No external dependencies
- Custom gas management

**Cons:**
- Requires private key management
- You pay gas fees
- Need RPC endpoint

### Mode 3: Custom Facilitator

Point to your own facilitator service.

```env
FACILITATOR_URL=https://your-facilitator.com
FACILITATOR_API_KEY=your_api_key
ASSET_ADDRESS=0xYourTokenAddress
ASSET_NAME=Your Token
CHAIN_ID=84532
```

**Pros:**
- Full customization
- Centralized settlement logic
- Support for any token
- Advanced features possible

**Cons:**
- Must deploy and maintain facilitator
- Most complex setup
- Requires facilitator API implementation

## Custom Token Configuration

### When to Use Custom Tokens

- Testing with your own token contract
- Using tokens not in the built-in list
- Implementing custom payment logic
- Working on private/custom networks

### Requirements for Custom Tokens

Your token contract must:

1. **Implement EIP-3009**
2. **Support EIP-712 typed data signing**
3. **Have correct domain configuration**

### Example Custom Token Contract

Here's a complete EIP-3009 compatible token:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.27;

import {ERC20} from "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import {ERC20Burnable} from "@openzeppelin/contracts/token/ERC20/extensions/ERC20Burnable.sol";
import {Ownable} from "@openzeppelin/contracts/access/Ownable.sol";
import {EIP712} from "@openzeppelin/contracts/utils/cryptography/EIP712.sol";
import {MessageHashUtils} from "@openzeppelin/contracts/utils/cryptography/MessageHashUtils.sol";

contract TestUSDC is ERC20, ERC20Burnable, Ownable, EIP712 {
    using MessageHashUtils for bytes32;

    bytes32 private constant TRANSFER_WITH_AUTHORIZATION_TYPEHASH =
        keccak256(
            "TransferWithAuthorization(address from,address to,uint256 value,uint256 validAfter,uint256 validBefore,bytes32 nonce)"
        );

    mapping(address => mapping(bytes32 => bool)) private _authorizationState;

    constructor(address initialOwner)
        ERC20("Test USDC", "TUSDC")
        Ownable(initialOwner)
        EIP712("Test USDC", "1")  // ← Domain name and version
    {}

    function mint(address to, uint256 amount) public onlyOwner {
        _mint(to, amount);
    }

    function authorizationState(address authorizer, bytes32 nonce) external view returns (bool) {
        return _authorizationState[authorizer][nonce];
    }

    function transferWithAuthorization(
        address from,
        address to,
        uint256 value,
        uint256 validAfter,
        uint256 validBefore,
        bytes32 nonce,
        uint8 v,
        bytes32 r,
        bytes32 s
    ) external {
        require(block.timestamp > validAfter, "Authorization not yet valid");
        require(block.timestamp < validBefore, "Authorization expired");
        require(!_authorizationState[from][nonce], "Authorization already used");

        bytes32 structHash = keccak256(
            abi.encode(
                TRANSFER_WITH_AUTHORIZATION_TYPEHASH,
                from, to, value, validAfter, validBefore, nonce
            )
        );

        bytes32 digest = _hashTypedDataV4(structHash);
        address signer = ecrecover(digest, v, r, s);
        require(signer != address(0) && signer == from, "Invalid signature");

        _authorizationState[from][nonce] = true;
        _transfer(from, to, value);
    }
}
```

### Custom Token Configuration

#### Environment Variables

```env
# Custom network identifier
NETWORK=my-custom-network

# Required for custom networks
ASSET_ADDRESS=0xYourTokenContractAddress
ASSET_NAME=Test USDC
ASSET_VERSION=1
CHAIN_ID=84532
EXPLORER_URL=https://sepolia.basescan.org

# For local settlement
SETTLEMENT_MODE=local
PRIVATE_KEY=0xYourPrivateKey
RPC_URL=https://sepolia.base.org

# Payment recipient
PAY_TO_ADDRESS=0xYourWalletAddress
```

#### Configuration Explanation

| Variable | Purpose | Example |
|----------|---------|---------|
| `NETWORK` | Network identifier | `my-custom-network` |
| `ASSET_ADDRESS` | Token contract address | `0xEF2C3C652033e9d27F9630EE6717e7fE59276C92` |
| `ASSET_NAME` | EIP-712 domain name | `Test USDC` |
| `ASSET_VERSION` | EIP-712 domain version | `1` |
| `CHAIN_ID` | Blockchain chain ID | `84532` |
| `EXPLORER_URL` | Block explorer base URL | `https://sepolia.basescan.org` |

### EIP-712 Domain Matching

**Critical:** The EIP-712 domain in your configuration must match your token contract.

#### Contract Domain
```solidity
// In your contract constructor
EIP712("Test USDC", "1")
//     ↑ name    ↑ version
```

#### Configuration Domain
```env
# Must match contract exactly
ASSET_NAME=Test USDC     # ← Must match contract name
ASSET_VERSION=1          # ← Must match contract version
```

#### Client Signing Domain
```typescript
// Client creates matching domain
const domain = {
  name: "Test USDC",      // ← From ASSET_NAME
  version: "1",           // ← From ASSET_VERSION  
  chainId: 84532,         // ← From CHAIN_ID
  verifyingContract: "0xEF2C3C652033e9d27F9630EE6717e7fE59276C92" // ← From ASSET_ADDRESS
};
```

**If domains don't match:** Signature verification will fail with "Invalid signature" error.

## Token Setup Walkthrough

### Step 1: Deploy Custom Token

1. **Deploy contract** with correct EIP-712 domain:
```solidity
constructor(address initialOwner)
    ERC20("My Token", "MTK")
    Ownable(initialOwner)
    EIP712("My Token", "1")  // ← Remember these values
{}
```

2. **Mint tokens** to test wallets:
```solidity
// Mint 1000 tokens to client wallet
mint(0x3816BA21dCC9dfD3C714fFDB987163695408653F, 1000000000); // 1000.00 tokens (6 decimals)
```

### Step 2: Configure Environment

```env
# Custom token configuration
NETWORK=my-token
ASSET_ADDRESS=0xYourDeployedTokenAddress
ASSET_NAME=My Token
ASSET_VERSION=1
CHAIN_ID=84532
EXPLORER_URL=https://sepolia.basescan.org

# Local settlement (required for custom tokens)
SETTLEMENT_MODE=local
PRIVATE_KEY=0xYourMerchantPrivateKey
RPC_URL=https://sepolia.base.org

# Payment configuration
PAY_TO_ADDRESS=0xYourMerchantWalletAddress
```

### Step 3: Update Test Client

If using a custom network name, update the test client:

```typescript
// In src/testClient.ts, add your network
const CHAIN_IDS: Record<string, number> = {
  base: 8453,
  'base-sepolia': 84532,
  'my-token': 84532,  // ← Add your custom network
  // ... other networks
};
```

### Step 4: Test the Setup

1. **Start server:**
```bash
npm run dev
```

2. **Check configuration:**
```bash
curl http://localhost:3000/health
```

Should show your custom token:
```json
{
  "status": "healthy",
  "payment": {
    "address": "0xYourMerchantAddress",
    "network": "my-token",
    "price": "$0.10"
  }
}
```

3. **Run full test:**
```bash
npm test
```

## Troubleshooting Custom Tokens

### Common Issues

#### 1. "Invalid signature" Error
```
❌ Settlement failed: execution reverted: "Invalid signature"
```

**Cause:** EIP-712 domain mismatch between contract and configuration.

**Solution:** Verify domain values match exactly:
- Contract: `EIP712("Test USDC", "1")`
- Config: `ASSET_NAME=Test USDC` and `ASSET_VERSION=1`

#### 2. "Asset address must be provided" Error
```
❌ Asset address must be provided for network "my-token"
```

**Solution:** Set `ASSET_ADDRESS` in `.env`:
```env
ASSET_ADDRESS=0xYourTokenContractAddress
```

#### 3. "Direct settlement requires CHAIN_ID" Error
```
❌ Direct settlement requires a numeric CHAIN_ID for network "my-token"
```

**Solution:** Set `CHAIN_ID` in `.env`:
```env
CHAIN_ID=84532
```

#### 4. "ERC20: transfer amount exceeds balance" Error
```
❌ Settlement failed: execution reverted: "ERC20: transfer amount exceeds balance"
```

**Solution:** Mint tokens to the client wallet:
```solidity
mint(clientAddress, 1000000000); // 1000.00 tokens
```

#### 5. "Authorization already used" Error
```
❌ Settlement failed: execution reverted: "Authorization already used"
```

**Cause:** Nonce replay (same nonce used twice).

**Solution:** Each request generates a new random nonce automatically. If you see this, check for duplicate requests.

### Debugging Steps

#### 1. Verify Contract Deployment
```bash
# Check contract exists
curl -X POST https://sepolia.base.org \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"eth_getCode","params":["0xYourTokenAddress","latest"],"id":1}'
```

#### 2. Check Token Balance
```bash
node check-wallet.js
```

#### 3. Verify Domain Configuration
```javascript
// Test EIP-712 domain locally
const domain = {
  name: process.env.ASSET_NAME,
  version: process.env.ASSET_VERSION,
  chainId: parseInt(process.env.CHAIN_ID),
  verifyingContract: process.env.ASSET_ADDRESS
};
console.log('Domain:', domain);
```

#### 4. Test Signature Verification
```javascript
// Test signature creation and verification
const authorization = { /* ... */ };
const signature = await wallet.signTypedData(domain, types, authorization);
const recovered = ethers.verifyTypedData(domain, types, authorization, signature);
console.log('Signer:', wallet.address);
console.log('Recovered:', recovered);
console.log('Match:', recovered.toLowerCase() === wallet.address.toLowerCase());
```

## Token Economics

### Pricing Considerations

#### Atomic Units
Tokens use atomic units (smallest denomination):
- USDC: 6 decimals (1 USDC = 1,000,000 atomic units)
- Your token: Check `decimals()` function

#### Price Configuration
```typescript
// In MerchantExecutor.ts
private getAtomicAmount(priceUsd: number): string {
  const atomicUnits = Math.floor(priceUsd * 1_000_000); // Assumes 6 decimals
  return atomicUnits.toString();
}
```

For custom decimals, override this function:
```typescript
private getAtomicAmount(priceUsd: number): string {
  const decimals = 18; // Your token decimals
  const atomicUnits = Math.floor(priceUsd * Math.pow(10, decimals));
  return atomicUnits.toString();
}
```

### Gas Considerations

#### Who Pays Gas?
- **Client:** Signs authorization (no gas)
- **Server/Facilitator:** Executes `transferWithAuthorization` (pays gas)

#### Gas Optimization
- Use networks with low gas fees (Polygon, Base)
- Batch multiple settlements if possible
- Monitor gas prices and adjust accordingly

## Multi-Token Support

### Supporting Multiple Tokens

You can configure different tokens for different services:

```typescript
// Different MerchantExecutor instances for different tokens
const usdcExecutor = new MerchantExecutor({
  network: 'base-sepolia',
  payToAddress: MERCHANT_ADDRESS,
  price: 0.10, // $0.10 in USDC
});

const customTokenExecutor = new MerchantExecutor({
  network: 'custom',
  assetAddress: '0xCustomToken...',
  assetName: 'Custom Token',
  payToAddress: MERCHANT_ADDRESS,
  price: 1.0, // 1.0 Custom Token
});
```

### Dynamic Token Selection

```typescript
// Let clients choose payment token
app.post('/process', async (req, res) => {
  const preferredToken = req.body.preferredToken || 'usdc';
  
  const executor = preferredToken === 'custom' 
    ? customTokenExecutor 
    : usdcExecutor;
    
  // Use selected executor for payment processing
  const verifyResult = await executor.verifyPayment(paymentPayload);
});
```

## Best Practices

### Security
- Always verify EIP-712 domains match
- Validate all payment parameters
- Check token balances before processing
- Monitor for unusual payment patterns
- Implement rate limiting per wallet

### Performance  
- Cache token contract instances
- Use efficient RPC providers
- Implement connection pooling
- Monitor settlement latency

### User Experience
- Provide clear error messages
- Show exact token amounts and fees
- Support multiple payment options
- Implement retry logic for failed settlements

## RPC Configuration

### RPC Provider Options

#### Alchemy (Recommended)
```env
RPC_URL=https://base-sepolia.g.alchemy.com/v2/YOUR_API_KEY
```
- Reliable and fast
- Good free tier
- Excellent documentation
- Sign up: https://www.alchemy.com

#### Infura
```env
RPC_URL=https://base-sepolia.infura.io/v3/YOUR_PROJECT_ID
```
- Established provider
- Good uptime
- Multiple network support
- Sign up: https://www.infura.io

#### QuickNode
```env
RPC_URL=https://your-endpoint.base-sepolia.quiknode.pro/YOUR_TOKEN/
```
- High performance
- Global infrastructure
- Advanced features
- Sign up: https://www.quicknode.com

#### Public RPCs (Not recommended for production)
```env
# Base Sepolia
RPC_URL=https://sepolia.base.org

# Base Mainnet
RPC_URL=https://mainnet.base.org

# Ethereum Sepolia
RPC_URL=https://sepolia.infura.io/v3/YOUR_PROJECT_ID
```
- Free but unreliable
- Rate limited
- No SLA

### Network-Specific RPC URLs

| Network | RPC URL | Chain ID |
|---------|---------|----------|
| **Base Sepolia** | `https://sepolia.base.org` | 84532 |
| **Base Mainnet** | `https://mainnet.base.org` | 8453 |
| **Polygon Amoy** | `https://polygon-amoy.g.alchemy.com/v2/YOUR_KEY` | 80002 |
| **Ethereum Sepolia** | `https://eth-sepolia.g.alchemy.com/v2/YOUR_KEY` | 11155111 |

## Summary

The x402 protocol provides flexible token and facilitator support:

- **Built-in tokens:** USDC on major networks with zero configuration
- **Custom tokens:** Any EIP-3009 compatible token with proper setup
- **Multiple modes:** Default facilitator, local settlement, or custom facilitator
- **Easy testing:** Comprehensive testnet support

Choose the approach that best fits your needs:
- **Simple USDC payments:** Use default facilitator mode
- **Custom tokens:** Use local settlement mode
- **Advanced features:** Implement custom facilitator

For most use cases, the built-in USDC support provides the easiest path to production.

---

*Custom tokens and flexible facilitator modes enable diverse payment options while maintaining the security and efficiency of the x402 protocol.*
