---
name: apple-navigation-deeplinks
description: >
  Implement and debug typed navigation, deep links, and Universal Links in modular Apple
  apps. Use for URL contracts, external entry routing, deferred navigation after startup
  or authentication, tab and scene coordination, and associated-domain validation.
---

# Apple Navigation and Deep Links

## Intended result

Convert a validated external request into the correct feature entry through explicit, testable routing. These conventions apply to new and existing apps; example domains, identifiers, and feature names are fictional.

Keep feature boundaries intact: `FooInterface` owns cross-module routes and starter values; `FooFeature` owns internal screens; the app constructs configurations and presents feature entries. For a new project, establish this split before connecting lifecycle callbacks. Do not force a particular internal architecture or a universal router abstraction.

## Routing responsibilities

Treat these as responsibilities, not mandatory classes or packages:

1. **Receive:** adapt a URL, user activity, notification tap, or system request at the app/scene boundary.
2. **Parse and validate:** check the supported scheme, exact host policy, path, and parameters; return a typed request without UI or network effects.
3. **Resolve:** determine the destination and any prerequisite such as authentication, session readiness, or entity lookup.
4. **Coordinate:** select the intended scene/tab and update navigation when its host is ready.
5. **Present:** the app builds `Feature.StartModuleView` using the appropriate interface-owned starter entry and injected configuration.

Keep purely local feature navigation internal. A package must not import another Feature implementation or reach into an app factory to open a destination.

## URL contract

- Define canonical paths and parameter semantics once per product; use them for link generation, parsing, web fallback, association rules, and tests.
- Parse structured URL components rather than substring matching. Validate identifiers and reject ambiguous required parameters, unsupported hosts, and malformed values.
- A URL is untrusted input, not proof of identity or access. Route through normal authorization and confirmation behavior; never encode credentials or silently perform destructive actions from a link.
- Keep optional attribution separate from the destination. Unknown marketing parameters may be ignored according to the product contract; unknown route paths must not silently become a successful route.
- An accepted request may target missing or inaccessible content. Define a useful unavailable state instead of displaying a different item.

Read [url-contract-and-association.md](references/url-contract-and-association.md) for canonical paths, URL validation, AASA, entitlements, and diagnostic boundaries.

## Readiness and lifecycle

- Handle both cold and warm launches. Receiving a request before navigation exists must not lose it.
- Store a pending typed request at the app/scene coordinator, with explicit ownership and replacement/queue policy. Persist it only if the product needs recovery across process termination.
- Revalidate prerequisites after login, onboarding, account changes, or asynchronous lookup. Preserve the requested destination without bypassing authorization.
- Make delivery idempotent for duplicate callbacks of the same request. Allow a later intentional tap of the same URL; do not blacklist URLs forever.
- Decide what happens when a second request arrives, the user cancels login, the target disappears, or an older lookup finishes after a newer request.
- Route notification taps separately from notification receipt. A background notification must not unexpectedly navigate unless the product explicitly defines that behavior.
- Choose a scene deliberately in multiwindow apps; do not mutate every scene or rely on a process-global path.

Read [routing-and-tests.md](references/routing-and-tests.md) when implementing pending requests, asynchronous resolution, or navigation coordination.

## SwiftUI integration

Use value-based `NavigationStack(path:)`. Keep one authoritative owner per path; in a tab host prefer local `@State NavigationPath` and deliberately apply coordinator requests to it. Avoid bidirectional mirrors that append the same destination twice.

Attach `.navigationDestination(for:)` to a stable non-lazy host. Reserve `NavigationLink` for passive navigation. If a tap also logs analytics or performs another side effect, invoke the explicit routing action from that source; never attach `.simultaneousGesture` to a `NavigationLink`.

Adapt lifecycle delivery to the app's actual lifecycle using the appropriate URL/user-activity callbacks. Do not register overlapping handlers without duplicate-delivery control. Keep app and feature SwiftUI views within the default 300-line limit by extracting cohesive hosts and components.

## Workflow and verification

1. Inspect or establish the URL contract, feature interfaces, root navigation, and lifecycle entry points.
2. Implement pure parsing and typed results, then prerequisite resolution and host coordination.
3. Wire real feature entries at the app root. Keep URL generation and associated-domain configuration aligned.
4. Test malformed requests, cold/warm delivery, authentication resume/cancel, duplicate callbacks, stale lookup completion, and correct tab/scene selection as relevant.
5. Build the affected app targets. Verify real Universal Link delivery separately from parser tests or direct URL injection.
6. Report what was verified: code routing, installed-app invocation, website association, and distributed-build behavior are different checks.

Framework behavior varies with lifecycle, OS, and SDK. Use the official references linked in the supporting documents when configuring or debugging platform integration. Do not infer that a passing parser proves website association.
