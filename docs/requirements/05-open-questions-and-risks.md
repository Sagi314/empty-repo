# 05 — Open Questions & Risks

Back to [README](README.md). Each item needs an owner and a due date before MVP scope is frozen.

## Open questions

| ID | Question | Recommendation / notes | Owner | Due |
|---|---|---|---|---|
| Q-01 | "A primary app might have hosted apps": does this mean *ownership* (same org runs several services) or *visual nesting* (sub-tabs inside the primary's tab)? | Ownership is simpler and covers most needs. Nesting adds a second navigation level | | |
| Q-02 | Is the primary service always a tab, or only the identity provider? | Make it configurable (AC-03.1.1) | | |
| Q-03 | Which enrollment mode(s) do the first real partners need? | MVP has one mode, probably A (federated). Confirm with the first two partners | | |
| Q-04 | Can one hosted service be placed in several tenants (resold partner)? | Yes, design for it now (README §2). It affects the data model and the analytics split | | |
| Q-05 | Do services ever sell digital goods inside the webview? | If yes, Apple IAP rules apply (3.1.1). This needs legal and partner review | | |
| Q-06 | Who owns each Apple/Google developer account: us or the tenant? | It must be the tenant's for iOS (4.2.6). This affects onboarding and contracts | | |
| Q-07 | Should notifications ever be *pulled* from services, e.g. to backfill history at enrollment? | Default to push only. Optionally allow a one-time `GET /notifications/backfill` at enrollment | | |
| Q-08 | Does the platform-detected "unavailable" status show to users automatically, or only alert the owner? | Automatic "Degraded" overlay with a generic message, and owner override (AC-25.3) | | |
| Q-09 | Is there a cross-service search, home feed or wallet aggregation? | Out of scope for MVP. A natural "super-app" v2 feature | | |
| Q-10 | What are the commercial roles (who pays whom: tenant, partner, us)? | Affects portal billing and the analytics needed for revenue share | | |
| Q-11 | Data residency and hosting region requirements per tenant? | Affects infrastructure cost early; ask the first customers | | |
| Q-12 | Native framework choice (native vs Flutter vs React Native)? | Technical decision, but driven by NFR-02 (smoothness) and the web container quality | | |

## Risks

| ID | Risk | Impact | Mitigation |
|---|---|---|---|
| R-01 | App Store rejection as a "webview wrapper" (4.2) or a template app (4.2.6) | Launch blocked | Native home, previews and notifications give real native value. Submit under tenant accounts. Run an early TestFlight review with a pilot |
| R-02 | Partners integrate slowly or inconsistently | Empty tabs, poor UX | A strict contract, conformance suite, SDKs, sandbox. Start with one friendly partner as design partner |
| R-03 | Slow or unreliable partner preview APIs make the home feel sluggish | Engagement drop | Server-side aggregation, hard timeouts, caching, push-based preview updates (BR-06) |
| R-04 | Insecure handoff (codes leaked via logs or referrers) | Account takeover | Single-use 60 s codes bound to client and device, server-side redemption, `Referrer-Policy`, security review (BR-07) |
| R-05 | A config publish breaks the app for all users | Outage | Draft/preview/publish, schema validation, versioning, instant rollback (BR-24) |
| R-06 | Notification spam from partners | Uninstalls, push opt-outs | Per-service rate limits, declared categories, user controls, tenant frequency caps (BR-09, BR-42) |
| R-07 | Per-tenant builds multiply release effort | Ops cost grows linearly with tenants | Automate the pipeline early (BR-45). Keep everything else runtime-configurable |
