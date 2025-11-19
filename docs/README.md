# x402 Protocol Documentation

Welcome to the comprehensive documentation for the x402 payment protocol implementation.

## 📚 Documentation Structure

This documentation is organized into 5 comprehensive guides that cover everything you need to know about x402:

### 🎯 Core Documentation

1. **[x402 Protocol Overview](./x402-protocol-overview.md)** - Complete guide to the x402 protocol
   - What is x402 and how it works
   - Use cases and real-world examples
   - Quick start guide
   - Agent-to-agent payments

2. **[Payment Flow Guide](./payment-flow-guide.md)** - Detailed payment flow documentation
   - Step-by-step payment process
   - EIP-3009 and EIP-712 explained
   - Client and server implementation

3. **[Architecture Guide](./architecture-guide.md)** - Codebase structure and implementation
   - File structure and purposes
   - Key components explained
   - Integration patterns

4. **[Token & Facilitator Guide](./token-support-guide.md)** - Token support and facilitator configuration
   - Built-in supported tokens (USDC)
   - Custom token setup
   - Facilitator modes (default, custom, local)
   - RPC configuration

5. **[Troubleshooting & API Reference](./troubleshooting-guide.md)** - Solutions and complete API docs
   - Common issues and fixes
   - Environment configuration
   - Complete API reference
   - Debugging tools

## 🚀 Quick Navigation

- **New to x402?** → Start with [Protocol Overview](./x402-protocol-overview.md)
- **Want to understand the flow?** → Read [Payment Flow Guide](./payment-flow-guide.md)
- **Setting up your implementation?** → Check [Architecture Guide](./architecture-guide.md)
- **Need to configure tokens?** → See [Token & Facilitator Guide](./token-support-guide.md)
- **Having issues?** → Check [Troubleshooting Guide](./troubleshooting-guide.md)

## 🎯 What is x402?

x402 is an open standard for internet-native payments over HTTP. It enables:

- **Pay-per-use APIs** - Charge for each API request
- **Agent-to-Agent payments** - AI agents paying each other autonomously  
- **Microtransactions** - Small payments (e.g., $0.10 per request)
- **No subscriptions** - Pay only for what you use

## 🏗️ Architecture Overview

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Client    │    │   Server    │    │ Facilitator │
│  (Agent)    │    │   (API)     │    │ (Optional)  │
└──────┬──────┘    └──────┬──────┘    └──────┬──────┘
       │                  │                  │
       │ 1. Request       │                  │
       ├─────────────────►│                  │
       │                  │                  │
       │ 2. 402 Payment   │                  │
       │    Required      │                  │
       │◄─────────────────┤                  │
       │                  │                  │
       │ 3. Sign Payment  │                  │
       │    (off-chain)   │                  │
       │                  │                  │
       │ 4. Request +     │                  │
       │    Signature     │                  │
       ├─────────────────►│                  │
       │                  │ 5. Verify/Settle │
       │                  ├─────────────────►│
       │                  │                  │
       │                  │ 6. Settlement    │
       │                  │    Result        │
       │                  │◄─────────────────┤
       │ 7. Response      │                  │
       │◄─────────────────┤                  │
```

## 📋 Key Files in This Codebase

| File | Purpose |
|------|---------|
| `src/server.ts` | Main HTTP server, handles payment validation |
| `src/MerchantExecutor.ts` | Payment verification and settlement logic |
| `src/ExampleService.ts` | Example service (OpenAI integration) |
| `src/testClient.ts` | Test client demonstrating payment flow |
| `src/x402Types.ts` | TypeScript type definitions |

## 🔗 External Resources

- [x402 Official Website](https://learnx402.dev)
- [x402 npm Package](https://www.npmjs.com/package/x402)
- [A2A Specification](https://github.com/google/a2a)
- [Coinbase x402 Documentation](https://www.coinbase.com/developer-platform/discover/launches/google_x402)

---

*This documentation covers everything you need to know about implementing and using the x402 payment protocol.*
