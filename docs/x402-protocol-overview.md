# x402 Protocol Overview

## What is x402?

x402 is an open standard for internet-native payments over HTTP. It uses the HTTP 402 "Payment Required" status code to enable programmatic, pay-per-use transactions without traditional accounts or subscriptions.

### Core Concept

- Uses HTTP 402 to signal payment required
- Uses EIP-3009 (`transferWithAuthorization`) for blockchain payments
- Works with the A2A (Agent-to-Agent) specification for autonomous payments
- Supports microtransactions (e.g., $0.10 per API call)

## Why x402?

### Traditional Problems
- APIs require subscriptions or API keys
- No way to pay per request
- AI agents can't pay autonomously
- Complex billing systems needed

### x402 Solutions
- Pay only for what you use
- No subscriptions or accounts needed
- Autonomous agent payments
- Simple HTTP-based protocol

## Where Can You Use x402?

### 1. API Monetization
Charge per API request without subscriptions.

**Example:** Weather API charging $0.001 per request
```http
GET /weather?city=London
HTTP/1.1 402 Payment Required
x402: {"accepts": [{"amount": "1000", "asset": "USDC"}]}
```

### 2. AI Agent Services
AI agents pay for services autonomously.

**Example:** AI agent pays for data access
```javascript
// Agent automatically handles payment
const weatherData = await agent.fetchWeather("Tokyo");
// Agent pays $0.001, gets weather data
```

### 3. Content Paywalls
Pay-per-article without full subscriptions.

**Example:** News site charging $0.05 per article
```http
GET /article/123
HTTP/1.1 402 Payment Required
x402: {"accepts": [{"amount": "50000", "asset": "USDC"}]}
```

### 4. On-Demand Services
Pay only for actual usage.

**Examples:**
- Image generation: $0.10 per image
- Video processing: $0.50 per minute
- Cloud storage: $0.01 per GB-hour
- Database queries: $0.001 per query

### 5. Microservices Architecture
Services pay each other for resources.

**Example:** Service A pays Service B for database access
```
Service A → Service B: "Get user data"
Service B: 402 Payment Required ($0.01)
Service A: Pays → Service B: Returns data
```

## Project Types Suitable for x402

| Project Type | Use Case | Example Price |
|--------------|----------|---------------|
| **SaaS APIs** | Per-request billing | $0.001-$1.00 per call |
| **AI/ML Services** | Model inference | $0.10-$5.00 per request |
| **Content Platforms** | Article/video access | $0.05-$2.00 per item |
| **IoT Services** | Device data processing | $0.001-$0.10 per event |
| **dApps** | Decentralized services | $0.01-$10.00 per action |
| **Microservices** | Internal service calls | $0.001-$0.01 per call |

## How x402 Works

### Basic Flow

```
1. Client → Server: "I want to access your service"
2. Server → Client: "402 Payment Required - Pay $0.10"
3. Client → Client: Signs payment authorization (off-chain)
4. Client → Server: "Here's my request + payment signature"
5. Server → Server: Verifies signature
6. Server → Blockchain: Executes transferWithAuthorization
7. Server → Client: "Payment received, here's your data"
```

### Payment Methods

x402 primarily uses **stablecoins** (USDC) for transactions:

- **Instant settlement** - Payments settle in seconds
- **Low fees** - Minimal blockchain transaction costs
- **Global** - Works across borders
- **Programmable** - Perfect for autonomous agents

### Two Payment Cases

#### Case 1: Direct Payment
```
Agent → Blockchain: Sends USDC directly
Agent → Server: "I paid, here's the tx hash"
Server → Blockchain: Verifies transaction
Server → Agent: Returns service response
```

#### Case 2: Signed Intent (Used by this codebase)
```
Agent → Agent: Signs payment authorization
Agent → Server: "Here's my signed payment intent"
Server → Blockchain: Executes transferWithAuthorization
Server → Agent: Returns service response
```

## Agent Wallets and Payments

### How Agents Pay

Agents need cryptocurrency wallets to pay:

```javascript
// Agent has a wallet
const agentWallet = new Wallet(PRIVATE_KEY);
// Address: 0x3816BA21dCC9dfD3C714fFDB987163695408653F
// Balance: 100 USDC
```

### Payment Process

1. **Agent makes request**
```javascript
fetch('/api/service', {
  method: 'POST',
  body: JSON.stringify({ query: "Hello" })
});
```

2. **Server requires payment**
```json
{
  "error": "Payment Required",
  "x402": {
    "accepts": [{
      "asset": "0xUSDC...",
      "amount": "100000",
      "payTo": "0xMerchant..."
    }]
  }
}
```

3. **Agent signs authorization**
```javascript
const signature = await wallet.signTypedData(domain, types, authorization);
// Creates signed authorization (NOT a blockchain transaction)
```

4. **Agent submits payment**
```javascript
fetch('/api/service', {
  method: 'POST',
  body: JSON.stringify({
    query: "Hello",
    metadata: {
      'x402.payment.payload': { signature, authorization }
    }
  })
});
```

5. **Server settles payment**
```javascript
// Server calls transferWithAuthorization on blockchain
// Transfers USDC from agent to merchant
// Returns service response
```

### Payment Frequency

