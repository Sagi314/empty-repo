# 02 — Management Portal (web)

Back to [README](README.md).

The portal is used by three kinds of organisation. Every requirement below is scoped by these roles:

| Role | Scope | Typical user |
|---|---|---|
| **Platform admin** | All tenants | Us (ops, support, onboarding) |
| **Tenant admin / editor** | One tenant: branding, which services appear, menu, broadcasts | The customer who bought the app |
| **Service owner admin / editor** | One hosted service, across the tenants it's placed in | The company running the hosted website |

---

## BR-20 — Multi-tenant access control · MVP

- **US-20.1** As a tenant admin, I want to invite my team with specific roles, so that the right people can edit the right things.
- **US-20.2** As a service owner, I want to manage my service without seeing other tenants' or services' data.

**Acceptance criteria**
- **AC-20.1.1** Portal login uses SSO (OIDC) or email + MFA. MFA is mandatory for admin roles.
- **AC-20.1.2** Admins can invite, change role and remove users in their scope. Roles: Admin, Editor (draft only), Publisher (can publish), Viewer.
- **AC-20.2.1** Given a service owner user, when they call any portal API for another service or tenant, then they get 404/403, and this is covered by automated authorisation tests.
- **AC-20.2.2** A service placed in multiple tenants is configured once. Per-tenant overrides (e.g. zero-state copy) are possible only if the tenant allows them.

---

## BR-21 — Branding and global app settings · MVP

**Acceptance criteria**
- **AC-21.1** Tenant admins can edit theme tokens (colours, fonts from an approved list, corner radius, light/dark), logo, splash artwork and app-wide copy, with a live phone-frame preview.
- **AC-21.2** Build-time assets (app icon, app name, bundle ID) are shown read-only, with a "request store update" action, because they need a new store release (BR-45).
- **AC-21.3** Global settings: supported languages, default language, session length, biometric availability, push pre-prompt trigger, support contact, links to legal documents.
- **AC-21.4** Contrast is validated: colour combinations that fail WCAG AA show a warning before publishing.

---

## BR-22 — Service configuration · MVP

This is the core of the portal. One screen or wizard per service, with sections that mirror the tab states.

**Acceptance criteria (configuration checklist)**

*Identity and listing*
- [ ] **AC-22.1** Service key (immutable), display name, icon, accent colour, short description, category, owner contacts.

*Zero state*
- [ ] **AC-22.2** Hero media (image/Lottie/video, size limits enforced), title, body (rich text, localised), benefit bullets, CTA label, terms URL. There is a preview for every supported language.

