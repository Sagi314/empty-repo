# 04 — Non-Functional Requirements

Back to [README](README.md). All targets are initial proposals and should be validated with the team.

| ID | Area | Requirement |
|---|---|---|
| NFR-01 | **Startup performance** | Cold start to an interactive home (cached) in 2.0 s or less at p75 on reference mid-range devices (to be defined, e.g. Pixel 6a, iPhone 12). Warm start in 1.0 s or less |
| NFR-02 | **Smoothness** | 60 fps (120 on ProMotion where supported) for tab swipes, drawer and transitions. Fewer than 1% dropped frames in the scripted UI benchmark run in CI |
| NFR-03 | **Availability** | Platform core (login, config, handoff, notifications) 99.9% monthly. The app degrades gracefully when the backend is unreachable (cached home, clear offline banner) |
| NFR-04 | **Scalability** | Design for X tenants × Y MAU each (to be defined by business). The notification pipeline handles bursts of 1M messages in 5 minutes or less |
| NFR-05 | **Security** | OWASP MASVS L2 for the mobile app, OWASP ASVS L2 for the portal and APIs. Annual third-party pentest. Certificate pinning on platform APIs (with a rotation plan). Root/jailbreak detection that is configurable per tenant |
| NFR-06 | **Privacy / compliance** | GDPR (and local equivalents, e.g. Israeli Privacy Protection Law, if applicable). Data minimisation via pairwise IDs and claim consent. Data residency configurable per tenant (to be decided). Apple privacy manifest and labels, Google data safety form |
| NFR-07 | **Localisation** | Full i18n of the app and portal content. RTL layouts (Hebrew/Arabic) supported from day one. Locale passed to services via the bridge and API headers |
| NFR-08 | **Accessibility** | WCAG 2.2 AA for portal and native screens. Dynamic type, VoiceOver/TalkBack labels, contrast checks (AC-21.4), reduce-motion support |
| NFR-09 | **Compatibility** | iOS current-2 major versions, Android 9+ (API 28+) or as agreed. System WebView / WKWebView versions per OS |
| NFR-10 | **Store compliance** | Comply with App Store Review Guidelines 4.2 (minimum functionality), 4.2.6 (template apps), 4.7 (mini apps / HTML5 content), 3.1.1 (in-app purchase for digital goods sold inside services), 5.1.1(v) (account deletion), and the Google Play equivalents. Re-check before each release |
| NFR-11 | **Maintainability** | One codebase for all flavors. Config schema versioned with backward compatibility for app versions supported in the last 12 months. A forced-upgrade mechanism for critical issues |
| NFR-12 | **Auditability** | All admin actions, credential changes, consent grants/revocations and data releases are logged immutably for at least 1 year |
