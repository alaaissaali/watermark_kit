# Upstream contribution and fork publish

This repo ships **`flutter_watermark_kit`** on pub.dev — a maintained fork of [tttocklll/watermark_kit](https://github.com/tttocklll/watermark_kit). Version **2.2.1** gives users two iOS improvements upstream 2.2.0 did not have:

| Change | User impact |
|--------|-------------|
| **iOS video compose fix** | Progress/completion/error callbacks run on the main queue; export no longer hangs when sample buffers end (especially with audio passthrough). |
| **iOS Swift Package Manager (SPM)** | Dual **SPM + CocoaPods** layout for Flutter **3.41+** (SPM path default on **3.44+**). CocoaPods remains supported. |

Fork-only packaging (renamed iOS module paths, `ios/flutter_watermark_kit/`, `Package.swift`, podspec name) lives on branch **`flutter_watermark_kit`**. Upstream contributions are split so each PR stays small and reviewable.

---

## Push this fork (`flutter_watermark_kit`)

Remote `origin` should be:

```text
https://github.com/alaaissaali/flutter_watermark_kit.git
```

Create an empty GitHub repo named `flutter_watermark_kit` if it does not exist, then from the repo root:

```bash
git checkout flutter_watermark_kit
git push -u origin flutter_watermark_kit:main
git tag v2.2.1
git push origin v2.2.1
```

## Publish on pub.dev

```bash
flutter test
dart pub publish --dry-run
dart pub publish   # interactive login required
```

Publish as **`flutter_watermark_kit`** (not `watermark_kit`). Keep `LICENSE` and upstream copyright; describe the fork in `README` / `CHANGELOG`.

---

## Upstream PR 1 — iOS video fix

Branch **`fix/ios-video-platform-thread-and-eos`** contains only the video fix under the **original** `watermark_kit` layout (no SPM / no rename).

1. Fork [tttocklll/watermark_kit](https://github.com/tttocklll/watermark_kit) on GitHub.
2. Add remote and push:

```bash
git checkout fix/ios-video-platform-thread-and-eos
git remote add watermark-fork https://github.com/alaaissaali/watermark_kit.git
git push -u watermark-fork fix/ios-video-platform-thread-and-eos
```

3. Open a PR into `tttocklll/watermark_kit:main`.

**Title:** `fix(ios): video callbacks on main thread and correct end-of-stream`

**Body:**

```markdown
## Problem
- Video compose progress/completion/error Pigeon callbacks were not always delivered on the main queue (Flutter platform thread).
- End-of-stream handling could hang when `copyNextSampleBuffer()` returned nil while `reader.status` remained `.reading` (notably with audio passthrough).

## Solution
- Dispatch video callbacks on the main queue.
- Treat nil sample buffers as EOS for video/audio tracks.
- Throttle progress (~250ms) and bound audio drain wait.

Touches `ios/Classes/VideoWatermarkProcessor.swift`. Bump version to **2.2.1** (or next patch upstream chooses).
```

---

## Upstream PR 2 — iOS SPM support (planned)

The **SPM + CocoaPods** work is on **`flutter_watermark_kit`** (see `ios/flutter_watermark_kit/Package.swift`, podspec, and `FlutterFramework` local package stub). Upstream still uses `ios/Classes/` + CocoaPods only.

When opening an SPM PR to upstream:

- Adapt paths to upstream’s **`watermark_kit`** naming (`ios/watermark_kit/` or whatever matches [Flutter’s SPM plugin layout](https://docs.flutter.dev/packages-and-plugins/developing-packages#spm)).
- Keep **both** integration paths unless upstream requests SPM-only.
- Document **Flutter >= 3.41** for the SPM integration path in `README` / `CHANGELOG`.
- Prefer a dedicated branch (e.g. `feat/ios-spm-dual-integration`) **after** or **in parallel with** the video fix PR, so reviewers can merge fixes independently.

Until that PR merges, consumers who need SPM on iOS should depend on **`flutter_watermark_kit: ^2.2.1`**.

---

## When apps can use upstream `watermark_kit` again

| App needs | Use |
|-----------|-----|
| iOS video fix + **SPM** | `flutter_watermark_kit: ^2.2.1` until upstream ships SPM |
| iOS video fix + **CocoaPods only** | `watermark_kit: ^2.2.1` after upstream merges the **video fix** PR |
| Both fixes in official package | `watermark_kit: ^2.2.1` (or later) after **both** upstream PRs merge — then drop the fork dependency and switch imports to `package:watermark_kit/...` |