# PayPal Resolve

An AI agent that investigates a PayPal dispute and proves every claim against a real source before suggesting a response. If evidence is missing, it says so plainly instead of inventing proof.

Built for the PayPal "Build What's Next with PayPal and AI" hackathon, 2026.

## The problem

A customer disputes a payment on PayPal. The merchant has a deadline to respond, and the evidence they need is scattered across the order record, shipping data and customer messages. Most AI tools applied to this would just write a persuasive-sounding paragraph defending the merchant, whether or not the facts actually support it.

That's the wrong failure mode for a dispute response. A confident, well-written response built on an invented fact is worse than no response at all, since PayPal and the buyer can both check it.

## What this does differently

The evidence table is built by deterministic code, not by asking the AI model to decide what's true. Each claim is checked against an actual source: a real PayPal order, a shipping record, or a delivery confirmation. The status of each claim, verified or missing, comes from that lookup, never from the model's judgment.

The AI model (NVIDIA Nemotron, through Nebius Token Factory) is only used to draft the response text, and it is explicitly constrained: it may state as fact only what the evidence table marked verified, citing the source, and it must say plainly when something can't be proven rather than filling the gap with a plausible-sounding guess.

## What's real and what's labeled

- **Order creation**: a genuine call to PayPal's sandbox Orders API. You can click the button and see a real order ID come back from PayPal.
- **The dispute itself**: PayPal's sandbox gives a merchant no API to create a dispute directly, since real disputes only exist once an actual buyer files one through the checkout flow. This demo uses a clearly labeled simulated dispute scenario (`SIM-DISPUTE-0001`) so the investigation flow can be demonstrated end to end without requiring a live buyer interaction during every run.
- **Evidence submission**: a genuine call to PayPal's real sandbox Disputes API (`/v1/customer/disputes/{id}/send-message`). Against the simulated dispute ID, PayPal correctly returns a real 404, since that dispute doesn't exist on PayPal's side. Paste a real sandbox dispute ID (from an actual buyer-filed dispute) into the input field to see this call succeed for real.

Nothing here claims to be live when it isn't. The app tells you which parts are real API calls and which part is a labeled stand-in, the same way it tells you when evidence for a claim is missing.

## Stack

- **AI**: NVIDIA Nemotron 3 Nano 30B A3B, served through Nebius Token Factory's OpenAI-compatible API. Returns a native reasoning trace alongside its draft response, shown in the UI as an observation, not proof of why it wrote what it wrote.
- **Payments**: PayPal REST API (sandbox), OAuth2 client-credentials flow, Orders API and Disputes API.
- **Backend**: Node.js and Express. Keeps both the PayPal and Nebius credentials server side.
- **Frontend**: plain HTML, CSS and JavaScript, no framework.

## Setup

```bash
npm install
cp .env.example .env
# edit .env: add PAYPAL_CLIENT_ID and PAYPAL_CLIENT_SECRET from a PayPal sandbox app
# (developer.paypal.com/dashboard/applications/sandbox), and NEBIUS_API_KEY
# from a Nebius Token Factory account
npm start
```

Open `http://localhost:4174`. Create a real order, investigate the simulated dispute, read the evidence table and the drafted response, then try the real submission call.

## Project structure

```
server.js            - Express backend: PayPal OAuth, Orders API, Disputes API, evidence engine
public/index.html    - frontend: order creation, evidence table, draft response, submission
.env.example          - required environment variables
```

## Known limitations

- The dispute scenario is simulated, labeled clearly throughout the UI and this README, because PayPal's sandbox has no merchant-side API to self-create a real dispute.
- The evidence sources (shipping record, delivery confirmation) are hardcoded for this one demo scenario rather than pulled from a real logistics or CRM system. A production version would connect these to whatever system a merchant actually uses.
- There's no persistence. Each investigation runs fresh, nothing is saved to a database.
- The claim-by-claim evidence table currently covers one fixed scenario (item not received). A general version would need a broader library of claim types per dispute reason code.

## What's next, beyond hackathon scope

- Connect the evidence sources to real systems (a shipping API, a CRM, an email inbox) instead of hardcoded demo data
- Support more PayPal dispute reason codes, each with their own relevant evidence checklist
- Persist every investigation and its outcome as an audit log, so a merchant can see what was claimed, what was provable, and what PayPal decided
