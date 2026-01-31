# subscriptions-openfort

Subscription system using Openfort embedded wallets and session keys.

## Overview
This repo will become a reference implementation for subscription billing with session keys.
It focuses on clean UX (one approval for recurring charges) and a secure, auditable backend.

## Features (planned)
- Embedded wallets for new users
- Session key authorization with scoped permissions
- Subscription lifecycle (create, pause, cancel, renew)
- Webhook-driven state reconciliation
- Admin dashboard to review charges and failures

## Architecture
- Client: web app that creates user accounts and initiates subscriptions
- Server: API that mints/updates session keys and executes recurring charges
- Chain: smart account / embedded wallet via Openfort

## Quickstart (coming soon)
1) Create an Openfort project and get API keys.
2) Add environment variables.
3) Install dependencies and run the dev server.

## Environment variables (draft)
Create a `.env` file with:
- `OPENFORT_PUBLISHABLE_KEY`
- `OPENFORT_SECRET_KEY`
- `OPENFORT_PROJECT_ID`
- `CHAIN_ID`
- `RPC_URL`

## Docs
- Implementation details: `implementation.md`

## Status
Documentation is in progress. Code and runnable demo are next.
