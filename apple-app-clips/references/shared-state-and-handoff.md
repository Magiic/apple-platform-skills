# Shared State and Reliable Handoff

## Define the contract before the store

Only introduce handoff persistence when the user task needs continuity. A fictional draft/action record may contain:

- Schema version and stable operation ID.
- Creation/update time and relevant expiry policy.
- Stable destination/entity ID and the action payload.
- Account/anonymous ownership context needed to verify who may resume it.
- Relative media references within the approved shared container.
- Minimal non-sensitive invocation context needed after installation.

Place values consumed by both targets in a small owning interface or shared contract module. Keep file I/O, migration, synchronization, and provider calls in implementation code. Do not maintain two independent copies of the serialized schema in the app and Clip.

## Storage boundary

For appropriate non-sensitive state, configure the same App Group on the corresponding app and Clip and resolve its container/preferences through that configuration. Keep credentials out of shared defaults and ordinary draft files. Review the supported keychain transition and OS behavior instead of assuming bidirectional shared login storage. [Apple: data sharing](https://developer.apple.com/documentation/appclip/sharing-data-between-your-app-clip-and-your-full-app)

Reliability conventions:

- Inject storage and configuration so tests use isolated temporary locations.
- Handle a missing shared container explicitly; do not silently write somewhere the full app cannot read.
- Use atomic replacement or an appropriate transaction for records. A Swift actor serializes work inside one process; it does not provide cross-process locking. Choose file/database coordination if concurrent writers are possible.
- Keep migrations versioned. Distinguish no saved data from corrupt or unsupported data; do not treat a decoding error as an empty success and erase recoverable work.
- Copy necessary media into owned storage before saving the referencing record. Validate relative paths stay within the container and do not rely on temporary URLs or a photo-library identifier being portable.
- Define cleanup for obsolete records and orphaned files. Preserve files still referenced by pending work.
- Do not assume the system retains Clip state indefinitely; define sensible expiry and unavailable-data behavior.

## Import workflow in the full app

```text
load supported records
  -> wait for required session/readiness
  -> verify ownership and current permissions
  -> submit using stable operation identity
  -> confirm success or already-applied result
  -> acknowledge the exact record version
  -> clean up data no longer referenced
```

These steps belong to a focused synchronizer/use-case when behavior warrants one, not to a giant root SwiftUI view. The app root triggers coordination; repositories and stores are injected.

- Persist an idempotency key before submission and reuse it for retries. Do not generate a new key each launch.
- True protection against a retry after server success/client interruption requires a server-side idempotency contract or equivalent deduplication. Local flags alone cannot prove exactly-once delivery.
- Keep failed or uncertain operations pending. Remove only acknowledged records, not the whole batch after partial success.
- Before removal, verify the stored record is still the version submitted. An updated draft for the same entity must not be deleted by an older completion.
- Prevent concurrent duplicate sync within the process and resolve competing writers through the storage protocol if needed.
- Re-check the current session/permissions when resuming. Never silently import one person's anonymous/private data into a different account without the product's intended ownership flow.
- Report recoverable failures through injected monitoring with sanitized context. Preserve recoverable work when media or network is temporarily unavailable.

## Behavioral tests

Cover save/reload, malformed/older schema, media missing, repeated import, partial batch failure, cancellation, and account changes as relevant. Simulate server success followed by a lost response: the next attempt must reuse the operation identity. Simulate a draft edit while an import is pending: acknowledging the old record must preserve the new one.

Use deterministic IDs/dates, controllable repository outcomes, and temporary storage. Tests must not require a live App Group, backend, or production keychain. A separate signed-device check verifies actual container access and the installation transition when that is part of the task.
