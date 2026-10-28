# Archway runtime

The Wine runtime that [Archway](https://github.com/playarchway/archway) downloads on first start, and how it is built. One package holds:

- Wine from [archway-wine](https://github.com/playarchway/archway-wine), WineHQ's Wine with Archway's macOS patches, at the commit in [`wine.ref`](wine.ref)
- FreeType 2.15.0
- Wine Mono 11.3.0, shared, so new bottles skip the Mono download dialog
- Apple's D3DMetal from the Game Porting Toolkit 4.0 beta 3, unmodified and staged into `Wine/lib`

Each [release](https://github.com/playarchway/archway-runtime/releases) carries `archway-wine-VERSION.tar.xz` and its SHA-256. `VERSION` is Wine's version and the short commit, for example `11.20-e51c772`. The archive holds `Wine/` and `WhiskyWineVersion.plist`. Archway unpacks it into `DATA/runtime-overlays/archway-wine-VERSION/`; each app release names the runtime it uses in `Sources/ArchwayCore/Resources/runtime.json`.

## A new runtime

1. Change `wine.ref` to the full commit ID in archway-wine and push it to `main`. The Runtime workflow (`.github/workflows/runtime.yml`, on a macOS runner, about 20 to 45 minutes) builds the package and publishes the release. It stops early if a release for that commit already exists.
2. The workflow then opens a pull request in Archway that points `runtime.json` at the new release, with its URL, SHA-256, and size (`.github/workflows/propose.yml`; it also runs by hand for an existing release). Test Steam and a game with the runtime, then merge it; the next app release ships it.

The workflow needs Archway Bot (organization variable `ARCHWAY_BOT_APP_ID`, secret `ARCHWAY_BOT_PRIVATE_KEY`). It reads Apple's Game Porting Toolkit from the private repository `playarchway/archway-build-inputs` and uses the `archway` command line tool from `playarchway/archway` to stage D3DMetal the way the app expects.

The same steps run locally, with Archway and archway-wine checked out next to this repository:

```sh
scripts/fetch-wine-tools.sh TOOLS          # pinned llvm-mingw, Wine Mono, x86_64 FreeType; needs `brew install bison`
WINE_TOOLS=TOOLS WINE_BUILD=BUILD scripts/build-wine.sh INSTALL
WINE_TOOLS=TOOLS scripts/package-runtime.sh INSTALL/usr/local Evaluation_environment_for_Windows_games_4.0_beta_3.dmg DIST VERSION
```

`WINE_SOURCE` and `ARCHWAY_SOURCE` point at other checkouts; the defaults are `~/dev/archway-wine` and `~/dev/archway`.

## Licenses

The scripts are under the GPL-3.0, like Archway; see [LICENSE](LICENSE). The packages contain software under its own licenses:

- **Wine** is LGPL. The archive names the exact archway-wine commit it was built from, and Wine's license files are in `Wine/share/doc/archway-wine`.
- **FreeType**'s license is in the same folder.
- **D3DMetal** is Apple's, redistributed unmodified and non-commercially under the Game Porting Toolkit license; that license ships in `Wine/lib/external/GPTK-License.rtf`.
