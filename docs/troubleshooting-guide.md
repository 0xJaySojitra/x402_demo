# Troubleshooting & API Reference

This guide helps you diagnose and fix common issues with the x402 implementation.

## Common Issues and Solutions

### 1. Payment Verification Issues

#### "Invalid signature" Error
```
❌ Settlement failed: execution reverted: "Invalid signature"
```

**Causes:**
- EIP-712 domain mismatch between contract and configuration
- Wrong signature format
- Incorrect authorization data

**Solutions:**

1. **Check EIP-712 domain matching:**
```env
# Contract: EIP712("Test USDC", "1")
# Config must match exactly:
ASSET_NAME=Test USDC
ASSET_VERSION=1
```

2. **Verify signature creation:**
```javascript
// Ensure domain matches exactly
const domain = {
  name: "Test USDC",      // Must match contract
  version: "1",           // Must match contract
  chainId: 84532,         // Must match network
  verifyingContract: "0xEF2C3C652033e9d27F9630EE6717e7fE59276C92"
};

// Check signature format
console.log('Signature length:', signature.length); // Should be 132 chars (0x + 130 hex)
```

3. **Debug signature verification:**
```javascript
const recovered = ethers.verifyTypedData(domain, types, authorization, signature);
console.log('Expected signer:', authorization.from);
console.log('Recovered signer:', recovered);
console.log('Match:', recovered.toLowerCase() === authorization.from.toLowerCase());
```

#### "Signature does not match payer address" Error
```
❌ Payment verification failed: Signature does not match payer address
```

**Causes:**
- Wrong private key used for signing
- Authorization data modified after signing
- Domain parameters incorrect

**Solutions:**

1. **Verify wallet address:**
```javascript
const wallet = new Wallet(PRIVATE_KEY);
console.log('Wallet address:', wallet.address);
console.log('Authorization from:', authorization.from);
// These should match
```

2. **Check authorization integrity:**
```javascript
// Ensure authorization wasn't modified after signing
const originalAuth = { /* original values */ };
const currentAuth = authorization;
console.log('Authorization changed:', JSON.stringify(originalAuth) !== JSON.stringify(currentAuth));
```

#### "Payment authorization has expired" Error
```
❌ Payment verification failed: Payment authorization has expired
```

**Causes:**
- `validBefore` timestamp in the past
- Clock synchronization issues
- Processing delays

**Solutions:**

1. **Increase timeout:**
```javascript
// Give more time for processing
const now = Math.floor(Date.now() / 1000);
const authorization = {
  // ...
  validBefore: String(now + 3600), // 1 hour instead of 10 minutes
};
```

2. **Check system time:**
```bash
# Ensure system clock is synchronized
date
# Should match current time
```

### 2. Settlement Issues

#### "ERC20: transfer amount exceeds balance" Error
```
❌ Settlement failed: execution reverted: "ERC20: transfer amount exceeds balance"
```

**Causes:**
- Client wallet has insufficient token balance
- Wrong token decimals calculation

**Solutions:**

1. **Check client balance:**
```bash
node check-wallet.js
```

2. **Mint tokens to client (for custom tokens):**
```solidity
// In your token contract
mint(clientAddress, 1000000000); // 1000.00 tokens (assuming 6 decimals)
```

3. **Verify amount calculation:**
```javascript
// Check atomic units conversion
const priceUSD = 0.10;
const decimals = 6; // USDC has 6 decimals
const atomicAmount = Math.floor(priceUSD * Math.pow(10, decimals));
console.log('Atomic amount:', atomicAmount); // Should be 100000 for $0.10
```

#### "Authorization already used" Error
```
❌ Settlement failed: execution reverted: "Authorization already used"
```

**Causes:**
- Nonce reuse (same nonce used twice)
- Duplicate request processing

**Solutions:**

1. **Ensure unique nonces:**
```javascript
function generateNonce() {
  return `0x${randomBytes(32).toString('hex')}`; // Always generates unique nonce
}
```

2. **Check for duplicate requests:**
```javascript
// Add request deduplication
const processedRequests = new Set();
if (processedRequests.has(requestId)) {
  return { error: 'Request already processed' };
}
processedRequests.add(requestId);
```

#### "Facilitator settle failed (500)" Error
```
❌ Settlement failed: Facilitator settle failed (500): {"success":false,"errorReason":"unexpected_settle_error"}
```

**Causes:**
- Facilitator doesn't support your token
- Network connectivity issues
- Facilitator service down

**Solutions:**

1. **Switch to local settlement:**
```env
SETTLEMENT_MODE=local
PRIVATE_KEY=0xYourPrivateKey
RPC_URL=https://sepolia.base.org
```

2. **Check facilitator status:**
```bash
curl https://x402.org/facilitator/health
```

