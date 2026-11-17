# Knobdeck/.github

Shared GitHub configuration for the Knobdeck organization.

## Reusable workflows

### `build-plugin.yml`

Builds KnobDeck plugin folders on one OS with `knobdeck-plugin build` + `package`, optionally runs
`knobdeck-plugin conform`, stages the bundles under their release names and uploads them as the
artifact `bundles-<os>`. Used by `knobdeck-plugins` (the first-party monorepo), plugins split out of
it, and third-party plugins that want to be listed in the KnobDeck catalog.

```yaml
jobs:
  build:
    strategy:
      matrix:
        os: [macos, windows, linux]
    uses: Knobdeck/.github/.github/workflows/build-plugin.yml@main
    with:
      os: ${{ matrix.os }}
      plugins: "."          # or space-separated plugin folders in a monorepo
      attest: ${{ github.event_name != 'pull_request' }}
    permissions:
      contents: read
      id-token: write       # only needed with attest: true
      attestations: write   # only needed with attest: true
```

Output (artifact `bundles-<os>`):

| File | What |
|---|---|
| `<dir>_<os>_<version>.knobdeck` | The bundle |
| `<dir>_<os>_readme.md` | Its readme, when it has one |
| `part-<os>.json` | One entry per bundle: `dir, id, version, sha256, size, platforms, uses, icon, file, readmeFile` |

Build-time public client ids (`DISCORD_CLIENT_ID`, `SPOTIFY_CLIENT_ID`, `TWITCH_CLIENT_ID`,
`GOOGLE_CLIENT_ID`, `KNOBDECK_GOOGLE_TOKEN_URL`, `VOICEMOD_CLIENT_KEY`) come from Actions
variables (repository, then organization). Never put secrets in them: they are compiled into the
bundles.

Releasing and listing a plugin is not part of this workflow: see
[knobdeck-registry](https://github.com/Knobdeck/knobdeck-registry) for the catalog.
