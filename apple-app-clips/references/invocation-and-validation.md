# App Clip Invocation and Validation

## Establish configuration

Use the project's Xcode or project-generation mechanism to create the App Clip target and its containing-app relationship. Keep generated signing and target relationships consistent with that mechanism. Make the Clip scheme, entry point, package products, resources, and intended deployment targets explicit.

Inventory the code actually linked by the Clip, including dependencies behind common packages. Avoid importing a large app assembly module just to obtain one service. Configure the backend/session/monitoring dependencies the Clip actually needs; unused SDK initialization can still affect startup and compatibility.

Select a size budget from current Apple requirements for the intended invocation and OS support, then measure the appropriate archived build. Do not equate source size, package count, or simulator output with distribution size. Check capability availability at runtime where the framework can compile but provide no usable functionality. [Apple: functionality and limits](https://developer.apple.com/documentation/appclip/choosing-the-right-functionality-for-your-app-clip)

## Invocation contract

- Receive the invocation user activity at the Clip lifecycle boundary and validate its URL.
- Resolve a typed starter request with stable IDs and necessary context. Keep tracking data separate from identity/authorization.
- Represent unavailable, expired, or malformed destinations explicitly.
- Handle a new invocation during an existing flow: define replacement, confirmation, or preservation behavior if unsaved work exists.
- Keep equivalent destinations available in the full app. Its installed-app routing must understand the intended invocation URLs as well.

For custom website invocations, align associated domains, the AASA `appclips` service, and the configured App Clip experience. Do not confuse this with the full app's `applinks` service or assume the Clip entry in AASA alone configures all full-app routes. Review default/demo links separately when those are the chosen invocation mechanism. [Apple: website association](https://developer.apple.com/documentation/appclip/associating-your-app-clip-with-your-website)

## Verification matrix

| Check | Evidence it provides |
| --- | --- |
| Pure URL parser tests | Contract validation and typed destination selection |
| Clip target build | Compilation and dependency compatibility at build time |
| Xcode scheme invocation using `_XCAppClipURL` | Delivery and handling of the configured invocation URL |
| Local experience on a device | App Clip card and device invocation behavior for that setup |
| Full-app Universal Link test with the invocation URL | Full app resolves the corresponding destination |
| Relevant archive inspection | Product packaging, signed capabilities, measured size |
| TestFlight/released experience, when in scope | Distribution-specific configuration and invocation |

Xcode's direct debug launch with `_XCAppClipURL` does not display the App Clip card. Test the card/launch experience separately; test full-app handling separately from the local Clip experience. Do not report these as interchangeable successes. [Apple: testing the launch experience](https://developer.apple.com/documentation/appclip/testing-the-launch-experience-of-your-app-clip)

Exercise unavailable network, failed data loading, repeated invocation, and termination/relaunch when those affect the task. Keep real website or App Store Connect mutations within the user's authorized scope; prepare configuration and local checks before reporting any external dependency.
