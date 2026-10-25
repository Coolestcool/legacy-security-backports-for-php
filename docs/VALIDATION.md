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

The initial public release does not enable GitHub Actions: the publishing
credential has no workflow scope. A [reviewable template](CI.md) is included,
but no passing public CI build is claimed. The new public CLI/CGI fixture was
not executed on a newly built Linux matrix during release preparation.
Historical private runtime tests below are separate evidence.

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