*Enrollment*
- [ ] **AC-22.3** Enrollment mode (A Federated / B OAuth linking / C Web onboarding; see BR-05) with the matching fields:
  - A: enrollment endpoint URL, requested claims (chosen from the tenant's available claims), consent text.
  - B: authorization endpoint, token endpoint, client ID/secret (write-only, stored in a secrets vault), scopes.
  - C: onboarding URL, allowed domains.
- [ ] **AC-22.4** A "Test enrollment" button runs the flow against the sandbox tenant and shows a pass/fail log.

*Setup state*
- [ ] **AC-22.5** "Requires setup" toggle, message, estimated duration, timeout, "allow access during setup" toggle, and support info shown on failure.

*Preview*
- [ ] **AC-22.6** Template choice (BR-06), preview endpoint URL, freshness/cache TTL, timeout, the masked-by-default flag, fallback content. A **live test** with a sandbox user renders the actual card.

*Launch*
- [ ] **AC-22.7** Launch URL, allow-listed domains, native permissions requested (camera, location, files), session keep-alive time, pull-to-refresh on/off.

*Notifications*
- [ ] **AC-22.8** Declared notification categories (name, description, default on/off, urgent, user can disable yes/no) and rate limits per user per day.

*Credentials*
- [ ] **AC-22.9** Generate and rotate Partner API credentials and webhook signing secrets. Two secrets can be active at once to allow zero-downtime rotation. Secrets are shown once.
- [ ] **AC-22.10** Webhook endpoint URL plus the events subscribed, with a delivery log and replay.

*Validation*
- [ ] **AC-22.11** Publishing is blocked until required fields for the chosen modes are valid and every URL is HTTPS.

---

## BR-23 — Service placement and side-menu builder · MVP

- **US-23.1** As a tenant admin, I want to choose which services appear and in what order.
- **US-23.2** As a tenant admin, I want to build the side menu with links to native screens and pages inside services.

**Acceptance criteria**
- **AC-23.1.1** Tenant admins can add services from the catalog of services available to them, reorder tabs by drag and drop, and set eligibility rules per service (claims-based, e.g. `segment in [gold, platinum]`).
- **AC-23.2.1** The menu builder supports sections, items, icons, localised labels and drag-and-drop ordering.
- **AC-23.2.2** Item targets: a native screen (from a list), or service + path. Service owners can publish **named deep-link targets** (e.g. "Top up", "My cards"), so tenant admins pick from a list instead of typing paths.
- **AC-23.2.3** Visibility rules per item: always / enrolled in service X / not enrolled / eligibility rule.
- **AC-23.2.4** Given a service owner removes a named deep-link target that is used in a menu, then the tenant admin is warned and the item is hidden in the app until it is fixed.

---

## BR-24 — Draft, preview, publish, rollback · MVP

**Why:** Configuration *is* the product. A bad publish breaks the app for every user, so changes need a safety net.

**Acceptance criteria**
- **AC-24.1** All edits are saved as a **draft**. The live config is unaffected until it is published.
- **AC-24.2** **Preview on device:** a QR code opens the real app (production or staging flavor) in preview mode with the draft config, for internal test users only.
- **AC-24.3** Publishing creates an immutable, numbered config version with author, timestamp and diff. Publishing needs the Publisher role. Optional four-eyes approval is configurable per tenant.
- **AC-24.4** **Rollback** to any previous version takes one action and reaches devices within 5 minutes (on next config fetch / push signal).
- **AC-24.5** Optional scheduled publish (e.g. a campaign at 09:00 on a date).
- **AC-24.6** The config schema is versioned. The backend serves each app version a config it understands, so old app versions never break.

---

## BR-25 — Service status, maintenance and health · MVP

**Acceptance criteria**
- **AC-25.1** Service owners can set the status (Operational / Degraded / Unavailable / Maintenance) with a localised message and expected end time, either immediately or scheduled. The same is possible via the Partner API (BR-42).
- **AC-25.2** A health dashboard per service shows preview success rate, latency (p50/p95), handoff redemption failures, webhook delivery failures and the platform-detected status, for the last 24 h / 7 d.
- **AC-25.3** Given platform-detected failures exceed a threshold (e.g. preview errors above 50% over 5 min), then the platform shows the Degraded/Unavailable overlay automatically and alerts the service owner's contacts by email/webhook. When health has recovered for N minutes, the overlay clears automatically.
- **AC-25.4** Platform admins can override the status of any service (kill switch).

---

## BR-26 — Tenant broadcasts and templates · Next

**Acceptance criteria**
- **AC-26.1** Tenant admins can compose a notification (push and/or in-app) with a deep link, target an audience (all, enrolled in service X, eligibility segment), schedule it, and send a test to themselves.
- **AC-26.2** Broadcasts respect user preferences and quiet hours, and a frequency cap set by the tenant.
- **AC-26.3** Delivery report: sent, delivered, opened, deep-link conversions.

---

## BR-27 — Analytics and audit · Next

**Acceptance criteria**
- **AC-27.1** Tenant dashboard: MAU/DAU, sessions, retention cohorts, funnel per service (zero-state view → CTA → enrolled → first open), notification engagement.
- **AC-27.2** Service owner dashboard: the same funnel and engagement figures for their service, in every tenant they're placed in, with no PII.
- **AC-27.3** Audit log of every portal change and credential operation (who, what, when, before/after), exportable, kept for at least 1 year.

---

## BR-28 — Partner self-onboarding · Next

**Acceptance criteria**
- **AC-28.1** A service owner can sign up (by invitation), create a service in a **sandbox tenant**, get test credentials and test users, and run the conformance suite (BR-40) without our engineers.
- **AC-28.2** Go-live checklist: conformance passed, security questionnaire submitted, contacts set, tenant admin approval. A platform admin can then promote the service to production.
