# Digipay B2B Discovery

This project is the final result of a **B2B discovery** of merchant services for Digipay’s **credit product** (Wallet, 1pay, 4pay) in Iran. Reference benchmarks: **Tabby** (business/product), **Klarna / Afterpay / Stripe** (merchant DX and API patterns).

---

## Discovery outputs (target)

### Problem space

#### 1. Segmenting the merchants

Merchants are segmented on **three independent attributes** (up to **12** combinations). Keep segments separate during discovery; merge later only when the **service bundle** is identical.

| Attribute | Values | Summary |
|-----------|--------|---------|
| **Development capacity** | More code · Mid code · Low code | How much native UI and API work the merchant can do. **More code** = full PDP/PLP/Checkout widgets (e.g. Digikala). **Mid code** = hosted components + merchant shell. **Low code** = redirect / embedded hosted checkout. |
| **Trust relationship** | Trusted · Untrusted | **Trusted:** Digipay returns numeric balances (1pay, 4pay, wallet, debt) for personalized UX. **Untrusted:** merchant sends basket amount; Digipay returns only **enough / not-enough** per method—no raw figures. |
| **Amount finalization** | At checkout · After checkout | **At checkout:** fixed basket (retail)—charge on confirm. **After checkout:** variable final amount (services, ride-hailing)—**authorize → capture** after service completes. |

**More-code sub-segments (prototyped):** Trusted/Untrusted × At/After = 4 variants in wireframes and final UI.

---

#### 2. Conventions of payment methods

| Method | Role | Rules (from discovery) |
|--------|------|-------------------------|
| **Wallet** | Debit + cashback | **OTP** required; **IPG uplift** allowed when balance &lt; basket (partial wallet + IPG). |
| **1pay** | Frictionless credit | **No OTP**; **no IPG uplift**—all-or-nothing within 1pay balance; good for “fast buy” from PDP. |
| **4pay** | Installment credit | **OTP** required; **IPG uplift** via server formula: `creditPayment = min(A, B×0.75)`, `debitPayment = A − creditPayment`. Schedule from API—not `A÷4` on client. |
| **SnappPay / Tara / IPG** | Alternatives | Must appear at **checkout for all dev tiers** to avoid dead-ends when Digipay credit is locked or insufficient. |

**User levels** (resolved via `GetUserLevel` on every page):

| Level | State | UX goal |
|-------|--------|---------|
| **L0** | No credit | Motivate **credit application**; lock 4pay/1pay at checkout until apply. |
| **L1** | Has credit | Motivate **best Digipay method**; show 4pay breakdown before commit. |
| **L2** | Has debt | Motivate **repayment** first; 4pay/1pay locked until repay; **wallet still allowed**. |

---

#### 3. Merchant needs in each segment

| Need | Summary |
|------|---------|
| **Journey coverage** | Surface Digipay on **PDP** (convert), **PLP** (scale badges/teasers), **Checkout** (full method list + pay). |
| **Level-aware UX** | Different CTAs and locks per L0/L1/L2; preserve cart/context through apply and repay redirects. |
| **Trust-aware API** | Trusted merchants personalize with balances; untrusted merchants only show eligibility labels. |
| **Finalization-aware flow** | Retail: confirm at checkout. Services: authorize at start, capture when amount is final; support adjust/void. |
| **Transparency** | Klarna-style installment timeline **before** payment; server-authoritative amounts and due dates. |
| **Reliability** | Idempotent checkout session; webhooks as source of truth; recoverable checkout after refresh. |
| **By dev capacity** | **More:** native widgets + full service orchestration. **Mid/Low:** fewer endpoints, hosted checkout (Phase 2). |

---

### Solution space

| # | Deliverable | Status | Artifact |
|---|-------------|--------|----------|
| 1 | UX per merchant segment | **Partial** — More-code (4 sub-segments) done | [wireframe-morecode-fa.html](wireframe-morecode-fa.html) · [merchant-ui.html](merchant-ui.html) |
| 2 | Services per segment | **Partial** — matrix + More-code trace | [service-catalog-matrix.html](service-catalog-matrix.html) |
| 3 | Product document (logic, limitations, design) | **Done** (v1) | [product-discovery-handbook.html](product-discovery-handbook.html) |
| 4 | Technical integration document | **Partial** — More-code services | [digipay-services-docs.html](digipay-services-docs.html) |

---

## Project artifacts

| File | Purpose |
|------|---------|
| [product-discovery-handbook.html](product-discovery-handbook.html) | PM / system designer handbook: segmentation, payment rules, levels, policies, limitations, open questions |
| [service-catalog-matrix.html](service-catalog-matrix.html) | Service × page × level × trust × finalization matrix with filters |
| [wireframe-morecode-fa.html](wireframe-morecode-fa.html) | Low-fi Farsi wireframes (Digikala theme, More-code) |
| [merchant-ui.html](merchant-ui.html) | Interactive final UI + per-interaction service trace |
| [digipay-services-docs.html](digipay-services-docs.html) | API contracts, sequences, PM/eng notes per service |

---

## Remaining work (to close discovery)

- [ ] Mid-code and Low-code UX prototypes
- [ ] Formal segment-merge sign-off table
- [ ] OpenAPI / error catalog / webhook matrix in technical docs
- [x] OQ-1: Wallet allowed at L2 when user has credit debt
- [x] OQ-3: No partial capture — capture full final amount only
- [x] OQ-2: Lock at authorize — honor eligibility at capture; do not re-check user level
