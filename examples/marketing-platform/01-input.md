# Marketing Automation Platform — Input

## The Idea

A marketing automation platform where agencies manage contacts, design email campaigns, set up automations (drip sequences), and track performance (opens, clicks, conversions).

## Actors

- **Agency Admin** — manages users, billing, and platform configuration
- **Marketer** — manages contacts, creates campaigns, configures automations, reviews analytics
- **Viewer/Client** — views campaign reports and contact lists (read-only)

## Key Features

- Contact management with lists, tags, and segmentation
- Email campaign builder with templates, scheduling, and sending
- Automation workflows: trigger-based drip sequences and auto-responses
- Event tracking: opens, clicks, bounces, unsubscribes, conversions
- Analytics dashboards per campaign and aggregate
- Billing based on contact count and email volume

## Business Rules

- Campaigns can only be edited while in "draft" status
- Campaign status flow: draft → scheduled → sending → sent (no skipping)
- Failed sends are retryable without duplicating to already-sent contacts
- Automations can be paused and resumed without losing state
- Contacts can unsubscribe from specific lists or globally
- Billing enforces limits: sending beyond plan quota is blocked, not charged overage

## Constraints

- All timestamps in UTC
- Email sending is async — campaigns with 100k contacts don't block the UI
- Analytics events arrive with up to 5-minute delay (near-real-time, not real-time)
