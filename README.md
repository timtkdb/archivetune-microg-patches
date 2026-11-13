# ArchiveTune microG Patches

[Morphe](https://github.com/MorpheApp) patches for [ArchiveTune](https://github.com/rukamori/ArchiveTune) that add
sign-in through [microG](https://github.com/ReVanced/GmsCore) and make brand accounts selectable.

> Unofficial. Not affiliated with or endorsed by ArchiveTune, Morphe, ReVanced, microG or Google.
> These patches do not bypass authentication: you still sign in to your own Google account, only the
> way you do it changes.

## ❓ About

ArchiveTune signs in with a copied web cookie, which expires and offers no obvious way to use a brand account.
These patches let you pick a Google account from microG instead of typing a password, and make brand accounts
show up in ArchiveTune's account switcher.

**Sign in with microG**: on the Login screen, microG's account picker replaces Google's password page. microG
provides a one-time sign-in link for the chosen account (the mechanism Google's own apps use to open a signed-in
web page), the login page loads it, and ArchiveTune's normal login completes. If microG is missing, refuses, or
you cancel, the original Google page loads so login is never blocked.

**Show all brand accounts**: ArchiveTune hides every account YouTube reports as having no YouTube channel, which
includes most music-only brand accounts. This stops that filtering so they appear under
Settings > Account > the dropdown arrow next to the Account button > "YouTube channels".

### Requirements

- ArchiveTune **15.1.0**, official release APK (do not change the package name or version code: the app's
  built-in gatekeeper sends both to its server before allowing network access).
- microG-RE (package `app.revanced.android.gms`) with your Google account added.
- Morphe Manager (built against Patcher 2.3.0).

### Limitations

- The session is still a web cookie, so it can expire. Re-login is one tap in microG instead of a password.
- Tested on the FOSS universal APK. The GMS flavor was checked by inspecting its code but not run.
- Newer ArchiveTune versions are not listed as compatible yet. If a patch fails with
  "Failed to match the fingerprint", the app changed that code; please open an issue with the error.

### How to use these patches

Click the button below, or manually add this repo url to Morphe Manager: https://github.com/timtkdb/archivetune-microg-patches

<a href="https://morphe.software/add-source?github=timtkdb/archivetune-microg-patches">
  <img src=".github/assets/add-to-morphe.svg" alt="Add to Morphe" height="48">
</a>

## 🩹 Patches list

<!-- PATCHES_START EXPANDED -->

<!-- Do not modify this section by hand. The patch list is generated when release.yml creates a new release.

     If you wish for the patches list to be collapsed, then remove the word 'EXPANDED' from the comment tag above.

     If you wish to manually keep this list updated then remove the PATCHES_START and PATCHES_END
     comment blocks entirely. -->

#### A list of the patches will automatically be shown here after the first release is created.

&nbsp;

## 🚀 First-time setup (removed automatically by the first release)

1. [Set up](https://github.com/MorpheApp/morphe-documentation/blob/main/docs/morphe-development/README.md) a development environment, including a GitHub PAT as described
   [here](https://github.com/MorpheApp/morphe-patcher/blob/main/docs/2_1_setup.md#-prepare-the-environment)
   (`gpr.user` and `gpr.key` in `~/.gradle/gradle.properties`).
2. Create the GitHub repository `timtkdb/archivetune-microg-patches`, then push **both** a `main` and a `dev` branch
   (`.releaserc` releases from `main` and pre-releases from `dev`):
   ```
   git init -b main
   git add . && git commit -m "chore: initial commit"
   git remote add origin git@github.com:timtkdb/archivetune-microg-patches.git
   git push -u origin main
   git checkout -b dev && git push -u origin dev
   ```
3. Enable "Allow GitHub Actions to create and approve pull requests" in the repo Settings > Actions > General >
   Workflow permissions.
4. (Optional) Sign releases so Morphe Manager refuses a swapped release: run `./gradlew generateSigningKey`,
   commit `morphe-signing.pub`, never commit `morphe-signing.key` (already git-ignored), and add its content as the
   `MORPHE_SIGNING_KEY` Actions secret. Keep a backup of the key somewhere safe.
5. (Optional) Add `patches-bundle.png` for a custom icon in Morphe Manager instead of your GitHub avatar.

## 🧑‍💻 Dev usage

- **Make all changes on the `dev` branch.**
- Local build: `./gradlew buildAndroid`, the `.mpp` is written to `patches/build/libs/patches-*.mpp`.
- Use [semantic commit](https://kapeli.com/cheat_sheets/Semantic_Commits.docset/Contents/Resources/Documents/index)
  messages: `feat:` (new feature), `fix:` (bug fix), `bump:` (support for a new app version),
  `chore:` (no release). `feat:`/`fix:`/`bump:` on `dev` create a pre-release; merge `dev` into `main`
  (merge commit, no squash) for a stable release.
- **Always release through `release.yml`.** Do not create releases by hand and do not edit or commit
  `patches-list.json`, `patches-bundle.json` or `CHANGELOG.md`; the workflow generates them.
- Do not force-push semantic release commits.

<!-- The patches end tag is intentionally placed here so the first release will clean up
     this readme of all developer instructions above. -->
<!-- PATCHES_END -->

### 🔧 How it works

| Piece | Detail |
|---|---|
| Manifest | Adds microG-RE's `SPOOFED_PACKAGE_NAME/SIGNATURE` metadata so it presents the app to Google as YouTube Music, `<queries>` for microG, and the account picker activity. |
| Login hook | Redirects the login screen's `WebView.loadUrl(<Google sign-in URL>)` call to the extension. |
| Extension | Shows microG's picker, asks microG for a `weblogin:` token (microG returns a sign-in URL), loads it in the WebView, falls back to the original page on failure. The URL is a one-time credential and is never logged. |
| Brand accounts | Makes `YouTube.booleanValue(JsonObject, String)` return no value for the key `hasChannel`, so the app's existing `?: true` default applies. |

Fingerprints match by names and string literals only. ArchiveTune's release builds are optimized by R8 (lambdas merged
into switch classes, methods moved into unrelated classes, most of OkHttp stripped), so nothing depends on lambda
class names or on OkHttp.

### 🛠️ Building locally

- Run `./gradlew buildAndroid`
- The built patches .mpp file is found in `patches/build/libs/patches-*.mpp`
- Patch the mpp file using [Morphe-Desktop](https://github.com/MorpheApp/morphe-desktop)
  like any other patch bundle.

See the [Morphe documentation](https://github.com/MorpheApp/morphe-documentation) for more information.

## 📜 License

timtkdb's ArchiveTune microG Patches are licensed under the [GNU General Public License v3.0](LICENSE)
