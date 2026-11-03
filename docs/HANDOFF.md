# Handoff: Club Super-App planning session

Written 2026-11-03 at the end of a cloud planning session, so a local Claude Code session can continue with the same context.

## 1. Read these first

| What | Where | Notes |
|---|---|---|
| **Requirements doc (source of truth)** | https://claude.ai/artifact/8HUGyEg8U4nbHXNxAnfNqg | A Claude Docs doc. Read it with the **Claude Docs connector** tools (`read`, `query`, `update`), never by web fetch. Doc (project) id: `3afcae12-1e5f-46b3-976e-cd689e75619b` |
| Time Clock doc (sister project, the template for project shape) | https://claude.ai/artifact/BNmhq5Ydmj1vEg2oxty3Qb | Doc id `54058e38-ee8a-43c7-9102-a31b9675aef0`. The "Technical architecture" tab (body node `d5b3180c-b833`) defines the monorepo layout we copied |
| Old markdown copy of the requirements | `docs/requirements/*.md` in GitHub `sagi314/empty-repo`, branch `ccr-6c8114de-xyp3ps` | **Outdated.** Written before the security, roles, notifications and tech-stack work. Use the doc instead |

How to read the doc with the connector: `read(ref={"object":"project","id":"3afcae12-1e5f-46b3-976e-cd689e75619b"})` lists the tabs. Then read each tab's body node with `engine:"prose"` and `container={"kind":"project","id":"3afcae12-…"}`. Use `payload={"kind":"view","outline":true}` first for long tabs. If the connector isn't available locally, ask the user to export the doc to Markdown (the doc's export menu) and drop it into the project folder.

Tabs of the requirements doc:

| Tab | Body node id | Holds |
|---|---|---|
| Overview | `507c4e87-da24` | How to read, glossary, components, BR summary table with Priority dropdowns, MVP cut |
| Decisions and open questions | `61b4f9c6-933b` | D-01 to D-15, Q-01 to Q-17 (with answers), risks R-01 to R-09 |
| Securing the platform API | `99a88e26-fcad` | Identity broker answer, sequence diagram, endpoint protection, BR-41 acceptance criteria |
| Mobile app | `09ed789a-2213` | BR-01 to BR-12 with stories and acceptance criteria, tab state diagram |
| Management portal | `99373735-4933` | BR-20 to BR-28 |
| Platform and integrations | `2ca4087e-e320` | BR-40, BR-42 to BR-46, "How a notification finds the user" |
| Technical architecture | `34a735e8-e87a` | Stack, monorepo layout, modules, tech decisions T1 to T12, spikes, Time Clock integration checklist |
| Non-functional | `47b7997a-e516` | NFR-01 to NFR-12 |

Dropdown enums in the doc (use these ids for status chips): Decision status `af64285b-9f54` (Proposed / Agreed / Changed / Rejected), Question status `be0341ce-b5c5` (Open / Answered / Deferred), Priority `b9dba7a7-583d` (MVP / Next / Later). The user sets the decision statuses themselves; don't flip them without asking.

## 2. The product in one paragraph

A white-label "super-app" (club app). Each customer ("tenant") gets its own branded iOS/Android app (a "flavor"). Users sign in once through the tenant's **primary** service (OAuth 2.1 / OIDC). Then they see a swipeable, tabbed home of **hosted services**. Each tab has states: Zero (not joined), Enrolling, Setup pending, Setup failed, Active (native preview card), Re-auth required, plus an availability overlay (Unavailable / Maintenance). Opening a service shows its website in a webview, signed in through a one-time handoff code. The hamburger menu becomes a home button. There's a configurable side menu, push and in-app notifications from services, and a web management portal for tenant and service owners. No end-user website.

## 3. Decisions made (final state)

Architecture and security
- **Identity broker** issues our own tokens. The primary's tokens never reach the phone. Broker = **OpenIddict** server + client embedded in the C# API (Q-06). Access tokens 10 min, DPoP-bound, rotating refresh tokens. The API checks tokens locally.
- The **first primary is the user's own company**, so we add OpenID Connect to it ourselves (OpenIddict) with roles in the ID token (Q-05).
- **App attestation is out of the MVP** (Q-10). Tokens, DPoP and rate limits only.
- Handoff = OIDC-shaped one-time code, single use, 60 s or less, redeemed server-to-server.
- Previews are fetched server-side by the BFF and rendered natively from 5 templates (Balance, List, Progress, Card, Actions). Hard timeout 1.5 s, stale-while-revalidate.

Data
- **The platform stores no profile data** (no name, email, phone, address). It keeps pseudonymous IDs, **each user's latest roles**, enrollments, consent records and device push tokens (D-11). Inbox text is encrypted, kept at most 30 days, with an optional per-tenant pointer mode (D-13).
- **Pairwise ID per user per service**. Services address notifications by that ID; we resolve it to devices. An unknown or unlinked ID returns 410 Gone (D-12).

Roles (Q-16, Q-17, D-14, D-15)
- Each primary defines its own roles (the first: **employer** and **employee**, more may come). The primary sends a user's roles in the login token. **Our portal maps role → services** (e.g. employer: Wallet, Business; employee: Wallet, Time Clock, Benefits).
- A user can hold several roles (e.g. an owner who is also an employee of their own company). They see the **combined tabs, each service once**. **No company switcher in the MVP.** If roles span companies, the service picks the company itself (as Time Clock's company picker does).
- Roles inside a service (e.g. manager in Time Clock) stay with that service.

Integrations and notifications
- Enrollment modes: A federated one-tap, B OAuth account linking, C web onboarding. **MVP ships B, OAuth account linking**, because Time Clock already supports it (Q-03). A comes next.
- Services **push** notifications to our API; we never pull, not even a history backfill (Q-08).
- Platform-detected failures show an automatic "Degraded" banner that the owner can override (Q-09).
- "A primary with hosted apps" = ownership. A company can run several services, and each is a normal tab. A primary is a company that also implements the login interface and gets the branded app (Q-01, Q-02).
- No digital goods sold in-app, so no store in-app-purchase fees (Q-04).
- No backup login if a primary is down. Show a clear message (Q-15).

Stack and delivery
- **C# / ASP.NET Core (.NET 10 LTS), Flutter (mobile app + management portal as Flutter web), PostgreSQL + EF Core code-first, one monorepo.**
- Modular monolith with two executables: `SuperApp.Api` (app/portal/partner APIs, OpenIddict, handoff, SignalR hub) and `SuperApp.Worker` (Hangfire: push fan-out, webhooks with retries, health checks, retention).
- Modules: Identity, Tenancy, Catalog, Enrollment, Previews, Handoff, Notifications, ServiceHealth, Partners, Platform. Feature folders; modules talk only through `Contracts/`; architecture tests enforce this.
- Flutter: flutter_bloc Cubits, go_router, flutter_inappwebview (pending spike). Packages: api_client (generated from OpenAPI), ui_kit, app_core, web_container, preview_cards. Flavors generated from `tenants/<key>/flavor.json`.
- Push through FCM HTTP v1 for both platforms; real-time through SignalR.
- **Hosting: Azure Israel Central** (Q-13). **Repos and pipelines: Azure DevOps**, same as Time Clock (T12).
- Monorepo top level: `CLAUDE.md, backend/, frontend/, sdk/ (web-bridge TS, dotnet), contracts/ (OpenAPI + AsyncAPI, generated), tenants/, e2e/, infra/, docs/, pipelines/`.
- Store publishing: the **first app goes under our own Apple/Google accounts** (safe, because the first primary is our own company). Later clubs probably need their own accounts (Apple 4.2.6, risk R-01). Q-07 stays open for later clubs.

## 4. First hosted service: Time Clock

The sister project. Its Host API already matches our contract: OAuth connect = enrollment mode B, `/host/v1/me/summary` = preview, `/host/v1/handoff` = handoff, company-scoped tokens.

Recommended alignment, which the **user is telling the Time Clock team**:
1. The super-app uses its pairwise ID as Time Clock's "link ID".
2. Time Clock calls our partner API with **OAuth client credentials** instead of an HMAC-signed push to a host endpoint.

The super-app is Time Clock's "host app", so it owns Time Clock's host checklist: server-side OAuth connect, the connect flow opened in ASWebAuthenticationSession / Custom Tabs, the handoff in the webview, notifications, webview location permission, iOS app-bound domain for offline, and calling Time Clock's revoke endpoint on unlink. This checklist is on the Technical architecture tab.

## 5. Still open or pending

- Q-07 for clubs after the first (store accounts).
- Q-11 cross-service features: deferred, out of MVP.
- Q-12 who pays whom: not decided. The MVP only collects usage numbers.
- Spikes not yet run: Flutter webview + JS bridge, OpenIddict adding a primary at runtime + DPoP middleware, flavor build time, FCM sensitive push on iOS.
- **The user will send a UI draft.** An earlier clickable demo artifact was deleted at the user's request ("not what I wanted"). Don't build another demo until the draft arrives, and build from their draft.
- The old markdown copy in the GitHub repo is outdated (see section 1).

## 6. Suggested next steps (the user hasn't picked one yet)

1. Draft **integration contract v1** (OpenAPI + AsyncAPI): enrollment (OAuth linking), preview, handoff, notifications, status, setup events, lifecycle webhooks.
2. Break the MVP into **epics and story designs**, like Time Clock's "Story designs" tab. Start with a walking skeleton: sign in through the primary → role-based tabs → link Time Clock → preview → open signed in → receive a notification. The user has a story-build workflow from Time Clock (`build-story` skill) that can build approved stories.
3. Create the **monorepo skeleton** (layout above), ready to push to Azure DevOps.
4. Run the four spikes.

## 7. Working with this user

- Prefers plain language and examples. When a question was unclear to them, rephrasing it with a concrete example worked.
- Likes making decisions through multiple-choice questions, and comments in the doc.
- Thinks in terms of the product; asks "why does it matter?" for infrastructure questions. Explain the practical effect in a sentence or two.
- Keep decisions in the doc (Decisions and open questions tab) rather than only in chat.
- Don't build unrequested artifacts. Wait for their direction or draft.
