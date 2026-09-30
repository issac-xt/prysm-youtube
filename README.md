# prysm-youtube

Hosts the YouTube APK that [Prysm](https://github.com/issac-xt/prysm) patches. Prysm's setup wizard
downloads the single `.apk` asset from this repo's **latest release**, embeds itself into it with
LSPatch, and installs the result.

## Why the package name is different

YouTube ships as a system app on most devices, and it cannot be uninstalled - only disabled. LSPatch
re-signs every APK it patches, so a patched `com.google.android.youtube` can never be installed over
the Google-signed copy already present: the signatures conflict.

The APK here is therefore repackaged as **`prysm.xt.youtube`**, so it installs *alongside* the
original rather than replacing it. Prysm prompts you to disable the stock YouTube after setup.

## Why GmsCore is required

Changing the package name and the signing key both break Google sign-in, because Play Services keys
OAuth on exactly those two things. [ReVanced GmsCore](https://github.com/ReVanced/GmsCore) stands in
for Play Services: the APK here declares its original package name and signature as metadata, and
GmsCore presents those to Google on the app's behalf.

**Install GmsCore before installing this.** A build that starts without it decides GmsCore is absent
and will not pick it up later. Prysm handles this ordering for you.

## How it is built

`tools/apkprep` in the Prysm repo prepares each release from a stock YouTube APK. It rewrites the
package name, the provider authorities and the custom permissions, redirects the Google Play Services
identifiers to GmsCore's vendor, and adds the spoofing metadata - editing the binary manifest and the
dex string pools directly, leaving `resources.arsc` untouched.

## Conventions

- Exactly **one** `.apk` asset per release. Prysm takes the first `.apk` it finds, with no version or
  ABI selection, so a second asset would silently break the download.
- Release tags are whole numbers (`v1.0.0`, `v2.0.0`, ...) and are unrelated to YouTube's own version,
  which is recorded in the release notes.
