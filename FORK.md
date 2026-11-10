# Fork notes: OpenCode 2.2.6 on macOS arm64

This is a **local workaround**, not an upstream qualification. It is based on
upstream `v1.51.2` (GPL-3.0-or-later). It exists to show that the installed
receiver *can* accept an event on OpenCode 2.2.6 once the exact-image gates are
passed, and to give maintainers a concrete diff to audit.

## Status and honest limits

- **Delivery path works locally**: `opencode-event` returns
  `{"status":"submitted","desktop":"submitted"}` and `usernoted` presents a
  banner on macOS 26 arm64 with OpenCode 2.2.6 (Bun 1.4.4).
- **This is not native qualification.** The Darwin clock row reuses the
  `nativeReadBoundNS` / `comparisonBoundNS` / `translationBoundNS` values from
  the reviewed 2.0.21 darwin/arm64 row. No independent calibration of Bun 1.4.4
  FFI timing was performed.
- Bun 1.4.4 FFI source differs from 1.4.2 (including JSValue-to-slot
  conversion). Adding a Bun version/hash row is **not** a substitute for source
  and clock review. Upstream correctly refuses hash/semver-only admission.
- The profile/reader/qualification rows below are literal data additions; they
  do not change the wire protocol.

## Changes

Agent Notifications:

- `internal/opencodecodec/qualification_rows.go`
  - candidate image and clock row for
    `darwin/arm64`, entry `serve`, image SHA-256
    `90e4d03597b03d2051b65b4d793467764e55bacd8e22bf4824479c7233ea0e15`
    (OpenCode 2.2.6 official binary), candidate version `2.2.6`, generation `v2`
- `cmd/claude-notifications/opencode_trusted_rows.go`
  - matching runtime reader row (`official_native_serve_default_dual_autoload`,
    `v2_direct_asynciterable_data_sparse_error_end_checkpoint_after_prepare`)
- `internal/opencodeplugin/dist/agent-notifications.js`
  - `2.2.6` pins in the V2 setup/eligibility checks
  - `2.2.6` + image SHA in the darwin/arm64 clock-qualification cell
  - derive accepted versions from the cell's `images` instead of a hardcoded
    `"2.0.21"`
  - Bun `1.4.4` row with the OpenCode executable digest in the native clock image
    table
- `internal/notifier/delivery_process.go`
  - launch the helper with `open -g -j` (do not activate; launch hidden)

Dependency (vendored because the published module has no 2.2.6 row):

- `third_party/integrationctl` is a copy of
  `github.com/777genius/plugin-kit-ai/install/integrationctl`
  `v0.1.1-0.20261028163038-6ca8dc291616` with:
  - `opencodehost/observer.go`: one `NativeObserverDescriptor` for 2.2.6
  - `opencodehost/profile.go`: `2.2.6` added to the `tested_exact` switch
- `go.mod`: `replace github.com/777genius/plugin-kit-ai/install/integrationctl => ./third_party/integrationctl`

This vendoring is only to make the fork buildable. A real upstream change
belongs in `plugin-kit-ai` as a separate PR.

## Local-only, not in this repository

The installed helper on the test machine was re-signed ad-hoc with
`LSUIElement=true` to keep it out of the Dock. That is an installation detail,
not part of the source diff.

## Build and install (macOS arm64)

```bash
go build -trimpath -ldflags="-s -w" -o /tmp/claude-notifications-fork ./cmd/claude-notifications
"/tmp/claude-notifications-fork" setup-opencode install --binary /tmp/claude-notifications-fork --desktop
touch ~/.config/opencode/plugins/agent-notifications.js   # reload the observer
```

## Caveats

- The image pin is exact. Any OpenCode update changes the binary SHA-256 and
  disables delivery until a new row is added.
- Do not run the upstream installer to update: it replaces the runtime with a
  build that does not support 2.2.6.
- Windows is not qualified. It would need equivalent rows for `windows/amd64`
  and a Windows build.
