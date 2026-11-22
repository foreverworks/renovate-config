# renovate-config

Shared [Renovate](https://docs.renovatebot.com/) presets for the Forever Works repos, so every repo
gets the same weekly rhythm and an Expo app never receives a PR that its SDK would undo.

## The presets

- **`github>foreverworks/renovate-config`** (`default.json`): the base. PRs arrive before 6 am on
  Monday, Bucharest time, and nobody but the owner merges them. A major version waits on the
  Dependency Dashboard until someone ticks it. npm minors and patches come as one PR, three days
  after release, and GitHub Actions as another.
- **`github>foreverworks/renovate-config:expo`** (`expo.json`): the base, plus the Expo rule. An Expo
  SDK decides the versions of React, React Native and every native module it bundles, so Renovate
  moves only `expo`. The `expo-fix` workflow then runs `npx expo install --fix` on that PR's branch
  and commits the versions that SDK names. An SDK major (57 to 58) waits on the dashboard, and its
  PR lists what to check before merging.

## Using it in a repo with an Expo app

1. `renovate.json` at the root, from `examples/renovate.json`.
2. `.github/workflows/expo-fix.yml`, from `examples/expo-fix.yml`, with the app's directory, the
   repo's runner label, and the CI workflow to re-run on the fixed commit. That workflow needs a
   `workflow_dispatch:` trigger, because a commit pushed with the workflow's own token starts no
   checks by itself.
3. The repo added to the Renovate app's installation (GitHub, organization settings, Installed
   GitHub Apps, Renovate, Repository access).

A `react-native-*` library that Expo does not bundle is switched off by the Expo rule. Turn it back
on in the repo's own `renovate.json`:

```json
{ "matchPackageNames": ["react-native-mmkv"], "enabled": true }
```

## Why the fix runs in a workflow

The hosted Renovate app cannot run commands on its branches, and an `expo` bump alone leaves React
Native and the native modules on the old SDK's versions: the build then fails, or `expo doctor`
flags every one of them. The workflow installs with `--ignore-scripts`, because the branch is not
reviewed yet and its job holds a token that can push.
