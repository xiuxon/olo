# olo — app build & test runners

Public CI runner repository. It holds the GitHub Actions workflows that build,
harden, and test Android apps hosted in other repositories. Source code of the
apps and of the hardening pipeline does **not** live here — only the workflows.

> This repo must stay **public**: workflows run here. Everything sensitive is passed
> through encrypted secrets; never commit keys, tokens, or keystores.

## Workflows

| Workflow | Purpose |
| --- | --- |
| `build-app.yml` | Generic app build: assemble + bundle an Android app repo, sign, publish release assets (`<name>.apk`, `<name>.aab`, `<name>.aab.mapping.txt`). Inputs: `repo`, `name`. |
| `build-secure.yml` | Runs the hardening pipeline from `xiuxon/secure-app` against an app config (guard, obfuscation, string encryption, integrity, signing) and publishes the hardened APK/AAB. Input: `app` (config name) plus optional `tag`. |
| `diag-secure.yml` | Emulator gate (API 29/30/33): installs plain + secure builds, asserts startup, onboarding, permission recovery, recompile/tamper integrity legs. |
| `permtest.yml` | Emulator permission-flow test (API 30/33): onboarding, storage/location/notification dialogs, deny rounds, latch behaviour. |
| `forcewall-test.yml` | Emulator hard-force update wall test: wall shown, BACK blocked, tapping UPDATE NOW does not dismiss. |
| `updatetest.yml` | Emulator update-path test: old secure build updated in place to the new one (versionCode-change and same-versionCode integrity paths). |

`build.yml` / `workflow.yml` from the previous life of this repo are not part
of this setup and must not be reintroduced.

## Secrets (repo-level)

| Secret | Value |
| --- | --- |
| `REPO_TOKEN` | Token with write access to this account's repos (workflow checkout, release upload). |
| `EXT_REPO_TOKEN` | Token with read/write access to external (non-`xiuxon/`) repos referenced by app configs. |
| `KEYSTORE_B64` | Base64 of the signing keystore (PKCS12). |
| `STORE_PASS` / `KEY_ALIAS` | Keystore password and key alias. |
| `LOG_KEY` | Optional. Decrypt key for encrypted app diagnostics (`tools/decrypt_log.py` in the hardening repo). |

Workflows resolve the token automatically: repos under this account use
`REPO_TOKEN`, everything else falls back to `EXT_REPO_TOKEN`.

## Conventions

- Releases are tagged `apk-YYYYMMDD-HHMMSS`; build workflows publish assets
  named after the app (`<name>.apk`, `<name>.aab`, mapping file).
- Test workflows are manual (`workflow_dispatch`) with inputs for `repo`,
  `package`, `tag`, and asset names; unknown values auto-detect where possible
  and print the candidates when they cannot.
- **Purge run logs after every dispatch**: `gh api -X DELETE
  repos/<owner>/olo/actions/runs/<id>` for each run, so no build logs linger
  on this public repo. `build-app.yml` and `build-secure.yml` purge their own
  runs automatically on success; the test workflows must be purged manually.
- App configs, hardening tools, and documentation live in
  [`xiuxon/secure-app`](https://github.com/xiuxon/secure-app).
