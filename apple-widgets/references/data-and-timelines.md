# Widget Data and Timelines

## Data ownership

Choose data access according to the product: a shared cached snapshot, a focused extension-safe repository, or a combination. Do not automatically copy all account data into shared storage or require the full app to be running.

A shared snapshot should carry only the necessary serializable values: version, stable IDs, display-relevant content, update time, and account/configuration scope when needed. The source of truth remains the app/backend's owning service. Entry display models stay internal unless another target actually consumes their contract.

When sharing local data, configure the intended App Group and access its container or suite through injected configuration. Targets use separate processes; a singleton or Swift actor does not synchronize their memory. Use a consistent atomic/transactional storage protocol for cross-process reads and writes. [Apple: app groups](https://developer.apple.com/documentation/xcode/configuring-app-groups)

- Handle missing, corrupt, incompatible-version, and stale snapshots distinctly.
- Scope data to the selected widget configuration and current account. On logout/account change, clear or invalidate private snapshots and request the appropriate refresh.
- Re-read persisted state at the provider/action boundary rather than trusting a snapshot injected at a previous process launch.
- Persist accepted changes before requesting a timeline reload, so the extension cannot reload older data due to ordering.
- Avoid credentials in display snapshots. Apply privacy/redaction behavior appropriate to the surface and product.
- Store or generate images at suitable sizes; avoid decoding full-resolution media for a small widget.

These storage conventions do not require a new persistence framework. Use a focused existing abstraction when it can provide the necessary guarantees.

## Provider responsibilities

Keep data retrieval and transformation outside the entry view. The entry view renders supplied values without starting network requests in `body`.

| Provider output | Purpose |
| --- | --- |
| Placeholder | Lightweight generic structure, with no dependency on live account/network data |
| Preview snapshot | Representative deterministic content when requested for a preview context |
| Non-preview snapshot | Suitable current/cached data for the request context |
| Timeline | Dated entries and a reload policy based on the data's expected evolution |

Use the appropriate provider API for static or configurable widgets supported by the project. Keep configuration-specific fetches and caches distinct; two instances selecting different entities must not overwrite one global unscoped result.

## Reload policy

Build future entries for changes that are predictable from available data. Choose `.atEnd`, `.after(...)`, or `.never` according to when the provider should next be consulted, not as an exact execution schedule. External data changes may justify a targeted `WidgetCenter` reload request; avoid indiscriminate refresh calls from every view render. The system controls actual reload timing and budget. [Apple: keeping a widget up to date](https://developer.apple.com/documentation/widgetkit/keeping-a-widget-up-to-date)

For this convention, `.never` is appropriate only when the data can remain valid without scheduled provider reloads or another deliberate update mechanism exists. It does not prevent other system/app-triggered updates. Avoid tight retry loops on network failure: return useful cached/unavailable content and a considered retry policy.

Clock-driven display elements may use system-supported dynamic rendering where appropriate; they do not require treating the widget as a foreground app with a repeating timer.

## Tests

Inject the clock, repository, and storage when behavior depends on them. Cover empty cache, stale data, corrupt schema, failed refresh, multiple configurations, logout, and action-persistence/reload ordering where relevant. Assert the returned entries and policy rather than expecting the OS to reload at an exact instant.

A provider unit test proves data/policy decisions. An installed-widget observation checks system integration; neither proves every future scheduling decision.
