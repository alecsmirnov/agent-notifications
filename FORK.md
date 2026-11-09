# Fork notes: OpenCode 2.2.6 (macOS arm64)

This fork adds OpenCode **2.2.6** support to Agent Notifications on
**macOS arm64**. It is based on upstream `v1.51.2` (GPL-3.0-or-later).

## Why

Upstream pins native delivery to exact OpenCode images. OpenCode 2.2.6 is not
in the qualification tables, so the observer self-disables with
`runtime_unverified` and no notifications are sent.

## Changes

- `internal/opencodecodec/qualification_rows.go`: candidate image, clock row
  for `darwin/arm64` OpenCode 2.2.6 (`sha256:90e4d03597b03d2051b65b4d793467764e55bacd8e22bf4824479c7233ea0e15`)
- `cmd/claude-notifications/opencode_trusted_rows.go`: matching runtime reader row
- `third_party/integrationctl`: local copy of
  `github.com/777genius/plugin-kit-ai/install/integrationctl`
  (via `replace` in `go.mod`) with:
  - observer descriptor for 2.2.6
  - `2.2.6` in `profile.go` qualification switch
- `internal/opencodeplugin/dist/agent-notifications.js`: 2.2.6 pins/checks and
  **Bun 1.4.4** clock image row (OpenCode 2.2.6 ships Bun 1.4.4)
- `internal/notifier/delivery_process.go`: launch helper with `open -g -j`
- Diagnostics write to `/tmp/an-diag.log` (safe to remove)

The installed helper used by this machine additionally has `LSUIElement=true`
and is ad-hoc signed; that is a local installation detail, not part of this
repository.

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
- Windows is not qualified here. It needs equivalent rows for `windows/amd64`
  (image SHA-256, Bun image, clock/reader descriptors) and a Windows build.
