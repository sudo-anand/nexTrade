
# nexTrade

**A B2B marketplace that takes a deal from price request to protected payment, on a single platform.**

nexTrade lets sellers list products without a fixed price, lets buyers request a live quote, and lets both parties agree on custom terms, pay, and book the order without leaving the platform. It is built for Indian MSME trade, where pricing is negotiated, orders are often customised, and trust is the main barrier to closing deals online.

> Status: Under active development (MVP stage). Interfaces, schemas, and scope described below may change.

---

## Table of Contents

1. [Overview](#overview)
2. [The Problem](#the-problem)
3. [How nexTrade Works](#how-nextrade-works)
4. [Key Features](#key-features)
5. [Tech Stack](#tech-stack)
6. [System Architecture](#system-architecture)
7. [Getting Started](#getting-started)
8. [Environment Variables](#environment-variables)
9. [Project Structure](#project-structure)
10. [Order and Quote Lifecycle](#order-and-quote-lifecycle)
11. [Roadmap](#roadmap)
12. [Compliance Notes](#compliance-notes)
13. [Contributing](#contributing)
14. [License](#license)
15. [Contact](#contact)

---

## Overview

Existing Indian B2B platforms are largely lead marketplaces. They connect a buyer and a seller, then the deal moves off-platform to phone calls, messaging apps, and direct bank transfers. This leaves both sides without a record of what was agreed and without payment protection.

nexTrade is designed around a different model: a **request-for-quote (RFQ) to order** workflow.

- Sellers list products with a hidden, indicative, or fixed price.
- Buyers press **Get Latest Price** to request a current quote.
- Sellers are notified and respond with a time-bound, price-locked quote.
- Buyers and sellers negotiate and record agreed custom terms as **Deal Notes**.
- Buyers press **Pay and Book** to pay through a licensed payment partner, and the order is confirmed.
- Funds are released to the seller after delivery is confirmed.

## The Problem

| Stakeholder | Common problem today                                                                                                |
| ----------- | ------------------------------------------------------------------------------------------------------------------- |
| Sellers     | Pay upfront for leads with no guarantee of conversion; the same inquiry is often shared with many competing sellers |
| Buyers      | Receive multiple unsolicited calls, cannot compare quotes easily, and have little protection when paying in advance |
| Both        | Terms discussed on a call are not recorded, prices change after discussion, and disputes are hard to resolve        |

nexTrade addresses these through success-based pricing, limited seller matching, price lock, structured deal records, and protected payments.

## How nexTrade Works

1. **List.** A verified seller creates a product listing with specifications, minimum order quantity, photos, and optional pricing.
2. **Request.** A buyer submits an RFQ with quantity, delivery location, and requirements.
3. **Match.** The platform sends the RFQ to a small number of relevant, responsive sellers.
4. **Quote.** Each seller replies with price, taxes, delivery terms, lead time, and quote validity. The price is locked until expiry.
5. **Negotiate.** Buyer and seller communicate through in-app chat. Agreed custom terms are written into Deal Notes and accepted by both parties.
6. **Pay and Book.** The buyer pays an advance or the full amount. Funds are held by the payment partner and the order is booked.
7. **Fulfil.** The seller dispatches and uploads shipment details. The buyer confirms delivery.
8. **Settle.** Funds are released to the seller after commission, and a GST invoice is generated.

## Key Features

**Marketplace**

- Product listings without mandatory pricing
- Category-specific specification templates
- Bulk catalogue upload (Excel/CSV)
- Search-engine-friendly product, category, and city pages

**RFQ and Quotation**

- Get Latest Price request flow
- Limited matching (maximum number of sellers per RFQ is configurable)
- Time-bound quotes with price lock and version history
- Side-by-side quote comparison

**Deal Management**

- In-app chat with attachments
- Deal Notes for custom specifications, packaging, delivery, and inspection terms
- Terms locked once payment is made

**Payments and Orders**

- Pay and Book using a licensed payment aggregator or escrow partner
- Advance or full payment, with configurable release conditions
- GST-compliant invoicing and settlement statements
- Refund and partial-release support

**Trust and Safety**

- Seller verification: GSTIN, PAN, and bank account validation
- Reputation score based on response time, fulfilment, and disputes
- Reviews restricted to completed paid orders
- Dispute desk with evidence upload and audit trail

**Notifications**

- WhatsApp Business API, email, and web push
- Queue-based delivery with retries and fallbacks

**Administration**

- Seller verification queue
- Dispute management and manual refund or release
- Moderation and category management
- Commission rules per category

## Tech Stack

| Layer            | Technology                                                                       |
| ---------------- | -------------------------------------------------------------------------------- |
| Frontend         | React, TypeScript, Tailwind CSS (Next.js recommended for SEO pages)              |
| Backend          | Node.js, Express, TypeScript                                                     |
| Database         | MongoDB (Mongoose)                                                               |
| Cache and Queues | Redis, BullMQ                                                                    |
| Notifications    | WhatsApp Business API, SMTP/SES, Web Push                                        |
| Payments         | Licensed payment aggregator or escrow partner (for example Razorpay or Cashfree) |
| Storage          | S3-compatible object storage with CDN                                            |
| Authentication   | Phone OTP with JWT sessions                                                      |
| DevOps           | Docker, GitHub Actions                                                           |

## System Architecture

```
Client (Web / PWA)
        |
   API Gateway / Express API
        |
 +------+---------+----------------+-----------------+
 |                |                |                 |
Auth &         RFQ / Quote /     Orders &          Notification
Profiles       Deal Notes        Payments          Service (BullMQ)
 |                |                |                 |
 +------+---------+----------------+-----------------+
        |
   MongoDB      Redis       Object Storage
                               |
              External: Payment Partner, WhatsApp API, KYC/GST APIs
```

Design principles:

- Quotes are immutable after they are viewed; changes create a new version.
- Deal Notes are locked at the time of payment.
- Payment and settlement records are append-only.
- All notifications go through a queue with retry and fallback logic.

## Getting Started

### Prerequisites

- Node.js 20 or later
- npm or pnpm
- MongoDB 6 or later (local or Atlas)
- Redis 7 or later
- Docker (optional, for local services)

### Installation

```bash
git clone https://github.com/<your-username>/nextrade.git
cd nextrade

# Install dependencies
npm install

# Copy environment template
cp .env.example .env
```

### Run in development

```bash
# Start MongoDB and Redis (optional, via Docker)
docker compose up -d

# Start the API
npm run dev:server

# Start the web client
npm run dev:client
```

The API runs on `http://localhost:5000` and the client on `http://localhost:3000` by default.

### Run tests

```bash
npm run test
npm run lint
```

> Script names above are indicative. Update them to match the `package.json` scripts in this repository.

## Environment Variables

Create a `.env` file in the project root. Never commit real secrets.

```env
# Server
NODE_ENV=development
PORT=5000
CLIENT_URL=http://localhost:3000

# Database
MONGODB_URI=mongodb://localhost:27017/nextrade
REDIS_URL=redis://localhost:6379

# Auth
JWT_SECRET=change-me
JWT_EXPIRES_IN=7d
OTP_PROVIDER_API_KEY=

# Payments
PAYMENT_PROVIDER=razorpay
PAYMENT_KEY_ID=
PAYMENT_KEY_SECRET=
PAYMENT_WEBHOOK_SECRET=

# Notifications
WHATSAPP_API_URL=
WHATSAPP_API_TOKEN=
SMTP_HOST=
SMTP_USER=
SMTP_PASS=

# Verification (KYC / GST)
KYC_PROVIDER_API_KEY=

# Storage
STORAGE_BUCKET=
STORAGE_ACCESS_KEY=
STORAGE_SECRET_KEY=
```

## Project Structure

Planned layout (adjust as the codebase evolves):

```
nextrade/
  apps/
    web/                 Buyer and seller web application
    admin/               Admin dashboard
    api/                 REST API (Express, TypeScript)
  packages/
    shared/              Shared types, validators, constants
    ui/                  Shared UI components
  docs/                  PRD, API reference, architecture notes
  docker-compose.yml
  .env.example
  README.md
```

## Order and Quote Lifecycle

```
Requested -> Quoted -> Negotiating -> Terms Agreed -> Paid (Booked)
          -> Dispatched -> Delivered -> Settled -> Closed

Side exits: Expired, Declined, Cancelled, Disputed
```

| Status        | Description                                                 |
| ------------- | ----------------------------------------------------------- |
| Requested     | Buyer submitted an RFQ                                      |
| Quoted        | One or more sellers have responded with price-locked quotes |
| Negotiating   | Buyer and seller are discussing terms                       |
| Terms Agreed  | Both parties accepted the Deal Notes                        |
| Paid (Booked) | Payment received and held; order confirmed                  |
| Dispatched    | Seller shipped the goods and added tracking details         |
| Delivered     | Buyer confirmed delivery, or auto-confirmation applied      |
| Settled       | Funds released to seller; invoice generated                 |
| Closed        | Order complete and review window ended                      |

## Roadmap

**Phase 1: MVP**

- [ ] Authentication and seller verification
- [ ] Product catalogue and bulk upload
- [ ] RFQ, quote, and price-lock flow
- [ ] Chat and Deal Notes
- [ ] Pay and Book with payment partner
- [ ] Dispatch, delivery confirmation, and settlement
- [ ] GST invoicing
- [ ] Admin panel: verification and disputes

**Phase 2: Growth**

- [ ] Hindi and regional language support
- [ ] Masked calling and call summaries
- [ ] Price history and market trend insights
- [ ] Seller CRM and analytics tools
- [ ] Referral programme

**Phase 3: Scale**

- [ ] Logistics integrations
- [ ] Financing partnerships
- [ ] Mobile applications
- [ ] ERP and accounting integrations (for example Tally)

## Compliance Notes

nexTrade does not hold customer funds directly. Payment collection and escrow are handled through an RBI-authorised payment aggregator or escrow partner, in line with the Reserve Bank of India's payment aggregator regulations. Operators deploying this software are responsible for obtaining the required legal, tax (GST, TCS), and data protection (DPDP Act) advice for their jurisdiction and business model. Nothing in this repository constitutes legal or financial advice.

## Contributing

Contributions are welcome. To contribute:

1. Fork the repository.
2. Create a feature branch: `git checkout -b feature/short-description`.
3. Commit with clear messages following Conventional Commits (for example `feat: add quote versioning`).
4. Run linting and tests before pushing.
5. Open a pull request describing the change, motivation, and testing performed.

Please open an issue first for significant changes so the approach can be discussed.

## License

Distributed under the MIT License. See `LICENSE` for details. (Replace with the licence you choose before publishing.)

## Contact

Maintainer: Anand
Project: nexTrade
Repository: https://github.com/your-username/nextrade
Email: your-email@example.com
