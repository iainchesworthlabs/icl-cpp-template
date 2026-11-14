# Continuous integration

The workflows in `.github/workflows/` call reusable workflows in `icl-common-infra`, pinned by
commit SHA, always at the newest `icl-common-infra` tag (`gh api repos/iainchesworthlabs/icl-common-infra/tags --jq '.[0].name'`).
A move to a new tag is one pull request made with `tools/ci/repin.py --infra <checkout at the tag> --version <tag>`, which
changes every `uses:` line and the vendored CMake modules in `cmake/icl` together. Dependabot proposes the same bump, but
do not add a commit to its branch: GitHub then adds a second `Co-authored-by` line to the squash message and the commit
text check rejects it in the merge queue. Close the Dependabot pull request and open the `repin.py` one instead.

## What runs

| Event | Jobs |
|---|---|
| Draft pull request | Policy, Plan, workflow analysis, dependency scan |
| Ready pull request | Above, plus Linux release and sanitizer builds and coverage |
| Merge queue, push to `develop` | Above, plus Linux debug and TSan, macOS, Windows release and ASan |
| Change to what the iOS build reads (a path filter), or a manual run | iOS compile check (`ios-build.yml`, not required) |
| Nightly | CodeQL |
| Tag `v*` | Release |

The Plan job reads `.github/ci/plan.json`. A change to documentation only builds nothing. A path
the plan does not recognise builds everything.

## Required checks

Two jobs defined in `ci.yml` are the required checks: **CI Status** and **Scan dependency diff**.
CI Status fails when any job failed or was cancelled, and when a job the plan asked for was
skipped. Do not require checks from the called workflows by name; their names carry the caller's
job name as a prefix.

`ios-build.yml` is a separate workflow because a macOS runner is billed at a multiple and a required check that a path
filter keeps from running never reports. It compiles `apps/ios` unsigned with the `ios-release` preset and looks inside
the bundle; it needs no secret. The signed build and the TestFlight upload are the `ios` job of `release.yml`
(`docs/releasing.md`).

## Run the checks locally

```bash
python3 tools/ci/precheck.py
```

It needs a checkout of `icl-common-infra` next to this repository or in `ICL_COMMON_INFRA`.
