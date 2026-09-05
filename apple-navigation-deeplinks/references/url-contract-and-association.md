# URL Contract and Website Association

## Define a product contract

For a new app, agree on its actual domain and supported destinations. Until they exist, use clearly illustrative values in examples and keep configuration injectable; never invent production identifiers.

A fictional contract might contain:

| Path | Required value | Result |
| --- | --- | --- |
| `/articles/{id}` | Valid article identifier | Article detail request |
| `/collections/{id}` | Valid collection identifier | Collection detail request |
| `/auth/callback` | Parameters required by the chosen auth protocol | Dedicated authentication handler |

Authentication callbacks are not ordinary content links: preserve provider verification, state/nonce handling, and error semantics. Never log raw callbacks containing tokens or codes. Canonical paths belong to the product, not to this skill.

Maintain one owner for URL generation/parsing rules. Share a small contract/parser across app targets when needed; do not make an interface depend on a concrete feature just to share a parser. Plain parsing behavior can live in a focused utility, while shared request values remain in the owning interface.

## Parse explicitly

- Use `URLComponents` and check the permitted scheme and host. For a single allowed domain, exact comparison is safer than `contains` or a naive suffix check.
- Decide whether subdomains, explicit ports, fragments, trailing slashes, path case, and custom schemes are supported. Reject user-info components for ordinary content links.
- Match complete path segments and validate IDs. Do not match `/articles-extra/` as `/articles/`.
- Define percent-encoding behavior; do not repeatedly decode paths or let an encoded separator introduce an unintended route segment.
- For required query parameters, reject duplicates unless the contract expressly defines repeated values. Avoid constructing a dictionary that traps or silently selects one duplicate.
- Preserve optional attribution as separate data. Bound large inputs where they could cause excessive work.
- Return a typed unsupported/malformed result with no effects; avoid opening arbitrary fallback URLs taken from an unvalidated query parameter.

These are input-contract design conventions. Adapt accepted syntax to a real product requirement rather than broadening the parser to accept every URL shape.

## Universal Links configuration

An app-side associated-domain entitlement and a matching website AASA file establish the association. Use `applinks:<domain>` for Universal Links. In AASA, `applinks` describes associated apps and matching URL components. App Clip association uses the distinct `appclips` service. Use the actual application identifier prefix and bundle identifiers; do not assume a copied team identifier is correct. [Apple: associated domains](https://developer.apple.com/documentation/xcode/supporting-associated-domains)

Review root domains, subdomains, and wildcard behavior against Apple's current rules. Serve the AASA JSON at the documented HTTPS location without treating a normal web redirect or HTML page as an association file. Keep web paths, app parsing, and association patterns consistent; exclude paths meant to stay on the web. [Apple: debugging Universal Links](https://developer.apple.com/documentation/technotes/tn3155-debugging-universal-links)

Keep entitlement values and web deployment changes explicit and reviewable. A task to implement navigation does not automatically require publishing unrelated website configuration.

## Verify distinct boundaries

1. **Contract tests:** generated URLs parse to the intended requests; malformed and unsupported values are rejected.
2. **App routing:** a delivered URL reaches the expected destination, including cold start and required login.
3. **Association:** inspect AASA and the signed app's entitlements, then test installed-app link activation on a device.
4. **Distribution:** verify the intended build and hosting configuration when that environment is part of the task.

Account for association caching and context-dependent browser behavior. A link typed into a browser, tapped within the same website, or delivered directly to the app does not prove the same thing. Use Apple's diagnostic procedure when a valid route still opens on the web. [Apple: link behavior](https://developer.apple.com/documentation/xcode/allowing-apps-and-websites-to-link-to-your-content)