3. **Use supported tokens:**
```env
# Use official USDC instead of custom token
NETWORK=base-sepolia
# Remove ASSET_ADDRESS (use built-in)
```

### 3. Configuration Issues

#### "OPENAI_API_KEY is required" Error
```
❌ OPENAI_API_KEY is required when AI_PROVIDER=openai
```

**Solutions:**

1. **Set OpenAI API key:**
```env
OPENAI_API_KEY=sk-your-openai-api-key-here
```

2. **Or switch to EigenAI:**
```env
AI_PROVIDER=eigenai
EIGENAI_API_KEY=your-eigenai-key
```

#### "PAY_TO_ADDRESS is required" Error
```
❌ PAY_TO_ADDRESS is required
```

**Solutions:**

1. **Set merchant wallet address:**
```env
PAY_TO_ADDRESS=0x237b7244ea07073e86e6f11c42db3a95378b181f
```

#### "Asset address must be provided" Error
```
❌ Asset address must be provided for network "my-custom-network"
```

**Solutions:**

1. **For custom networks, provide asset details:**
```env
NETWORK=my-custom-network
ASSET_ADDRESS=0xYourTokenAddress
ASSET_NAME=Your Token Name
ASSET_VERSION=1
CHAIN_ID=84532
```

2. **Or use built-in network:**
```env
NETWORK=base-sepolia
# No ASSET_ADDRESS needed
```

### 4. Network and RPC Issues

#### "Network request failed" Error
```
❌ Error communicating with facilitator: Network request failed
```

**Causes:**
- Internet connectivity issues
- Facilitator service down
- Firewall blocking requests

**Solutions:**

1. **Check connectivity:**
```bash
curl https://x402.org/facilitator/health
```

2. **Use local settlement:**
```env
SETTLEMENT_MODE=local
```

3. **Check firewall settings:**
```bash
# Ensure outbound HTTPS is allowed
curl -I https://google.com
```

#### "RPC URL connection failed" Error
```
❌ Direct settlement requires an RPC URL for network "base-sepolia"
```

**Solutions:**

1. **Set RPC URL:**
```env
RPC_URL=https://sepolia.base.org
```

2. **Use reliable RPC provider:**
```env
# Alchemy (recommended)
RPC_URL=https://base-sepolia.g.alchemy.com/v2/YOUR_KEY

# Infura
RPC_URL=https://base-sepolia.infura.io/v3/YOUR_PROJECT_ID
```

3. **Test RPC connection:**
```bash
curl -X POST https://sepolia.base.org \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}'
```

### 5. Gas and Transaction Issues

#### "Insufficient funds for gas" Error
```
❌ Settlement failed: insufficient funds for gas * price + value
```

**Causes:**
- Merchant wallet has no ETH for gas
- Gas price too high

**Solutions:**

1. **Add ETH to merchant wallet:**
```bash
# Get testnet ETH from faucets
# Base Sepolia: https://www.alchemy.com/faucets/base-sepolia
```

2. **Check wallet balance:**
```bash
node check-wallet.js
```

3. **Optimize gas settings:**
```javascript
// In settlement code, add gas optimization
const tx = await contract.transferWithAuthorization(
  /* params */,
  {
    gasLimit: 100000, // Set reasonable limit
    maxFeePerGas: ethers.parseUnits('20', 'gwei'), // Cap gas price
  }
);
```

#### "Transaction underpriced" Error
```
❌ Settlement failed: transaction underpriced
```

**Solutions:**

1. **Increase gas price:**
```javascript
const feeData = await provider.getFeeData();
const tx = await contract.transferWithAuthorization(
  /* params */,
  {
    maxFeePerGas: feeData.maxFeePerGas * 110n / 100n, // 10% higher
    maxPriorityFeePerGas: feeData.maxPriorityFeePerGas * 110n / 100n,
  }
);
```

### 6. Client-Side Issues

#### "CLIENT_PRIVATE_KEY not configured" Warning
```
⚠️ TEST 2: Skipped (no CLIENT_PRIVATE_KEY configured)
```

**Solutions:**

1. **Set client private key for testing:**
```env
CLIENT_PRIVATE_KEY=0xYourTestWalletPrivateKey
```

2. **Create test wallet:**
```javascript
const wallet = Wallet.createRandom();
console.log('Address:', wallet.address);
console.log('Private Key:', wallet.privateKey);
```

3. **Get testnet tokens:**
   - **Base Sepolia ETH:** https://www.alchemy.com/faucets/base-sepolia
   - **USDC:** Swap testnet ETH for USDC on testnet DEXs

#### "Request failed: undefined" Error
```
❌ Request failed: undefined
```

**Causes:**
- Settlement failed but request was processed
- Response parsing error

**Solutions:**

1. **Check server logs for settlement status:**
```bash
# Look for settlement result in server logs
npm run dev
```

