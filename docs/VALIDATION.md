# Validation matrix and limits

## Reproducible publication checks

The release preparation checks use official pristine-source archives listed in
sources.json and the unmodified vendor patches in SHA256SUMS. They check:

- applicability with zero fuzz to PHP 5.6.40;
- the PHP 7.2.34 prerequisite and main patch in that order;
- no modification of the supplied source during default check mode;
- explicit source application and rejection of repeated application;
- checksum mismatch and unexpected-version rejection;
- the complete public file set for deployment data and accidental credentials.

GitHub Actions is enabled on `main` after publication of the initial v0.1.0
archive. The public [run history](https://github.com/Coolestcool/legacy-security-backports-for-php/actions)
is the authority for the actual build/runtime result. The active workflow builds
on Ubuntu 22.04 for each release in the matrix. Historical private runtime tests
below are separate evidence; they must not be substituted for a public CI result.

## Verified public CI result — 24 October 2026 (America/Toronto)

[Run 40149083499](https://github.com/Coolestcool/legacy-security-backports-for-php/actions/runs/40149083499)
completed successfully for commit `2212daf2c73cee25922a1f514769a744b0a76202`.

| Official source | Zero-fuzz patch sequence | Minimal CLI/CGI/FPM build | Local CLI/CGI fixtures |
| --- | --- | --- | --- |
| PHP 5.6.40 | Passed | Passed | Passed |
| PHP 7.2.34 | Passed, including prerequisite | Passed | Passed |

The first CI run caught a syntax error in the new compatibility fixture. It was
corrected before this successful run; the vendor patches did not change. The
v0.1.1 release contains the corrected fixture. Source changes should be tested
again; documentation-only updates do not extend this test coverage.

## Runtime fixtures

`tests/check-runtime.py CLI CGI` checks isolated binaries with `-n`, temporary
session storage and no web listener. It checks a basic CLI compatibility fixture,
bounded OOM process termination without a user shutdown callback, CGI GET and
POST decoding, session persistence across separate CGI processes, a normal
multipart upload digest and the absence of upload-progress records.

The minimal configure recipe enables CLI, CGI and FPM. The fixtures exercise
CLI and CGI; they do not start FPM or Apache. Running `php-fpm -t` without the
actual pool configuration would not prove production compatibility.

## Historical deployment evidence

Before publication, these same patch bytes were built and exercised in one
isolated legacy environment, then deployed on 22 October 2026 to a PHP 5.6.40
vendor-derived installation (CLI/CGI/Apache) and a PHP 7.2.34-FPM image. The
private record includes artifact hash verification, request/session/upload
checks, worker replacement and rollback activity. That is one environment's
historical result, not a reproducible certification for other users.

No production logs, server identities, addresses, credentials, binary artifacts,
rollback backups, customer data or original service configuration are published.

## Deliberate behavior changes

- OOM handling may exit the PHP worker immediately with status 1. User shutdown
  functions and graceful per-request error handling are bypassed on that path.
  Supervisors/pools must replace workers; interrupted requests can fail.
- Built-in session upload-progress tracking is disabled by an unconditional early
  return. Normal multipart file upload support is retained.
- Source preparation does not update installed packages or active workers.
- Package version labels may remain unchanged after a local rebuild. Track
  artifact hashes and provenance; a version-only scanner may not see a backport.

## Not established by these checks

Complete mitigation of every possible state-lifecycle variant, absence of other
CVEs, ZTS/Windows/32-bit compatibility, arbitrary extension safety, broad distro
coverage, application correctness and end-to-end authenticated workflows are
not certified. Report additional tested matrices through reviewed contributions.
