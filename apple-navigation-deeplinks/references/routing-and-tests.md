# Routing State and Behavioral Tests

## Request ownership

Represent the destination as an interface-owned value such as `ArticleStarterEntry.detail(id:)`. Wrap it at the app boundary only when delivery metadata is necessary: request identity, source, target scene, or processing generation. Do not put app routing state in a feature's interface.

A useful processing model is:

```text
received -> validated -> waiting for readiness/prerequisite -> resolving -> presented
                      -> rejected                         -> failed/cancelled
```

This is a reasoning model, not a required enum or framework. A simple app can implement it with a small pending value and explicit methods. A complex flow may justify a coordinator or state machine with tests.

## Pending navigation

- Keep a request until the owning host can accept it. Separate accepting/presenting a request from merely starting login or selecting a tab.
- Choose an intentional policy for multiple requests: latest wins for ordinary user navigation, a queue when each item must be processed. Do not add an unbounded queue by default.
- Deduplicate only the same delivery/in-flight request. If the platform gives no identity, scope coalescing to the current processing operation rather than persisting a URL denylist.
- When resolution is asynchronous, associate its result with the request generation; discard stale completions after cancellation or replacement.
- On sign-out or account switch, clear or revalidate account-dependent pending work. A previously resolved destination may no longer be authorized.
- Define whether navigation replaces the path, appends, or focuses an existing destination. Do not indiscriminately append on every callback.
- If persistence is needed, store versioned serializable route data, not view instances, concrete ViewModels, secrets, or an opaque navigation object that the product cannot migrate.

The host decides the selected tab/scene and builds the feature configuration. The feature's entry selects its internal screen. A route must not require Feature A to import Feature B.

## Test cases

| Boundary | Scenario | Expected behavior |
| --- | --- | --- |
| Parser | Unknown host, invalid ID, duplicate required parameter | Rejected without side effects |
| Parser/generator | Valid canonical link | Round trip preserves destination and permitted context |
| Coordinator | Cold launch before root appears | Request remains pending and is applied when ready |
| Coordinator | Login required | Login occurs first; destination resumes only after readiness/access checks |
| Coordinator | Login cancelled | Defined cancellation/fallback; no unexpected later navigation |
| Coordinator | Duplicate delivery during processing | One navigation effect |
| Coordinator | A later intentional tap of the same URL | New navigation can occur |
| Resolver | Request A finishes after newer request B | A cannot overwrite B |
| App assembly | Request for another tab | Correct tab and path updated once |
| App assembly | Notification received without user interaction | No navigation under the default policy |
| Destination | Missing or inaccessible entity | Appropriate unavailable state |

Use deterministic dependencies and controllable async completion. Test observable routes/effects rather than private properties. Keep parsing tests independent of SwiftUI and network. App-level tests cover composition; feature tests cover internal presentation behavior. UI tests follow the user's scope and existing project policy.

Instrument accepted/rejected/deferred/presented outcomes through injected monitoring where useful. Log sanitized destination categories and non-sensitive identifiers; do not emit full inbound URLs by default.