2. **Add better error handling:**
```javascript
// In test client
if (response.success && response.task) {
  console.log('✅ Success');
} else {
  console.log('❌ Failed:', response.error || 'Unknown error');
  console.log('Settlement:', response.settlement);
}
```

### 7. Environment Configuration Issues

#### Missing Environment Variables
```
❌ OPENAI_API_KEY is required
❌ PAY_TO_ADDRESS is required
```

**Complete .env template:**
```env
# Required
OPENAI_API_KEY=sk-your-openai-api-key
PAY_TO_ADDRESS=0xYourWalletAddress

# Optional - Basic Config
PORT=3000
NETWORK=base-sepolia
X402_DEBUG=true

# Optional - Local Settlement
SETTLEMENT_MODE=local
PRIVATE_KEY=0xYourPrivateKey
RPC_URL=https://sepolia.base.org
CHAIN_ID=84532

# Optional - Custom Token
ASSET_ADDRESS=0xYourTokenAddress
ASSET_NAME=Your Token Name
ASSET_VERSION=1

# Optional - Testing
CLIENT_PRIVATE_KEY=0xYourTestWalletPrivateKey
```

## Debugging Tools

### 1. Enable Debug Logging
```env
X402_DEBUG=true
```

### 2. Check Wallet Balances
```bash
node check-wallet.js
```

### 3. Test Facilitator Communication
```bash
node test-facilitator.js
```

### 4. Manual API Testing
```bash
# Test health endpoint
curl http://localhost:3000/health

# Test payment requirement
curl -X POST http://localhost:3000/process \
  -H "Content-Type: application/json" \
  -d '{"message": {"parts": [{"kind": "text", "text": "Hello"}]}}'
```

### 5. Blockchain Explorer
Check transactions on block explorers:
- Base Sepolia: https://sepolia.basescan.org
- Polygon Amoy: https://amoy.polygonscan.com
- Ethereum Sepolia: https://sepolia.etherscan.io

## Performance Issues

### Slow Response Times

**Causes:**
- Network latency
- RPC provider delays
- OpenAI API delays

**Solutions:**

1. **Use faster RPC provider:**
```env
# Switch to premium RPC
RPC_URL=https://base-sepolia.g.alchemy.com/v2/YOUR_KEY
```

2. **Optimize OpenAI calls:**
```env
AI_MODEL=gpt-3.5-turbo  # Faster than gpt-4
AI_MAX_TOKENS=100       # Shorter responses
```

3. **Add timeouts:**
```javascript
// Add request timeouts
const controller = new AbortController();
setTimeout(() => controller.abort(), 30000); // 30s timeout

const response = await fetch(url, {
  signal: controller.signal,
  // ...
});
```

### High Gas Costs

**Solutions:**

1. **Use cheaper networks:**
```env
NETWORK=polygon-amoy  # Very low gas fees
```

2. **Batch transactions:**
```javascript
// Process multiple payments in one transaction
// (Requires custom contract implementation)
```

3. **Optimize gas usage:**
```javascript
// Use gas estimation
const gasEstimate = await contract.estimateGas.transferWithAuthorization(/* params */);
const gasLimit = gasEstimate * 120n / 100n; // 20% buffer
```

## Monitoring and Alerts

### Key Metrics to Monitor

1. **Payment Success Rate**
```javascript
const successRate = successfulPayments / totalPayments;
if (successRate < 0.95) {
  alert('Payment success rate below 95%');
}
```

2. **Average Settlement Time**
```javascript
const avgSettlementTime = totalSettlementTime / settledPayments;
if (avgSettlementTime > 30000) { // 30 seconds
  alert('Settlement time too high');
}
```

3. **Error Rates**
```javascript
const errorRate = failedRequests / totalRequests;
if (errorRate > 0.05) {
  alert('Error rate above 5%');
}
```

### Health Check Endpoint

Add monitoring to your health endpoint:
```javascript
app.get('/health', async (req, res) => {
  const health = {
    status: 'healthy',
    timestamp: new Date().toISOString(),
    checks: {
      database: await checkDatabase(),
      rpc: await checkRPC(),
      facilitator: await checkFacilitator(),
      openai: await checkOpenAI(),
    }
  };
  
  const allHealthy = Object.values(health.checks).every(check => check.status === 'ok');
  
  res.status(allHealthy ? 200 : 503).json(health);
});
```

## Getting Help

### 1. Check Logs
Always check both client and server logs for error details.

### 2. Verify Configuration
Double-check all environment variables match your setup.

### 3. Test Components Individually
- Test RPC connection
- Test facilitator communication  
- Test token contract calls
- Test OpenAI API calls

### 4. Use Minimal Configuration
Start with the simplest setup and add complexity gradually.

