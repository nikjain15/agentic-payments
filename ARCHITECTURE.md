# Agentic Payments: System Architecture

---

## System Overview

```
┌──────────────────────────────────────┐
│         AI AGENTS                    │
│  (ChatGPT, Claude, Perplexity)       │
└────────────────┬─────────────────────┘
                 ↓ OAuth + API
┌──────────────────────────────────────┐
│   AGENTIC PAYMENTS INFRASTRUCTURE    │
│                                      │
│  • OAuth Server (agent authorization)│
│  • Credential Vault (tokenized)      │
│  • Authorization Engine (limits)     │
│  • Payment API (tokens)              │
│  • Transaction Logs (audit)          │
└────┬────────────────────────┬────────┘
     ↓                        ↓
┌─────────────┐      ┌─────────────────┐
│     PSPs    │      │  UCP/ACP        │
│ (Stripe)    │      │  (Merchants)    │
└─────────────┘      └─────────────────┘
```

---

## Core Components

| Component | Purpose | Key Functions |
|-----------|---------|---------------|
| **OAuth Server** | Agent authorization | User grants/revokes agent access<br>Issues OAuth tokens (90-day validity)<br>Manages consent flow |
| **Credential Vault** | Secure storage | Stores tokenized credentials (AES-256)<br>Never stores raw card numbers<br>Users add/remove payment methods |
| **Authorization Engine** | Permission checking | Validates OAuth tokens<br>Checks spending limits (per-transaction, daily, monthly)<br>Verifies merchant not blocked |
| **Agent API** | Payment requests | `/v1/authorize-payment` endpoint<br>Returns single-use tokens<br>Compatible with UCP/ACP |
| **Transaction Log** | Audit & visibility | Records all transactions<br>User dashboard access<br>Tracks spending against limits |

---

## Data Flows

### 1. User Setup (One-Time)
```
User → Adds card → Stripe tokenizes → Vault stores token → Card ready
```

### 2. Agent Authorization
```
User → Authorizes agent → OAuth consent → Token issued → Agent can request payments
```

### 3. Payment Execution
```
Agent → Requests payment → Engine checks limits → Token issued → Agent uses in UCP/ACP → Transaction logged
```

---

## Integration Points

### With UCP
- Implements payment handler (`/tokenize`, `/detokenize` endpoints)
- Extends OAuth scopes for payment credentials
- Tokens bound to checkout session + merchant + amount

### With ACP
- Provides credential vault for `delegate_payment` flow
- Returns credentials in ACP-compatible format
- OAuth-based user consent layer

---

## Security Model

| Layer | Implementation |
|-------|----------------|
| **Credentials** | Tokenized via Stripe, encrypted at rest (AES-256), never store raw cards |
| **OAuth Tokens** | JWT-based, 90-day expiry, user can revoke instantly |
| **Payment Tokens** | Single-use, bound to checkout/merchant/amount, 15-min expiry |
| **Audit** | All actions logged with timestamps, user can view transaction history |

