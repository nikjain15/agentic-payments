# Agentic Payments: Terminology

Plain-English reference guide for payment and protocol terms.

---

## Payment Terms

| Term | What It Means | Example |
|------|---------------|---------|
| **PSP (Payment Service Provider)** | Company that processes payments and handles credentials | Stripe, Adyen, Square |
| **Tokenization** | Converting sensitive data into a non-sensitive token | `4532-1234-5678-9010` → `tok_abc123` |
| **Detokenization** | Converting token back to original data (PSP does this) | `tok_abc123` → actual card for charging |
| **Credential Vault** | Secure database storing tokenized payment methods | Your encrypted storage of Stripe tokens |
| **PCI-DSS** | Compliance standard for handling card data | Required if storing payment credentials |

---

## Authorization Terms

| Term | What It Means | Example |
|------|---------------|---------|
| **OAuth** | Standard protocol for user authorization | "Allow ChatGPT to access your payment methods" |
| **OAuth Scopes** | Granular permissions defining what app can do | `payment:read`, `payment:authorize` |
| **Authorization** | Explicit user permission for agent to act | User says "ChatGPT can spend up to $500" |
| **Allowance** | Permission with spending constraints | Max $500/transaction, $2000/day |

---

## Token Types

| Term | What It Means | When Used |
|------|---------------|-----------|
| **Single-Use Token** | Can only be used once | Most transactions (highest security) |
| **Multi-Use Token** | Can be used multiple times with limits | Subscriptions, recurring payments |
| **Binding** | Tying token to specific conditions | Token only works for Merchant X, Amount Y |

---

## Protocol Terms

| Term | What It Means | Purpose |
|------|---------------|---------|
| **UCP** | Universal Commerce Protocol | Standardizes how agents discover and checkout with merchants |
| **ACP** | Agentic Commerce Protocol | Defines ChatGPT's checkout flow with merchants |
| **MoR (Merchant of Record)** | Entity legally responsible for transaction | Usually the merchant, not the payment platform |
| **Checkout Session** | Temporary record of a potential purchase | Tracks items, amounts, payment through completion |

---

## Key Concepts

### Encryption
- **At Rest**: Data encrypted on disk (AES-256)
- **In Transit**: Data encrypted during transmission (TLS/HTTPS)

### Payment Flow
```
User Card → Tokenize (Stripe) → Store Token (Encrypted) → 
Agent Requests → Check Limits → Issue Payment Token → 
Merchant Charges → Log Transaction
```

### Security Layers
1. **Never store raw cards** — only Stripe tokens
2. **Encrypt all tokens** — AES-256 encryption
3. **Bind payment tokens** — to checkout, merchant, amount
4. **Expire quickly** — 15-minute limit on payment tokens
5. **Log everything** — complete audit trail

