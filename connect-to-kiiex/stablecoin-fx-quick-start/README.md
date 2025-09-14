---
description: >-
  This guide explains how to swap any stablecoin using our Kii API. It covers
  retrieving balances, checking the current exchange rate, sending the order and
  verifying execution.
---

# Stablecoin FX Quick Start

**Requirements to be able to trade stablecoins on Kii, via dashboard or API:**

* Have a user&#x20;
* Have an account level 1 at least (to upgrade from level 0 to level 1, you need to complete our KYC verification inside your user settings)&#x20;
* Have balances on any stablecoin

## Endpoints

**Staging URL :**&#x20;

```yaml
https://apstage.proxy.kiiex.io/ap
```

**Prod URL :**&#x20;

```yaml
https://alphaprod.proxy.kiiex.io/ap
```

## Swap Workflow example

For this example will assume a goal and specify what to do in order, also we'll provide you with all the endpoints related to each step. The goal is:

* **Swap USDT → COPM** using a market order.

### Prerequisites

These are the prerequisites for the step-by-step guide.

1. Create your Kii account:
   1. Go to [https://kiiex.io/](https://kiiex.io/) and click on login, and create your user
      1. You can also follow our guide on [set-up-your-kiiex-account](../set-up-your-kiiex-account/ "mention")
2. Update your account from level 0 to level 1 by following our KYC flow to be able to swap
3. Create your trader API Key on our panel to follow the next steps

### Step-by-step

These are the steps to fully create a swap and withdraw it:

1. Authenticate your user via api
   1. [#authenticate-your-user-via-api](./#authenticate-your-user-via-api "mention")
2. Create a USDT deposit. If you have already created a deposit, check your current balance
   1. [#create-deposit](./#create-deposit "mention")
   2. [#post-getaccountpositions](./#post-getaccountpositions "mention")
3. Get the current COPM/USDT rate
   1. [#get-current-rate](./#get-current-rate "mention")
4. Create market order (buy COPM)
   1. [#create-a-swap](./#create-a-swap "mention")
5. Verify order was filled
   1. [#get-swap-status](./#get-swap-status "mention")
6. Get trade details (swap info)
   1. [#post-gettradeshistory](./#post-gettradeshistory "mention")
7. Confirm the new COPM balance using the get account positions endpoint
   1. [#get-account-balances](./#get-account-balances "mention")
8. Create a COPM withdrawal, it could be to another wallet or to your bank account
   1. [#create-a-withdrawal](./#create-a-withdrawal "mention")
9. Check your COPM withdrawal status
   1. [#check-withdrawal-status](./#check-withdrawal-status "mention")

**How do I perform the swap workflow using the API?**

## API endpoints

### Authenticate your User via API

{% openapi-operation spec="swap-kiiex" path="/AuthenticateUser" method="post" %}
[OpenAPI swap-kiiex](https://gitbook-x-prod-openapi.4401d86825a13bf607936cc3a9f3897a.r2.cloudflarestorage.com/raw/35214ac23d9928896910cf1474dfa48f9fd7d2585cd1675c58f2332c1cf63386.txt?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=dce48141f43c0191a2ad043a6888781c%2F20250825%2Fauto%2Fs3%2Faws4_request&X-Amz-Date=20250825T191814Z&X-Amz-Expires=172800&X-Amz-Signature=558c98e789e7f205dbe4ae09bbaf5b65a0b6059dabdea4af748073b5d4cc9f86&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject)
{% endopenapi-operation %}

### Create Deposit

{% openapi-operation spec="swap-kiiex" path="/CreateDepositTicket" method="post" %}
[OpenAPI swap-kiiex](https://gitbook-x-prod-openapi.4401d86825a13bf607936cc3a9f3897a.r2.cloudflarestorage.com/raw/35214ac23d9928896910cf1474dfa48f9fd7d2585cd1675c58f2332c1cf63386.txt?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=dce48141f43c0191a2ad043a6888781c%2F20250825%2Fauto%2Fs3%2Faws4_request&X-Amz-Date=20250825T191814Z&X-Amz-Expires=172800&X-Amz-Signature=558c98e789e7f205dbe4ae09bbaf5b65a0b6059dabdea4af748073b5d4cc9f86&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject)
{% endopenapi-operation %}

### Get Account Balances

{% openapi-operation spec="swap-kiiex" path="/GetAccountPositions" method="post" %}
[OpenAPI swap-kiiex](https://gitbook-x-prod-openapi.4401d86825a13bf607936cc3a9f3897a.r2.cloudflarestorage.com/raw/35214ac23d9928896910cf1474dfa48f9fd7d2585cd1675c58f2332c1cf63386.txt?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=dce48141f43c0191a2ad043a6888781c%2F20250825%2Fauto%2Fs3%2Faws4_request&X-Amz-Date=20250825T191814Z&X-Amz-Expires=172800&X-Amz-Signature=558c98e789e7f205dbe4ae09bbaf5b65a0b6059dabdea4af748073b5d4cc9f86&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject)
{% endopenapi-operation %}

### Get Current Rate

{% openapi-operation spec="swap-kiiex" path="/GetLevel1SummaryMin" method="post" %}
[OpenAPI swap-kiiex](https://gitbook-x-prod-openapi.4401d86825a13bf607936cc3a9f3897a.r2.cloudflarestorage.com/raw/35214ac23d9928896910cf1474dfa48f9fd7d2585cd1675c58f2332c1cf63386.txt?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=dce48141f43c0191a2ad043a6888781c%2F20250825%2Fauto%2Fs3%2Faws4_request&X-Amz-Date=20250825T191814Z&X-Amz-Expires=172800&X-Amz-Signature=558c98e789e7f205dbe4ae09bbaf5b65a0b6059dabdea4af748073b5d4cc9f86&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject)
{% endopenapi-operation %}

### Create a SWAP

{% openapi-operation spec="swap-kiiex" path="/SendOrder" method="post" %}
[OpenAPI swap-kiiex](https://gitbook-x-prod-openapi.4401d86825a13bf607936cc3a9f3897a.r2.cloudflarestorage.com/raw/35214ac23d9928896910cf1474dfa48f9fd7d2585cd1675c58f2332c1cf63386.txt?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=dce48141f43c0191a2ad043a6888781c%2F20250825%2Fauto%2Fs3%2Faws4_request&X-Amz-Date=20250825T191814Z&X-Amz-Expires=172800&X-Amz-Signature=558c98e789e7f205dbe4ae09bbaf5b65a0b6059dabdea4af748073b5d4cc9f86&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject)
{% endopenapi-operation %}

### Get SWAP status

{% openapi-operation spec="swap-kiiex" path="/GetOrderStatus" method="post" %}
[OpenAPI swap-kiiex](https://gitbook-x-prod-openapi.4401d86825a13bf607936cc3a9f3897a.r2.cloudflarestorage.com/raw/35214ac23d9928896910cf1474dfa48f9fd7d2585cd1675c58f2332c1cf63386.txt?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=dce48141f43c0191a2ad043a6888781c%2F20250825%2Fauto%2Fs3%2Faws4_request&X-Amz-Date=20250825T191814Z&X-Amz-Expires=172800&X-Amz-Signature=558c98e789e7f205dbe4ae09bbaf5b65a0b6059dabdea4af748073b5d4cc9f86&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject)
{% endopenapi-operation %}

{% openapi-operation spec="swap-kiiex" path="/GetTradesHistory" method="post" %}
[OpenAPI swap-kiiex](https://gitbook-x-prod-openapi.4401d86825a13bf607936cc3a9f3897a.r2.cloudflarestorage.com/raw/35214ac23d9928896910cf1474dfa48f9fd7d2585cd1675c58f2332c1cf63386.txt?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=dce48141f43c0191a2ad043a6888781c%2F20250825%2Fauto%2Fs3%2Faws4_request&X-Amz-Date=20250825T191814Z&X-Amz-Expires=172800&X-Amz-Signature=558c98e789e7f205dbe4ae09bbaf5b65a0b6059dabdea4af748073b5d4cc9f86&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject)
{% endopenapi-operation %}

### Create a Withdrawal

{% openapi-operation spec="swap-kiiex" path="/CreateWithdrawTicket" method="post" %}
[OpenAPI swap-kiiex](https://gitbook-x-prod-openapi.4401d86825a13bf607936cc3a9f3897a.r2.cloudflarestorage.com/raw/35214ac23d9928896910cf1474dfa48f9fd7d2585cd1675c58f2332c1cf63386.txt?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=dce48141f43c0191a2ad043a6888781c%2F20250825%2Fauto%2Fs3%2Faws4_request&X-Amz-Date=20250825T191814Z&X-Amz-Expires=172800&X-Amz-Signature=558c98e789e7f205dbe4ae09bbaf5b65a0b6059dabdea4af748073b5d4cc9f86&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject)
{% endopenapi-operation %}

### Check Withdrawal status

{% openapi-operation spec="swap-kiiex" path="/GetWithdrawTickets" method="post" %}
[OpenAPI swap-kiiex](https://gitbook-x-prod-openapi.4401d86825a13bf607936cc3a9f3897a.r2.cloudflarestorage.com/raw/35214ac23d9928896910cf1474dfa48f9fd7d2585cd1675c58f2332c1cf63386.txt?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=dce48141f43c0191a2ad043a6888781c%2F20250825%2Fauto%2Fs3%2Faws4_request&X-Amz-Date=20250825T191814Z&X-Amz-Expires=172800&X-Amz-Signature=558c98e789e7f205dbe4ae09bbaf5b65a0b6059dabdea4af748073b5d4cc9f86&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject)
{% endopenapi-operation %}