**Every request requires payment:**
- Request 1: Agent pays $0.10 → Gets response
- Request 2: Agent pays $0.10 → Gets response  
- Request 3: Agent pays $0.10 → Gets response

**No subscriptions or credits** - pure pay-per-use.

### Approval Requirements

#### For EIP-3009 Tokens (USDC)
- **No pre-approval needed**
- The signed authorization IS the approval
- Each authorization is single-use (nonce prevents replay)

#### For Regular ERC-20 Tokens
- Would need `approve()` transaction first
- x402 uses EIP-3009 to avoid this complexity

## Real-World Examples

### Example 1: AI Weather Agent

```
Scenario: AI agent needs weather for 3 cities

1. Agent: "Weather for New York"
   → API: 402 Payment Required ($0.001)
   → Agent: Signs payment
   → API: "Temperature: 72°F"
   → Cost: $0.001

2. Agent: "Weather for London"  
   → API: 402 Payment Required ($0.001)
   → Agent: Signs payment
   → API: "Temperature: 15°C"
   → Cost: $0.001

3. Agent: "Weather for Tokyo"
   → API: 402 Payment Required ($0.001)
   → Agent: Signs payment
   → API: "Temperature: 22°C"
   → Cost: $0.001

Total: Agent paid $0.003 for 3 weather requests
```

### Example 2: Image Generation Service

```
User wants 5 images:

Request 1: "Generate cat" → Pay $0.10 → Get image
Request 2: "Generate dog" → Pay $0.10 → Get image
Request 3: "Generate bird" → Pay $0.10 → Get image
Request 4: "Generate fish" → Pay $0.10 → Get image
Request 5: "Generate lion" → Pay $0.10 → Get image

Total: User paid $0.50, Service received $0.50
```

### Example 3: Microservices Economy

```
Service A needs user data:
  Service A → Service B: "Get user profile"
  Service B: 402 Payment Required ($0.01)
  Service A: Pays → Service B: Returns profile data

Service B needs image processing:
  Service B → Service C: "Resize image"
  Service C: 402 Payment Required ($0.05)
  Service B: Pays → Service C: Returns processed image

Each service pays others autonomously
```

## Benefits of x402

### For Service Providers
- **Fair pricing** - Charge exactly for usage
- **No freeloaders** - Every request is paid
- **Global reach** - Accept payments worldwide
- **Automated billing** - No invoices or subscriptions
- **Instant settlement** - Get paid immediately

### For Clients/Agents
- **Pay per use** - No wasted subscription fees
- **Autonomous payments** - Agents pay automatically
- **No accounts** - No signup or authentication
- **Transparent pricing** - Know cost upfront
- **Global access** - Use any service worldwide

### For the Ecosystem
- **Machine economy** - Enables AI-to-AI commerce
- **Micropayments** - Makes small transactions viable
- **Interoperability** - Standard protocol across services
- **Innovation** - New business models possible

## Technical Requirements

### For Service Providers
- HTTP server supporting 402 status codes
- EIP-3009 compatible token (USDC recommended)
- Wallet for receiving payments
- Optional: Facilitator service for settlement

### For Clients/Agents
- Cryptocurrency wallet with funds
- EIP-712 signing capability
- HTTP client supporting x402 protocol
- Token balance for payments

## Supported Networks and Tokens

x402 works on multiple blockchain networks:

| Network | Token | Use Case |
|---------|-------|----------|
| Base | USDC | Production (low fees) |
| Base Sepolia | USDC | Testing |
| Polygon | USDC | Production (very low fees) |
| Ethereum | USDC | Production (higher fees) |
| Avalanche | USDC | Production |
| Solana | USDC | Production (very low fees) |

### Custom Tokens
You can use any token that implements EIP-3009:
- Must have `transferWithAuthorization` function
- Must support EIP-712 typed data signing
- Must have correct domain name and version

## Quick Start

### Prerequisites
- Node.js 18+ installed
- OpenAI API key ([Get one here](https://platform.openai.com/api-keys))
- A wallet address to receive USDC payments

### Setup Steps

1. **Install and configure:**
```bash
npm install
cp .env.example .env
# Edit .env with your OPENAI_API_KEY and PAY_TO_ADDRESS
```

2. **Start the server:**
```bash
npm run dev
```

3. **Test the API:**
```bash
curl http://localhost:3000/health
```

4. **Test payment flow:**
```bash
npm test  # Requires CLIENT_PRIVATE_KEY in .env
```

### Configuration

**Required variables:**
```env
OPENAI_API_KEY=sk-your-openai-api-key
PAY_TO_ADDRESS=0xYourWalletAddress
```

**Optional variables:**
```env
NETWORK=base-sepolia          # Testing network
SETTLEMENT_MODE=local         # Direct settlement
PRIVATE_KEY=0xYourPrivateKey  # For local settlement
```

## Next Steps

- [Payment Flow Guide](./payment-flow-guide.md) - Detailed technical flow
- [Architecture Guide](./architecture-guide.md) - Code structure
- [Token Support Guide](./token-support-guide.md) - Token configuration
- [Facilitator Guide](./facilitator-guide.md) - Settlement options

---

*x402 enables a new economy where services can charge fairly and agents can pay autonomously.*
