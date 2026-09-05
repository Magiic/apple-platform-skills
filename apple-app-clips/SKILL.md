---
name: apple-app-clips
description: >
  Create, integrate, and test App Clips in modular iOS apps. Use for a focused Clip target,
  invocation URLs, capability and dependency selection, shared data handoff to the full app,
  pending-action migration, and launch-experience verification.
---

# Apple App Clips

## Intended result

Deliver a focused App Clip experience that reuses the app's feature contracts and suitable implementation modules, with its own composition root and a deliberate transition to the full app. These conventions require no reference project.

An App Clip is a separate app target with its own lifecycle and constraints; do not treat it as an ordinary full-app screen or assume generic app-extension rules all apply. Review current platform requirements before choosing capabilities. Size limits depend on supported OS versions and invocation modes, so do not hard-code a universal budget in generated architecture. [Apple: choosing functionality](https://developer.apple.com/documentation/appclip/choosing-the-right-functionality-for-your-app-clip)

## Scope and module ownership

- Identify one short user task or a small coherent set of invocation experiences. Avoid copying full onboarding, settings, or account requirements into the Clip unless necessary for that task.
- Reuse `FooInterface` and compatible `FooFeature` packages. Keep `StartModuleView` as the feature's unique public screen entry and use typed startup context for alternate supported modes.
- Give the Clip its own dependency factories, root host, lifecycle handlers, and configuration. Never import the full app target or call its `AppDependency` factories.
- Only link necessary feature/client/utility products. Audit transitive SDKs, resources, capabilities, and runtime availability; a successful compile alone does not prove an API works in an App Clip.
- If a reusable feature drags in incompatible functionality, extract the focused capability or inject a suitable implementation. Do not duplicate the whole feature or sprinkle runtime Clip checks throughout its internals.
- If a capability is intentionally unavailable, represent that in configuration/UI and the contract's behavior. Do not fake success using a no-op repository for a required operation.
- Keep ViewModels optional and views within the default 300-line limit. The Clip host composes; reusable business behavior remains in its owning implementation module.

## Invocation

Parse the invocation into a typed, validated request and pass it to the selected feature entry. Support repeat invocations while running and an appropriate invalid/unavailable state. The full app must also handle the supported invocation destinations when installed. Share parsing where useful without coupling the two app targets. [Apple: responding to invocations](https://developer.apple.com/documentation/appclip/responding-to-invocations)

Read [invocation-and-validation.md](references/invocation-and-validation.md) for target setup, website association, test environments, and build checks. Treat production domains, bundle IDs, app groups, and signing identifiers as project inputs, never invented constants.

## Continuity with the full app

Choose a supported storage mechanism for data that must survive the transition. Use a shared container or shared preferences for suitable non-sensitive values; handle sensitive credentials through the supported keychain/authentication mechanisms. Do not assume keychain sharing is symmetric across both targets and all OS versions. [Apple: data sharing](https://developer.apple.com/documentation/appclip/sharing-data-between-your-app-clip-and-your-full-app)

Read [shared-state-and-handoff.md](references/shared-state-and-handoff.md) when persisting drafts, moving media, resuming authenticated actions, or importing pending work. That document defines reusable reliability conventions, not a mandatory storage framework.

## Workflow

1. Establish the user task, supported invocation modes, deployment targets, and existing feature boundary. For a new app, create the app/Clip targets and local package pair with separate composition roots.
2. Review current Apple capability/size constraints for those choices and keep the dependency closure small.
3. Implement typed invocation handling, focused UI, and explicit dependency configuration.
4. If continuity is needed, define a versioned shared payload and an idempotent handoff before implementing persistence.
5. Verify parsing, pending-action behavior, target builds, archive size when relevant, and actual invocation separately.
6. Report which checks passed and which require the device, signing, hosting, or distribution environment. Do not claim a local launch proves a released experience.

## Done means

- The Clip can perform its intended task with its configured capabilities.
- Both app targets reuse contracts without depending on each other's source target.
- Valid invocations select the right feature mode; invalid/repeated invocations have defined behavior.
- Needed state can be resumed without silently dropping data or duplicating a confirmed operation.
- Tests exercise the task's failure/recovery cases, and the relevant target builds were checked.
