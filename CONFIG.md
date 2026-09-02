# Config

Adding another revanced app is as easy as this:
```toml
[Some-App]
apkmirror-dlurl = "https://www.apkmirror.com/apk/inc/app"
# or uptodown-dlurl = "https://app.en.uptodown.com/android"
```

## Instagram Variant:
- https://www.apkmirror.com/apk/instagram/instagram-instagram/instagram-439-0-0-37-89-release/instagram-439-0-0-37-89-21-android-apk-download/
## Twitter Variant:
- https://www.apkmirror.com/apk/x-corp/twitter/x-12-11-0-release-0-release/x-12-11-0-release-0-android-apk-download/

> [!WARNING]
> When a patch name itself contains a single quote, double it inside the string (e.g. 'Hide ''Get Music Premium''').

## More about other options:

There exists an example below with all defaults shown and all the keys explicitly set.  
**All keys are optional** (except download urls) and are assigned to their default values if not set explicitly.  

```toml
parallel-jobs = 1                    # amount of cores to use for parallel patching, if not set $(nproc) is used
compression-level = 9                # module zip compression level
remove-rv-integrations-checks = true # remove checks from the revanced integrations
dpi = "nodpi anydpi 120-640dpi"      # dpi packages to be searched in order. default: "nodpi anydpi"

patches-source = "revanced/revanced-patches" # where to fetch patches bundle from. default: "revanced/revanced-patches"
cli-source = "ReVanced/revanced-cli"             # where to fetch cli from. default: "ReVanced/revanced-cli"
# options like cli-source can also set per app
rv-brand = "ReVanced Extended" # rebrand from 'ReVanced' to something different. default: "ReVanced"

# channel appends "-<channel>" to the brand in release names and scopes the
# module's auto-update JSON path to update/<channel>/ so multiple flavors of
# the same module (e.g. stable and dev) can be installed side-by-side and each
# receive independent updates. default: "" (no suffix, no scope).
channel = "dev"

patches-version = "v2.160.0" # 'latest', 'dev', or a version number. default: "latest"
cli-version = "v5.0.0"       # 'latest', 'dev', or a version number. default: "latest"

[Some-App]
app-name = "SomeApp" # if set, release name becomes SomeApp instead of Some-App. default is same as table name, which is 'Some-App' here.
enabled = true       # whether to build the app. default: true
build-mode = "apk"   # 'both', 'apk' or 'module'. default: apk

# 'auto' option gets the latest possible version supported by all the included patches
# 'latest' gets the latest stable without checking patches support. 'beta' gets the latest beta/alpha
# 'auto-from-binaries' reads the version from the 'binaries' release body (see below).
# whitespace seperated list of patches to exclude. default: ""
version = "auto"     # 'auto', 'auto-from-binaries', 'experimental', 'latest' or a version number (e.g. '17.40.41'). default: auto

# optional args to be passed to cli. can be used to set patch options
# multiline strings in the config is supported
patcher-args = """\
  -OdarkThemeBackgroundColor=#FF0F0F0F \
  -Oanother-option=value \
  """

excluded-patches = """\
  'Some Patch' \
  'Some Other Patch' \
  """

included-patches = "'Some Patch'"                          # whitespace seperated list of non-default patches to include. default: ""
include-stock = "merged"                                   # 'merged', 'split' or 'disable'. default: merged
exclusive-patches = false                                  # exclude all patches by default. default: false

apkmirror-dlurl = "https://www.apkmirror.com/apk/inc/app"
uptodown-dlurl = "https://spotify.en.uptodown.com/android"
# direct download url. the url must point to an apk file. The literal token
# {version} is substituted with the resolved version for this table.
direct-dlurl = "https://website/com.google.android.youtube-{version}-all.apk"

module-prop-name = "some-app-module"                       # module prop name.
dpi = "360-480dpi"                                         # used to select apk variant from apkmirror. default: nodpi
arch = "arm64-v8a"                                         # 'arm64-v8a', 'arm-v7a', 'all', 'both'. 'both' downloads both arm64-v8a and arm-v7a. default: all
```

## Pinning versions via the `binaries` release

Create a GitHub release in this repo named exactly `binaries` (tag = `binaries`)
and put the stock APK files you want to build against as release assets. The
release **body** acts as a small key=value config:

```text
// Pin the piko pre-release. `Piko=` is stable-only (and the stable-channel
// fallback when no `stable.Piko=` is set). Dev auto-detects the latest
// pre-release from crimera/piko unless you pin it explicitly with `dev.Piko=`.
Piko=v1.0.0                // stable: v1.0.0; dev: auto-detect
stable.Piko=v1.5.0         // optional stable-only override
dev.Piko=v2.0.0            // dev-only pin (dev otherwise auto-detects)

// Stock app versions. Unprefixed applies to both channels; channel-prefixed
// wins when set. Pair with a `direct-dlurl` containing `{version}` so the
// build can resolve the right asset automatically.
Instagram=435.0.0.37.76
dev.Instagram=435.0.0.37.89  // dev needs a newer Instagram compatible with v2.0.0
Instagram-Clone=435.0.0.37.89

Twitter=12.11.0-release.0
```

Keys:

- `Piko=<tag>` pins the `crimera/piko` pre-release tag. **Stable-only by
  default** — the unprefixed `Piko=...` is honored on the stable channel and
  acts as a fallback when no `stable.Piko=...` is set. It is intentionally
  **not** applied to the dev channel, so dev builds keep auto-detecting the
  latest pre-release. To pin a dev build, set `dev.Piko=<tag>` explicitly.
  (For symmetry, the table-name pins below *do* honor the unprefixed form on
  both channels — only `Piko=` is channel-asymmetric.)
- `<TableName>=<version>` sets the version for the matching table when its
  `version` field is set to `auto-from-binaries`. Use `stable.<Table>` or
  `dev.<Table>` to override for a single channel.
- Unprefixed table keys apply to every channel that doesn't have a more
  specific override. So if you only have `Instagram=435.0.0.37.76`, both
  stable and dev use that version.
- Unknown channel prefixes (anything other than `stable.` or `dev.`) are
  ignored, so a typo can't accidentally break a build.
- Lines starting with `//` (or with `#` if the `#` is the first non-whitespace
  character) are comments. Inline `//` also comments out the rest of a line.
  Blank lines and lines without `=` are ignored. `//` is preferred because
  GitHub renders a leading `#` as a markdown heading in release bodies.

The CI ignores the `binaries` release when picking the next build's
`versionCode`, so it won't accidentally try to version-build it.
