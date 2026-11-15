> [!IMPORTANT]
> **This fork is the Android edition of [OpenIDEA](https://github.com/l0ng-ai/openidea).** Every release here is an upstream release with the Android build on top. Download the APK from [Releases](https://github.com/ptrkhh/openidea/releases) and install it over the previous one. Below this section, the README is upstream's.

## Android edition: making a release

### Branches

| Branch | What it holds |
| --- | --- |
| `feat/android-termux` | One commit: the Android build (also [PR #1253](https://github.com/l0ng-ai/openidea/pull/1253) upstream). It always sits on an upstream release tag. |
| `android-apk` (default) | That commit plus one more: this README section and the release workflow (`.github/workflows/android.yml`). Releases are tagged here. |
| `main` | Not used. |

### One-time setup

```sh
git clone https://github.com/ptrkhh/openidea.git && cd openidea
git remote add upstream https://github.com/l0ng-ai/openidea.git
git switch feat/android-termux   # creates the local branch from origin
git switch android-apk
```

The workflow signs the APK with two repository secrets: `ANDROID_KEYSTORE` (the `.jks` file, base64) and `ANDROID_KEYSTORE_PASSWORD`. Keep that keystore safe: Android installs an update only if it is signed with the same key as the installed app.

### When upstream publishes a release

Example: upstream publishes `v0.1.51`. Replace `0.1.51` with the real version everywhere below.

1. **Fetch** upstream's release tag and this fork's branches:
   ```sh
   git fetch upstream --tags
   git fetch origin
   ```
2. **Move the Android commit** onto the release:
   ```sh
   git switch feat/android-termux
   git reset --hard origin/feat/android-termux
   git rebase v0.1.51
   ```
   If git stops on conflicts: fix the files it lists, `git add` them, then `git rebase --continue` (or `git rebase --abort` to start over).
3. **Move the release commit** on top of it:
   ```sh
   git switch android-apk
   git reset --hard origin/android-apk
   git rebase --onto feat/android-termux android-apk~1
   ```
4. **Push** both branches:
   ```sh
   git push --force-with-lease origin feat/android-termux android-apk
   ```
5. **Tag** the release. Pushing the tag starts the build:
   ```sh
   git tag v0.1.51-android android-apk
   git push origin v0.1.51-android
   ```
6. **Wait** for Actions → "Android release" to finish (about 25 minutes). The APK then appears under [Releases](https://github.com/ptrkhh/openidea/releases) as `OpenIDEA_0.1.51-android_arm64.apk`.

### If the build fails

Usually upstream added window or desktop code that Android doesn't have; the log names the file and line. Fix it on `feat/android-termux`, then:

```sh
git switch feat/android-termux && git commit -a --amend --no-edit
git switch android-apk && git rebase --onto feat/android-termux android-apk~1
git push --force-with-lease origin feat/android-termux android-apk
git tag -f v0.1.51-android android-apk && git push -f origin v0.1.51-android
```

If a release already exists for that tag, delete it first (Releases → the release → Delete). If pushing the tag doesn't start a build, start it by hand: Actions → "Android release" → Run workflow → pick the tag.

---

<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="src-tauri/icons/source/openidea-wordmark-dark.svg">
    <img src="src-tauri/icons/source/openidea-wordmark.svg" alt="OpenIDEA" height="56">
  </picture>
</p>

<p align="center">
  The IDE you know, rebuilt in Rust.
</p>

<p align="center">
  English · <a href="README.zh-CN.md">简体中文</a> · <a href="README.tr.md">Türkçe</a>
</p>

<p align="center">
  <a href="https://github.com/l0ng-ai/openidea/releases/latest"><b>Download</b></a>
  &nbsp;·&nbsp;
  <a href="https://openidea.sh">Docs</a>
  &nbsp;·&nbsp;
  <a href="#benchmark">Benchmark</a>
  &nbsp;·&nbsp;
  <a href="#plugins">Plugins</a>
  &nbsp;·&nbsp;
  <a href="#community">Community</a>
</p>

https://github.com/user-attachments/assets/234dd54c-1255-4b3a-b35c-533d78d737c0

OpenIDEA brings the IntelliJ IDEA way of working — its keymap, its tool windows,
its search-everything habits — to a native app with a Rust core.

Editor, code insight, debugger, Git, terminal, database and HTTP clients,
for 60+ languages. Builds for macOS, Windows and Linux.

## Benchmark

Opening [freqtrade](https://github.com/freqtrade/freqtrade) (Python, ~770 files)
with warm caches, median of 5 runs:

|                             | OpenIDEA          | IntelliJ IDEA 2026.2.3 |
| --------------------------- | ----------------- | ---------------------- |
| Window with the project     | **0.9 s**         | 3.1 s                  |
| Ready (CPU settled)         | **1.1 s**         | 13.2 s                 |
| Memory (USS, all processes) | **169 MB**        | 2011 MB                |
| CPU time until ready        | **2.4 s**         | 47.9 s                 |
| Idle CPU                    | **4 %** of a core | 7 % of a core          |
| Quit                        | **0.3 s**         | 5.9 s                  |

A release build of OpenIDEA from `main` and IDEA with default settings and plugins, on
Windows 11, 32-core AMD, 62 GB RAM.
IDEA analyses Python more deeply, so "ready" doesn't mean the same work on
both sides. Both are measured from the outside, the same way: see
[`perf/compare`](perf/compare) to run it on your machine and projects.

## Plugins

Plugins add actions, completion, quick fixes, tool windows, custom editors,
themes, file icons and languages. They run as sandboxed WebAssembly components;
themes, icon themes and languages need no code at all.

Install them from Settings › Plugins, or start your own with
`openidea-plugin new` — see [Writing OpenIDEA plugins](https://openidea.sh/en/plugins) and
the [official plugins](plugins/official). The plugin API is still a preview.

## Community

- [Discord](https://discord.gg/2Zy4yn7gP) — questions, feedback and release news.
- WeChat group — scan to join:

  <img src=".github/assets/wechat-group-3.png" alt="WeChat group QR code" width="200">

- Contributing: see [CONTRIBUTING.md](CONTRIBUTING.md) and the [contributor docs](https://openidea.sh/en/contribute/setup).

## License

[Apache 2.0](LICENSE). Bundled third-party assets are listed in [`THIRD_PARTY_LICENSES.md`](THIRD_PARTY_LICENSES.md).

<sub>Not affiliated with JetBrains. IntelliJ IDEA is a trademark of JetBrains s.r.o.</sub>
