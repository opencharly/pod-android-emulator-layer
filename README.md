# android-emulator-layer

A booting Android 16 emulator as an OpenCharly candy.

This candy is a **composite glue layer**: it turns the underlying selkies
desktop + Android SDK + Appium stack into a running Android 16 emulator (API 36,
`google_apis_playstore` x86_64, `pixel_9a`). The Play Store image ships Play
Store, Google Play services, and Google Chrome preinstalled.

It ships two helper scripts (`start-emulator`, `wait-for-display`), two
supervisord services (`emulator` + `adb-server`), the AVD volume, and the
`/dev/kvm` device-passthrough declaration that round-trips through the OCI label
into the quadlet. CPU and RAM are auto-sized from the host at runtime, so the
same image right-sizes on a 4-core laptop and a 16-core box.

## What it provides

| Property | Value |
|---|---|
| Layer / candy | `android-emulator-layer` |
| Services / port | `emulator` (30), `adb-server` (35) on `5037` |
| Volume | `android-avd` → `/home/user/.android` |
| Device | `/dev/kvm` passthrough (`keep-groups`) |
| Requires | `layer-android-sdk`, `pod-appium-server`, `layer-supervisord`, `plugin-appium`, `plugin-adb` |

Optional, opt-in `secret_accept` entries (`GOOGLE_ACCOUNT_EMAIL`,
`GOOGLE_AAS_TOKEN`) feed the `charly check adb install-app --source google-play`
path; when absent, the check is green. The `plugin-appium` and `plugin-adb`
requires are the out-of-tree plugins that serve the `appium:` and `adb:` check
verbs and the `target: android` deploy.

## How to use it

Compose the candy into a box that provides a desktop (selkies) and the Android
toolchain:

```yaml
my-android:
  candy:
    base: cachyos.cachyos
    candy:
      - '@github.com/opencharly/pod-android-emulator-layer:<tag>'
```

Then build and deploy with the charly CLI:

```bash
charly box build my-android
charly start my-android
```

Drive the booted emulator with the `adb:` and `appium:` check verbs. See the
owning skills for the deploy substrate, the emulator boot flow, and the check
verbs.

## Layout

- `charly.yml` — the `android-emulator-layer:` candy entity: the five requires,
  env, `secret_accept`, the `/dev/kvm` security block, the two services, the
  volume, the port, and the plan (including the emulator and Appium checks).
- `start-emulator`, `wait-for-display` — the helper scripts installed by the plan.
- `tests/data/ApiDemos-debug.apk` — a test APK.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `CHANGELOG/` — per-CalVer history.
- `README.md` — this user overview.

## Related

- Owning skill: `/charly-check:android` — the `kind: android` schema, the
  in-pod emulator, the `target: android` deploy, and the `apk:` package format.
- Check verbs: `/charly-check:adb` and `/charly-check:appium`.
- [`opencharly/charly`](https://github.com/opencharly/charly) — the charly CLI and image builder
- [`opencharly/opencharly`](https://github.com/opencharly/opencharly) — the umbrella
