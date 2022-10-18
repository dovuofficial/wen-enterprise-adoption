# Integrating with DOVU Market 🕊
 
In this document you learn how you can work with DOVU to support the purchase of fully auditable carbon credits to your clients that have been created through DOVU's audit trail.

> Please note, this is evolving documentation and may be subject to change, we will always take "best effort" precautions for the communication of the backwards compatibility of systems.

Below, we will describe the basic flow of what you can expect it will follow with example pay loads of data packets that your systems can receive to keep track of your clients purchasing carbon credits through the [DOVU marketplace](https://app.dovu.market/).

## Your interaction with DOVU, a flow.

As fully-auditable carbon credits are a highly limited resource we will work with whitelisted clients on a one-on-one basis to ensure that we can reserve enough carbon for qualifying usecases.

```mermaid
sequenceDiagram
    participant DOVU
    participant Integrator
    loop Business Development
        Integrator-->>DOVU: Requirements of proposed usecase
        DOVU-->>Integrator: Descision of allocation with contract
        Integrator-->>DOVU: Acceptanace/Deny or legal redlines
        DOVU-->>Integrator: Reserve amount of carbon
    end

    DOVU->>Integrator: Assign HMAC secret and market identifier
    Integrator->>Customer: Display dynamic DOVU button 
    Customer->>DOVU: Purchase of carbon from DOVU marketplace
    DOVU->>Customer: On purchase, redirect back to frontend
    DOVU-->>Integrator: Webhook payload of carbon purchase
```

## Starting an integration

You should provide these items to DOVU before starting an integration:

1. Company name
2. DOVU Market Webhook Endpoint (we will test this with a "hello world" request)
3. Redirect Endpoint (Redirecting back to a SaaS platform)

You will receive these following items when you start integrating with DOVU.

1. Marketplace Identifier
2. HMAC Secret Key

Your marketplace identifier allows us to capture a purchasing connect it to your application, You should combine all interactions with the DOVU marketplace with a **ref** to identify a given customer for your incoming webhook.

## Example Button/Structure

We will provide example brand assets of how you could integrate a DOVU offset button on your service, currently you are Free to use whatever styling you wish that we would prefer that you add the DOVU logo. 

The button and link will be dynamic, And it will display to a consumer how much carbon they need to offset.

```
https://dovu.market/{client-identifier}?amount={carbon-amount}&ref={customer-reference}
```

## After purchase flow

The market identifier will be stored within the cookies of a given Consumer, this will provide them access to your reserved carbon, when they complete a purchase they can be redirected back to your platform, and a corresponding webhook will be sent to your system.

### Example webhook payload

Below is the example payload with a specification of what your server will receive. 

It is ultimately up to you (but highly recommended) whether you wish to include the HMAC verification, using your **HMAC Secret Key** provided earlier to decode the message and verify DOVU is the correct originator.

Please follow our documentation for [Guardian security on how to construct and deal with HMAC](https://github.com/dovuofficial/guardian-middleware-api#security) you can also follow our implementation of [validation of HMAC signatures](https://github.com/dovuofficial/guardian-middleware-api/blob/main/src/utils/hmac.ts).  

#### Headers

This below signature is using the **HMAC Secret Key** of *"secret"*, so you will be able to test and compare generated signatures with the example payload. 

```json
{
  "x-signature": 'ZdSc2naWh4+hjSsJ24LxeCVIZSRzL+QWG9P+HO7ETaE='
}
```

#### Payload 

The payload is constructed of four items:

- context: A duplication of your client reference.
- reference: A reference to your customer that is offset carbon through the marketplace, will be null if no ref is provided.
- retirement-tx: This identifier is a state proof ready string that links back to Hedera transaction.
- reserve-remaining-kg: This provides a figure for the remaining kilograms that have been reserved for you after retirement.

```json
{
  "data": {
    "context": "client-identifier",
    "reference": "customer-ref",
    "retirement-tx": "0.0.1156-1663839551-50378818",
    "reserve-remaining-kg": 2000
  }
}
```

### Stateproof Generation

DOVU uses state proofs to indicate when a particular purchase of carbon has been retired as a fixed point in time, this is simply a transfer. We then generate the certificates which customers can view from their DOVU dashboard. 

Upon receiving and validating the webhook you can generate a state proof through the above **retirement-tx** using this endpoint below, on testnet. These state proofs can be large, So the onus is on the integrator to have a GET request to the mirrornode's stateproof endpoint. 

```
https://testnet.mirrornode.hedera.com/api/v1/transactions/0.0.1156-1663839551-50378818/stateproof
```

## Final Words

Currently, our B2B integration features are in constant development, We will happily consider adding additional features and APIs to support your business requirements just give us a shout! 