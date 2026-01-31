# Implementation Guide: Subscription System with Openfort Session Keys

This document specifies how the subscription system should work, how session keys are used, and what needs to be implemented. It is intentionally detailed so others can fork and build quickly.

## Goals
- One-time user approval for recurring subscription charges
- Clear, auditable billing flow
- Safe defaults (principle of least privilege)
- Easy to clone, configure, and extend

## Non-goals
- Full production billing stack on day one
- Handling every edge case for every chain
- Complex enterprise invoicing

## High-level flow
1) User signs up and gets an embedded wallet.
2) User selects a plan and authorizes a session key.
3) Backend stores a subscription and schedules charges.
4) Recurring job uses the session key to execute the charge.
5) Webhooks reconcile chain state with the database.

## System components
### Client (web)
- Auth (email or OAuth)
- Plan selection and checkout UI
- Wallet status and billing history
- Session key authorization prompt (one time)

### Server (API)
- Auth + session management
- Subscription CRUD
- Session key creation + rotation
- Charge execution (scheduled)
- Webhook handler and reconciliation

### Data store
Tables/collections:
- users
- wallets
- session_keys
- subscriptions
- invoices (or charges)
- webhook_events

### Chain
- Openfort embedded wallet
- Session keys with scoped permissions

## Core data model
### users
- id, email, created_at

### wallets
- id, user_id, address, chain_id, created_at

### session_keys
- id, wallet_id, public_key, scopes, expires_at, revoked_at

### subscriptions
- id, user_id, plan_id, status, next_charge_at, created_at, canceled_at

### charges
- id, subscription_id, amount, currency, status, tx_hash, created_at

### webhook_events
- id, provider, event_type, payload, received_at, processed_at

## Subscription lifecycle
### Create
- User chooses plan.
- Server creates a session key with restricted scope:
  - Only the subscription token/contract
  - Only charge function
  - Max amount per charge
  - Validity window (e.g. 30–90 days)
- Subscription stored with `status=active` and `next_charge_at`.

### Renew / charge
- Scheduled job finds due subscriptions.
- For each, server uses session key to execute charge.
- Store charge record and tx hash.
- Update `next_charge_at` if successful.

### Pause
- Mark subscription `paused`.
- Skip future charges until resumed.

### Cancel
- Mark subscription `canceled`.
- Revoke session key.

### Failed charge
- Record failure reason.
- Retry policy: 3 attempts, 24h apart (configurable).
- If retries fail, set `status=past_due`.

## Session key policy
### Scoping
- Allow only the minimal set of contract methods.
- Restrict max value per transaction.
- Restrict validity to reduce blast radius.

### Rotation
- Rotate keys before expiry.
- Revoke old keys after rotation.
- Keep audit trail of rotations.

### Revocation
- On cancel or suspected compromise, revoke immediately.
- Store `revoked_at` and prevent use in API.

## Security considerations
- Never expose secret keys to the client.
- Log and rate-limit charge endpoints.
- Validate webhook signatures.
- Use idempotency keys for charge attempts.
- Encrypt sensitive data at rest (session key metadata).

## Webhooks & reconciliation
### Webhooks
Expected events:
- Charge succeeded
- Charge failed
- Wallet events (optional)

### Reconciliation
- For each webhook event, update charge status.
- If a tx hash appears without a matching charge, create a reconciliation record.

## API endpoints (draft)
- `POST /api/subscriptions` create subscription + session key
- `POST /api/subscriptions/:id/pause`
- `POST /api/subscriptions/:id/cancel`
- `GET /api/subscriptions` list user subscriptions
- `POST /api/charges/run` internal cron endpoint
- `POST /api/webhooks/openfort`

## Cron / scheduler
Run every hour:
- Select subscriptions where `next_charge_at <= now` and `status=active`
- Execute charges and update records

## Testing strategy
- Unit tests: session key creation, scoping rules
- Integration tests: end-to-end subscription flow
- Webhook tests: signature validation + idempotency
- Failure tests: retries, expirations, and revocations

## Observability
- Structured logs for each charge attempt
- Metrics: charge success rate, retries, webhook lag
- Alerts for repeated failures or webhook backlog

## Deployment notes
- API server requires access to Openfort secret keys.
- Cron runner should run in the same environment.
- Webhook endpoint must be publicly reachable.

## Milestones
1) Scaffolding + basic UI
2) Session key issuance
3) Subscription create/charge
4) Webhook reconciliation
5) Demo video + docs polish
