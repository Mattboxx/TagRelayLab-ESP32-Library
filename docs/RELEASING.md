# Release and publication guide

## 1. Verify the source

Build every target from the standalone library repository:

```bash
pio run -d ci -e esp32
pio run -d ci -e esp32c3
pio run -d ci -e esp32c3_broadcaster
pio run -d ci -e esp32c3_minimal
pio run -d ci -e esp32c3_bluedroid
pio run -d ci -e esp32s3
pio run -d ci -e esp32c6
```

Check that examples contain no real keys, update `CHANGELOG.md`, and set the
same semantic version in `library.json` and `library.properties`.

Run `python test/check_metadata.py` and `python test/check_lifecycle.py` too.
The manifest dependency ranges must agree for Arduino and PlatformIO. CI pins
NimBLE-Arduino 2.5.0; the supported dependency range is `>=2.5.0 && <3.0.0`.

The current source snapshot intentionally remains **1.1.0** and includes
documented Unreleased fixes. This does **not** replace the original 1.1.0 release.
Never move an existing tag, replace an existing versioned archive, or republish
different bytes under an already released version. A future library release
requires a separately approved version bump; until then identify snapshots by
their full Git commit and leave the existing tags untouched.

## 2. Validate the PlatformIO package

From the repository root:

```bash
pio pkg pack . --output FindMyAdv-1.1.0.tar.gz
```

Inspect the archive before publishing. The PlatformIO CLI validates the
manifest while packing it.

## 3. Publish GitHub source

Use the public `Mattboxx/TagRelayLab-ESP32-Library` repository and merge verified
source through a protected pull request. For a newly approved version only,
create a new version tag and attach its generated archive. Do not
commit the personal `FindMyAdv-configured.zip` produced by TagRelayLab Server.

## 4. Verify installation from GitHub

From a clean PlatformIO project, reference the public repository:

```ini
lib_deps = https://github.com/Mattboxx/TagRelayLab-ESP32-Library.git
```

This unpinned URL follows the default branch, not a release tag. For a reproducible
snapshot use `https://github.com/Mattboxx/TagRelayLab-ESP32-Library.git#FULL_COMMIT_SHA`,
replacing the placeholder with the reviewed 40-character commit. Use `#vX.Y.Z`
only when that tag actually exists and you intend to test that historical release.
Build the clean project without a local library copy. A PlatformIO Registry
account is not required.

Official references:

- <https://docs.platformio.org/en/latest/librarymanager/creating.html>
- <https://docs.platformio.org/en/latest/core/userguide/pkg/cmd_pack.html>
