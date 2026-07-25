# Agentic Payments Infrastructure

Payment delegation layer for AI agents acting on behalf of users.

---

## The Problem

AI agents like ChatGPT, Claude, and Perplexity can execute complex tasks for users, including making purchases. But there's a fundamental gap in how payments work:

**When an agent needs to make a payment:**
- There's no standardized way for the agent to access payment credentials
- Users have no central place to authorize which agents can spend
- Payment details must be entered separately for each agent
- Users can't see agent spending or easily revoke access
- No standard exists for spending limits or authorization rules

Both UCP and ACP protocols have sophisticated checkout flows, but they don't specify how users authorize agents to access payment credentials. They assume either:
- The agent already has card details (unspecified how)
- Users will enter payment info at each transaction (friction)
- Merchants will manage agent credentials (not user-controlled)

**Result:** Users can't confidently delegate spending. Agents can't reliably complete purchases. Commerce stays fragmented.

---

## The Solution

A simple infrastructure layer that sits between users' payment methods and AI agents.

| Stakeholder | Capabilities |
|-------------|-------------|
| **Users** | • Store payment credentials once (tokenized, secure)<br>• Authorize specific agents with granular controls<br>• Set spending rules (per-transaction, daily, categories)<br>• Revoke agent access instantly<br>• View complete transaction history across all agents |
| **Agents** | • Request payment authorization via single API<br>• Receive pre-authorized tokens for UCP/ACP flows<br>• Work across multiple merchants without separate setup<br>• Complete purchases without adding user friction |
| **Merchants** | • Receive pre-authorized transactions<br>• User consent layer for fraud signals<br>• Works with existing PSPs (Stripe, Adyen) |

---

## Why It Matters for Agentic Commerce

### Users
Set up payment once. Authorize multiple agents from a single dashboard. Control spending with per-agent limits. Revoke access instantly. See every transaction across all agents.

### Agent Platforms
Integrate one API instead of building payment infrastructure. Users complete purchases without friction. No credential storage liability.

### Merchants
Receive pre-authorized transactions from authenticated agents. User consent is explicit. Works with existing payment rails (Stripe, Adyen).

### Protocols (UCP, ACP)
Fills the missing credential delegation layer. Provides a reference implementation for user authorization flows.

## How It Fits with Existing Protocols

### UCP (Universal Commerce Protocol)
UCP defines how platforms discover business capabilities and execute checkout sessions. It includes:
- Checkout session management (create, update, complete)
- Payment handler abstraction (tokenization, detokenization)
- Identity linking via OAuth

**Where we fit:** We become the credential provider that UCP checkout flows integrate with. When a platform needs to complete a checkout, it can:
1. Call our API to get a pre-authorized payment token (if the user authorized this agent)
2. Submit that token to UCP's tokenization handler
3. Complete the checkout

We also extend UCP's identity-linking scope to include payment-credential management: users authorize agents to access their payment methods via OAuth scopes.

### ACP (Agentic Commerce Protocol)
ACP defines how agents complete checkouts with merchants via a REST API. It includes:
- Checkout session creation and updates
- Delegate payment API (submit credentials, get tokens)
- Order webhooks for updates

**Where we fit:** We become the credential vault that ACP agents reference. When an agent wants to complete a checkout, it:
- Calls our API with the user's authorization (OAuth token)
- Receives payment tokens scoped to specific merchants and amounts
- Uses those tokens with ACP's delegate_payment flow
- Relies on us for the missing user-consent and authorization layer, with every transaction logged for audit and user visibility

### Complementary Approach
This does not replace UCP or ACP. It fills the gap both protocols assume but do not specify: how users authorize agents to access their payment credentials.

---

## What We're NOT Building

❌ Payment processors (Stripe, Adyen, etc. do this)
❌ Shopping experiences (agents/merchants do this)
❌ Merchant relationships (protocols like ACP/UCP do this)
❌ Fraud/risk analysis (PSPs do this)
❌ Analytics platforms (nice-to-have, not core)
❌ Budget management apps (user dashboards are simple, not comprehensive)

---

## Scope

**Building:**
- Credential vault and tokenization
- OAuth authorization server
- Permission checking and spending limits
- Agent API for payment requests
- Transaction logging and user dashboard

**Not building:**
- Payment processing (PSPs handle this)
- Shopping experiences (agents handle this)
- Merchant relationships (protocols handle this)
- Fraud/risk analysis (PSPs handle this)

**Focus:** The layer between users' payment credentials and agents' purchase requests.