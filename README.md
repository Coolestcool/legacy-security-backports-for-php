# Legacy Security Backports for PHP

Community distribution and validation tools for the **PHP 5.6.40** and
**PHP 7.2.34** source backports targeting **CVE-2026-109618** and
**CVE-2026-109619** (PHP2Shell).

A community redistribution of **Sury vendor backports**, with an application
guide and validation tooling. Repository maintained by
[Coolestcool](https://github.com/Coolestcool). All patch authorship stays with
the original authors; no original fix is claimed here. This is not an official
Sury repository or a GitHub fork of its packaging history. See
[provenance](PROVENANCE.md).

## Scope

These are source patches for administrators maintaining legacy systems while
planning a supported migration. They preserve the PHP release number. They do
**not** turn an end-of-life runtime into a supported or comprehensively secured
release. They are not an official PHP project release or an LTS service.

The mitigation changes out-of-memory handling to terminate the affected process
immediately, resets selected request state and disables session upload-progress
tracking in code. An OOM request will fail; shutdown callbacks will not run on
that path. Ordinary file uploads remain supported and must be tested separately.

| Target | Main patch | Prerequisite |
| --- | --- | --- |
| PHP 5.6.40 | [php56-php2shell.patch](patches/php56-php2shell.patch) | None for the official 5.6.40 source tree |
| PHP 7.2.34 | [php72-php2shell.patch](patches/php72-php2shell.patch) | [ZSTR_MAX_LEN backport](patches/php72-prerequisite-ZSTR_MAX_LEN.patch), unless those macros already exist |

Each main patch targets **both CVEs**; the two main patches are alternatives for
the two PHP versions, not one patch per vulnerability. Package revisions such as
`5.6.40-38` include distribution-specific patches: retain those patches and build
from the matching package source. Do not replace the vendor runtime with a
minimal vanilla build just because its version string matches.

## Download and prepare a source tree

Download a tagged release from [Releases](https://github.com/Coolestcool/legacy-security-backports-for-php/releases)
or clone this repository at the initial release:

```sh
git clone --branch v0.1.1 https://github.com/Coolestcool/legacy-security-backports-for-php.git
cd legacy-security-backports-for-php
sha256sum -c SHA256SUMS

# Default: validate the complete sequence in temporary copies; leave source unchanged.
python3 tools/prepare-source.py /path/to/php-5.6.40
# Or use /path/to/php-7.2.34; its prerequisite is handled automatically.

# Explicitly apply to a disposable, backed-up source working tree after validation.
python3 tools/prepare-source.py /path/to/php-5.6.40 --apply
```

On macOS, use `shasum -a 256 -c SHA256SUMS` instead of `sha256sum`.
Checksums detect changes relative to the checked-out manifest; they are not a
signature or an independent authenticity guarantee.

The helper never installs binaries, changes php.ini, restarts services or
contacts a server. It refuses unexpected source versions, patch checksum
mismatches, rejected hunks and fuzzy application. It does not treat an already
patched or incompatible tree as success.

**Applying a patch to source does not patch the running server.** Rebuild the
matching packages/image, validate every active SAPI and replace the running
workers as described in the [installation and rollback guide](docs/INSTALL.md).

- [Guía en español](docs/INSTALL.es.md)
- [Validation and known limits](docs/VALIDATION.md)
- [Security scope](SECURITY.md)
- [Contributing](CONTRIBUTING.md)
- [Credits and provenance](PROVENANCE.md)

## Validation

Publication checks confirmed zero-fuzz applicability against official release
archives with recorded SHA-256 values. GitHub Actions is now enabled on `main`
and builds minimal CLI/CGI/FPM targets with local CLI/CGI smoke tests. Consult
[the actual runs](https://github.com/Coolestcool/legacy-security-backports-for-php/actions)
for their status; a workflow file alone is not proof of success. See
[CI setup](docs/CI.md) and [validation limits](docs/VALIDATION.md).

The immutable `v0.1.0` archive predates workflow activation and contains a
compatibility-fixture syntax error corrected in **v0.1.1**. Download v0.1.1 or
use the current branch for testing. The three Sury patch files are unchanged.

## Contributing

Issues and pull requests are welcome, especially reproducible build reports,
additional distribution coverage, regression tests and reviewed follow-up fixes.
Never attach production credentials or customer data. For a new exploitable
vulnerability, use the private reporting guidance in [SECURITY.md](SECURITY.md).

## References and license

- [PHP security announcement, 21 October 2026](https://www.php.net/archive/2026.php#2026-10-21-5)
- [CVE-2026-109618 advisory](https://github.com/php/php-src/security/advisories/GHSA-w8hw-w9fx-j79w)
- [CVE-2026-109619 advisory](https://github.com/php/php-src/security/advisories/GHSA-6wmf-4rv5-p3rp)
- [PHP 7.4 source backport](https://github.com/php/php-src/tree/PHP-7.4-CVE-2026-109618-109619)

Patch code retains its upstream PHP/Zend licensing. Supporting project materials
are MIT licensed; see [LICENSE](LICENSE) and [LICENSES](LICENSES/). This product
includes PHP software, freely available from <http://www.php.net/software/>.
