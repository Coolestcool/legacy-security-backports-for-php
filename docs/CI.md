# Optional GitHub Actions validation

The initial release publishes a workflow template at
[ci/validate.yml](ci/validate.yml), rather than an installed workflow. The
publishing CLI credential could create the repository and push source, but
GitHub rejected workflow installation because it lacks the `workflow` scope.
No CI success is claimed. No additional permissions were requested or granted.

A repository maintainer with the appropriate GitHub workflow permissions may
review the template and install it:

```sh
mkdir -p .github/workflows
cp docs/ci/validate.yml .github/workflows/validate.yml
git add .github/workflows/validate.yml
git commit -m "Enable legacy source validation workflow"
git push
```

The template downloads checksum-pinned official source archives, applies the
patches, builds minimal CLI/CGI/FPM and exercises the local CLI/CGI fixtures on
Ubuntu 22.04. It uses read-only repository permissions and no secrets. FPM is
built, not started by the fixture; Apache and application integration remain
separate checks. Review the run results before claiming a supported matrix.
