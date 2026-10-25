# GitHub Actions validation

The active workflow is [.github/workflows/validate.yml](../.github/workflows/validate.yml).
It was installed through the GitHub web interface after v0.1.0 was published,
without expanding the publishing CLI credential's permissions. The initial
immutable source archive predates this activation.

Consult [Actions](https://github.com/Coolestcool/legacy-security-backports-for-php/actions)
for run status before claiming a tested matrix. The workflow downloads
checksum-pinned official source archives, applies patches with zero fuzz, builds
minimal CLI/CGI/FPM and exercises local CLI/CGI fixtures on Ubuntu 22.04.
It uses read-only repository permissions and no secrets. FPM is built, not
started by the fixture; Apache and application integration are separate checks.

A copy at [ci/validate.yml](ci/validate.yml) is available as a template for forks.
When changing the workflow, update both copies. A publishing token lacking the
`workflow` permission cannot create or edit the active workflow; a maintainer
may use GitHub's editor with their existing authorized account instead. No
permission changes are required merely to download patches or submit a PR.
