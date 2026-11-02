# 03 — Platform & Integrations (outside the two apps)

Back to [README](README.md).

These components are invisible to end users, but the product does not work without them. The **integration contract** (BR-40)
is effectively a third product: it is what partner companies build against, and should be specified as OpenAPI + AsyncAPI documents of its own.

---

## BR-40 — Published integration contract · MVP

**Why:** Each new service must plug in by configuration and a standard contract, with no custom code on our side.

**Contract surface v1** (names are indicative)

| Direction | Endpoint / event | Purpose |
|---|---|---|
| Service implements → we call | `POST /enrollments` | Mode A: create the user's account at the service |
| Service implements → we call | `GET /preview?user={pairwiseId}` | Return preview data in a template schema (BR-06) |
| Service implements → we call | `GET /health` | Optional liveness for the health monitor |
| Service implements → we call | Webhook receiver | Lifecycle events (BR-43) |
| We implement → service calls | `POST /handoff/token` | Redeem a handoff code (BR-07) |
| We implement → service calls | `POST /notifications` | Send push / in-app to a user (BR-42) |
| We implement → service calls | `PUT /status` | Declare an outage or maintenance (BR-42) |
| We implement → service calls | `POST /enrollments/{id}/events` | `setup.completed`, `setup.failed`, `enrollment.completed` (Mode C), `enrollment.revoked` |
| We implement → service calls | `POST /previews/{user}/invalidate` or push the payload | Preview changed (BR-06 AC-06.2.3) |

**Acceptance criteria**
- **AC-40.1** The contract is published as OpenAPI 3.1 (REST) and AsyncAPI (webhooks), with examples, error codes and a changelog.
- **AC-40.2** The contract is versioned (`/v1/…`). Breaking changes require a new major version and at least 6 months of parallel support.
- **AC-40.3** All calls in both directions are authenticated. Service→platform uses OAuth 2.1 client credentials (private_key_jwt preferred). Platform→service requests are signed (JWS or HTTP Message Signatures) and the service can verify them with our published JWKS.
- **AC-40.4** All write endpoints accept an `Idempotency-Key`.
- **AC-40.5** A **conformance test suite** (run from the developer portal) checks a partner's implementation end to end against the sandbox and produces a pass/fail report.
- **AC-40.6** Server SDK helpers (at least Node and one JVM/.NET option) cover handoff redemption, signature verification and calling the notification API.

---

## BR-41 — Identity brokering · MVP

**Acceptance criteria**
- **AC-41.1** The identity broker is the OAuth 2.1 client of the primary IdP, and an OIDC provider to hosted services. The mobile app holds only *platform* tokens.
- **AC-41.2** Pairwise subject identifiers are issued per service (OIDC `subject_type=pairwise`) unless the tenant configures a shared identifier for a specific service.
- **AC-41.3** Only claims the user consented to (BR-05 AC-05.2.1) are released to a service, and every release is logged.
- **AC-41.4** Service tokens obtained in Mode B are encrypted at rest (KMS), refreshed by the backend, and never sent to the app.
- **AC-41.5** When the primary IdP revokes a user's session (back-channel logout), all platform sessions for that user end within 60 s and services receive `user.logged_out`.
- **AC-41.6** Supported primary IdP integration: any OAuth 2.1 / OIDC-compliant provider, configured per tenant (issuer, client, scopes, claim mapping).

---

## BR-42 — Inbound events from services (notifications, status, setup) · MVP

**Acceptance criteria**
- **AC-42.1** `POST /notifications` accepts the pairwise user ID (or a list, max N), category, channel (push / in-app / both), localised title and body, deep link, optional image, TTL, and priority. It returns a notification ID.
- **AC-42.2** The request is rejected (4xx with a clear reason) if: the user is not enrolled in the calling service, the category is not declared, or the rate limit is exceeded. A user's opt-out returns 202 with `suppressed: true`, so we don't reveal preference details beyond that flag.
- **AC-42.3** End-to-end latency from API accept to device push is 5 s or less at p95, excluding APNs/FCM delays outside our control.
- **AC-42.4** Delivery status (accepted, sent, delivered where available, opened) can be queried by notification ID and/or sent via webhook.
- **AC-42.5** `setup.completed` / `setup.failed` events trigger the tab state change (AC-04.1.3) and, optionally, a push using a template the service configured.
- **AC-42.6** `PUT /status` applies the availability overlay within 30 s on all online devices.

---

## BR-43 — Outbound lifecycle webhooks · MVP

**Acceptance criteria**
- **AC-43.1** Events: `user.enrolled`, `user.unlinked`, `user.logged_out`, `user.deleted`, `user.consent_changed`, `service.placement_changed`.
- **AC-43.2** Webhooks are signed, carry an event ID for de-duplication, and are retried with exponential backoff for up to 24 h. After that they go to a dead-letter queue the service owner can see and replay in the portal.
- **AC-43.3** `user.deleted` includes a deadline by which the service must delete or anonymise the data. The service confirms via the API, and compliance status is shown to the tenant admin.

---

## BR-44 — Web SDK / JS bridge · MVP

**Why:** Hosted websites need a few native behaviours. The bridge must be small, versioned and allow-listed. App store rules restrict exposing native APIs to downloaded code (Apple 4.7.2).

**Acceptance criteria**
- **AC-44.1** The bridge is delivered as a versioned npm package plus a CDN script, and works only on allow-listed origins for the open service.
- **AC-44.2** v1 methods: `goHome()`, `openService(key, path)`, `close()`, `requestReauth()`, `setTitle()`, `share(payload)`, `openExternal(url)`, `getContext()` (locale, theme tokens, safe-area insets, app version), `onAppEvent(handler)` (foreground/background, theme change).
- **AC-44.3** Calls not permitted for the service (per the portal) fail with a defined error, never silently.
- **AC-44.4** The bridge is feature-detectable, so the same website still works in a normal browser (graceful degradation).
- **AC-44.5** A web UI kit (CSS tokens) is published so hosted services can match the tenant's theme.

---

## BR-45 — White-label build and release pipeline · Next (manual/scripted in MVP)

**Acceptance criteria**
- **AC-45.1** Given a tenant's build config (BR-01 AC-01.1.1), the CI pipeline produces signed iOS and Android builds and uploads them to TestFlight / the Play internal track with no manual steps.
- **AC-45.2** Store metadata (listings, screenshots, privacy labels / data safety form) is generated from tenant config and kept in source control.
- **AC-45.3** A platform release can be rolled out to all flavors, or staged by tenant, from one command or dashboard. Per-flavor build status is visible.
- **AC-45.4** iOS apps are submitted under the **tenant's own Apple developer account**. Apple guideline 4.2.6 rejects template-generated apps submitted by the template provider. The pipeline supports per-tenant credentials stored in the vault.
- **AC-45.5** Target: a new tenant goes from config complete to store submission in 2 working days or less.

---

## BR-46 — Observability and support · MVP

**Acceptance criteria**
- **AC-46.1** Distributed tracing covers app → BFF → service calls (W3C `traceparent` is passed to services), searchable by user (internal ID) and service.
- **AC-46.2** A support tool lets platform admins look up a user, see their enrollments, tab states, recent notifications, handoff attempts and errors, without seeing service-private data.
- **AC-46.3** Crash and ANR reporting runs per flavor. Releases are tagged with app version and config version.
- **AC-46.4** On-call alerting covers: login failure rate, handoff failures, notification pipeline lag, config fetch errors, and per-service health thresholds.
