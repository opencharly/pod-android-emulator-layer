# AGENTS.md — pod-android-emulator-layer

Standalone candy repo for the `android-emulator-layer` candy — a composite glue
layer that boots an Android 16 emulator (API 36, `google_apis_playstore`
x86_64, `pixel_9a`) on top of selkies + Android SDK + Appium. The candy lives in
`charly.yml` at the repo root, with two helper scripts and a test APK beside it;
the emulator system image and the Appium/ADB tooling come from the required
layers and the out-of-tree `plugin-appium` / `plugin-adb` plugins.

Canonical files:

- `charly.yml` — the `android-emulator-layer:` candy entity (description,
  `require`, `env`, `secret_accept`, `security`, `service`, `volume`, `port`,
  `plan`).
- `start-emulator`, `wait-for-display` — the helper scripts the plan installs.
- `tests/data/ApiDemos-debug.apk` — a test APK for the `adb:` / `appium:` verbs.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `CHANGELOG/` — per-CalVer history.
- `README.md` — user overview only; never agent guidance.

## Load these skills first (R0)

- `/charly-check:android` — the owning skill: the `kind: android` schema, the
  in-pod emulator, the `target: android` deploy, the `apk:` package format, and
  the android deploy preresolver. Load before editing, building, deploying, or
  troubleshooting this candy.
- `/charly-check:adb` — the `adb:` check verb (device shell, install, getprop,
  screencap, logcat) served by `plugin-adb`.
- `/charly-check:appium` — the `appium:` check verb (WebDriver sessions,
  element find/click, mobile caps) served by `plugin-appium`.
- `/charly-image:layer` — the candy authoring reference (`charly.yml` schema,
  `plan:` step verbs incl. `check:`, package sections, service declarations).
  Load before editing any entity field or plan step.
- `/charly-internals:git-workflow` — before any git/PR action.

The candy carries no `skill:` entity of its own; the family skill
`/charly-check:android` covers the surface. The gap is routed to the named skill-authoring
batch [opencharly/opencharly#291](https://github.com/opencharly/opencharly/issues/291);
when that lands, add the owning skill here.

## Build / validate / test

- `charly box validate` at the repo root — the structural check: the manifest
  must parse and validate at the installed charly.
- The merge gate is the **org-wide** `charly/pr-validator` (required check
  `validate / validate`, defined in `opencharly/.github`); this repo has **no**
  per-repo candy gate. Its only workflow file is
  `.github/workflows/tag-on-merge.yml`.
- The live R10 witness is the candy's own `plan:` runtime checks (the booted
  emulator, the `adb:` device probe, the `appium:` session) plus the consuming
  box's check bed. These require a host with `/dev/kvm` and the rootless podman
  socket.

## Modify this repo

- Edit the `android-emulator-layer:` candy entity in `charly.yml`; the helper
  scripts and the inline checks are part of that entity's `plan:`.
- The `/dev/kvm` passthrough and the `keep-groups` annotation are load-bearing
  for the rootless emulator — preserve them.
- The `plugin-appium` and `plugin-adb` pins and the `appium:` / `adb:` checks
  move together; a verb-surface change needs the matching plugin pin.
- Keep the emulator API level / device / system-image vars and the chromedriver
  major in step with the `pod-appium-server` candy.

## Landing

- PR-only. Every change lands through a pull request; the org-required
  `charly/pr-validator` validates the diff and body and arms native auto-merge
  on PASS. Direct pushes to `main` are blocked.
- History lives in `CHANGELOG/` (written by `tag-on-merge` at merge time); the
  PR body IS the changelog.
- The authoritative rulebook is the umbrella `AGENTS.md` in
  `opencharly/opencharly` and `charly/AGENTS.md` in the charly repo. Do not
  restate its rules here.