### 5. Community Resources
- [x402 Documentation](https://learnx402.dev)
- [GitHub Issues](https://github.com/coinbase/x402)
- [Discord Community](https://discord.gg/x402)

---

## Complete API Reference

### Server Endpoints

#### Health Check - `GET /health`

**Response:**
```json
{
  "status": "healthy",
  "service": "x402-payment-api", 
  "version": "1.0.0",
  "payment": {
    "address": "0x237b7244ea07073e86e6f11c42db3a95378b181f",
    "network": "base-sepolia",
    "price": "$0.10"
  }
}
```

#### Main Processing - `POST /process`

**Request (No Payment):**
```json
{
  "message": {
    "messageId": "msg-1762947427975",
    "role": "user",
    "parts": [{"kind": "text", "text": "What is the meaning of life?"}]
  }
}
```

**Response (Payment Required):**
```json
{
  "success": false,
  "error": "Payment Required",
  "task": {
    "status": {
      "state": "input-required",
      "message": {
        "metadata": {
          "x402.payment.required": {
            "x402Version": 1,
            "accepts": [{
              "scheme": "exact",
              "network": "base-sepolia",
              "asset": "0xEF2C3C652033e9d27F9630EE6717e7fE59276C92",
              "payTo": "0x237b7244ea07073e86e6f11c42db3a95378b181f",
              "maxAmountRequired": "100000",
              "resource": "http://localhost:3000/process",
              "description": "AI request processing service",
              "mimeType": "application/json",
              "maxTimeoutSeconds": 600
            }]
          }
        }
      }
    }
  }
}
```

**Request (With Payment):**
```json
{
  "message": {
    "messageId": "msg-1762947428004",
    "role": "user", 
    "parts": [{"kind": "text", "text": "Tell me a joke!"}],
    "metadata": {
      "x402.payment.payload": {
        "x402Version": 1,
        "scheme": "exact",
        "network": "base-sepolia",
        "payload": {
          "signature": "0xa9d468eaeceead272c80cf174fc06d23f4e83eefb4dda7baecc3c9b2b2981da129b1550e6537fad6f75fe7a8ae054d10efcf7c3d94070d1e5a0bf75b3dff35391c",
          "authorization": {
            "from": "0x3816BA21dCC9dfD3C714fFDB987163695408653F",
            "to": "0x237b7244ea07073e86e6f11c42db3a95378b181f",
            "value": "100000",
            "validAfter": "0",
            "validBefore": "1762948027",
            "nonce": "0xe38c00ee1694ca83254f6f42c027c0fa2b460ce6d3728ff5c49739dc447dd717"
          }
        }
      },
      "x402.payment.status": "payment-submitted"
    }
  }
}
```

**Response (Success):**
```json
{
  "success": true,
  "task": {
    "status": {
      "state": "completed",
      "message": {
        "parts": [{"kind": "text", "text": "Why did the TypeScript developer go broke? Because he lost his 'type' of investment!"}]
      }
    },
    "metadata": {
      "x402.payment.status": "payment-completed",
      "x402.payment.receipts": [{
        "success": true,
        "transaction": "0xab9cdff9b4ae8400aaf22ab27b6d36cf2094e437f71c3e889270c918e8e8a8cb",
        "network": "base-sepolia",
        "payer": "0x3816BA21dCC9dfD3C714fFDB987163695408653F"
      }]
    }
  },
  "settlement": {
    "success": true,
    "transaction": "0xab9cdff9b4ae8400aaf22ab27b6d36cf2094e437f71c3e889270c918e8e8a8cb",
    "network": "base-sepolia",
    "payer": "0x3816BA21dCC9dfD3C714fFDB987163695408653F"
  }
}
```

### Data Types

#### PaymentPayload
```typescript
interface PaymentPayload {
  x402Version: number;
  scheme: string;
  network: string;
  payload: {
    signature: string;
    authorization: {
      from: string;
      to: string;
      value: string;
      validAfter: string;
      validBefore: string;
      nonce: string;
    };
  };
}
```

### HTTP Status Codes

| Code | Meaning | When Used |
|------|---------|-----------|
| `200` | OK | Successful request (with or without payment) |
| `400` | Bad Request | Invalid request format |
| `402` | Payment Required | Payment verification failed |
| `500` | Internal Server Error | Server error during processing |

### Payment Status Values

| Status | Description |
|--------|-------------|
| `payment-required` | Payment is needed to process request |
| `payment-submitted` | Client has submitted payment signature |
| `payment-verified` | Payment signature verified successfully |
| `payment-rejected` | Payment verification failed |
| `payment-completed` | Payment settled on blockchain |
| `payment-failed` | Settlement failed |

---

*This comprehensive guide covers troubleshooting common issues and provides complete API reference documentation for the x402 payment protocol.*
