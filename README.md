# renovate-config

Shared [Renovate](https://docs.renovatebot.com/) presets: one weekly rhythm for dependency updates
across a set of repositories, and a rule that stops an Expo app from receiving a PR that its SDK
would undo.

## Who this is for

These presets are what the Forever Works organization uses for its own repositories, and its
repositories read them live from `main`. They encode one team's choices: pull requests arrive before
6 am on Monday in the **Europe/Bucharest** time zone, nothing merges by itself (apart from one preset
for our own container images), and a new npm release waits three days.

**If you are not us, copy a preset instead of extending it.** Copy `default.json` (and `expo.json`
if you build Expo apps) into your own preset repository or your repo's `renovate.json`, then change
the schedule, time zone and limits to yours. Extending ours means that any edit we push to `main`
changes your Renovate runs the next time they run, with no review on your side. If you do extend it,
you accept that. The `base-images` preset is of no use outside our organization: it names our own
images. For the `expo-fix` workflow, either copy `.github/workflows/expo-fix.yml` into your repo and
call it with `uses: ./.github/workflows/expo-fix.yml`, or keep calling ours but pin it to a commit
(`uses: foreverworks/renovate-config/.github/workflows/expo-fix.yml@<commit sha>`) rather than `@main`.

## What you need

- The [Renovate GitHub app](https://github.com/apps/renovate) (or your own Renovate) installed on
  the repository.
- For the Expo preset: an Expo app with an `npm` lockfile, and GitHub Actions enabled. The workflow
  runs `npx expo install --fix`.
- No cost beyond what Renovate and your Actions minutes already cost.

## The presets

- **`github>foreverworks/renovate-config`** (`default.json`): the base. PRs arrive before 6 am on
  Monday, Bucharest time, and none is merged automatically. A major version waits on the Dependency
  Dashboard until someone ticks it. npm minors and patches come as one PR, three days after release
 , and GitHub Actions as another. At most six
  PRs are open at once, and commits start with `renovate:`.
- **`github>foreverworks/renovate-config:expo`** (`expo.json`): the base, plus the Expo rule. An Expo
  SDK decides the versions of React, React Native and every native module it bundles, so Renovate
  moves only `expo`. The `expo-fix` workflow then runs `npx expo install --fix` on that PR's branch
  and commits the versions that SDK names. An SDK major (57 to 58) waits on the dashboard, and its
  PR lists what to check before merging. The rule also holds back `typescript`, which Expo pins
  too. The full list of held packages is in `expo.json`.
- **`github>foreverworks/renovate-config:base-images`** (`base-images.json`): the one exception to
  "nothing merges by itself". Our own base images, `ghcr.io/foreverworks/debian` and
  `ghcr.io/foreverworks/node` (built by
  [debian-oci-images](https://github.com/foreverworks/debian-oci-images)), stay pinned by digest, and
  a new release within the same line merges on its own, at any hour, once every check on its PR has
  passed. A security fix in a base image then reaches the repo's main without waiting for Monday. A
  PR with no checks, or a red one, waits for a person. A new major line (Debian 14, the next Node
  LTS) stays a PR. A few of our repos extend it.

## Using it in a repo with an Expo app

1. `renovate.json` at the root, from `examples/renovate.json`. It extends
   `github>foreverworks/renovate-config:expo`. If you copied the presets, replace that line with
   your own copy of `expo.json`, and make the `extends` in your copy of `expo.json` point at your own
   base preset: as shipped it still extends `github>foreverworks/renovate-config`.
2. `.github/workflows/expo-fix.yml`, from `examples/expo-fix.yml`, with the app's directory, the
   runner label (`ubuntu-latest` for a repo on GitHub's runners), and the CI workflow to re-run on
   the fixed commit. Its `uses:` line points at `foreverworks/renovate-config/...@main`: if you copied
   the workflow, change it to `./.github/workflows/expo-fix.yml`; if not, pin the `@main` to a commit
   SHA. The workflow runs on pushes to `renovate/**` branches and needs `contents: write` and
   `actions: write` (the example sets both). The CI workflow it re-runs needs a `workflow_dispatch:`
   trigger, because a commit pushed with the workflow's own token starts no checks by itself.
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

## Questions and changes

Open an issue in this repository. A change to a preset reaches every repo that extends it as soon as
it lands on `main`, so describe what it does to their pull requests.
