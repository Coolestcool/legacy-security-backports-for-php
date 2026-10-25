# Authorship and provenance

**Origin:** Sury public source packages, incorporating upstream PHP work.
**Community repository maintainer:** [Coolestcool](https://github.com/Coolestcool).
This repository redistributes vendor patches with documentation and validation
tooling. It claims no authorship of the original fixes or version backports.
It is a documented mirror/distribution, not an official Sury project or a GitHub
fork of the complete vendor source repository.

**Patch authorship is retained upstream.** All three distributed patch files
were compared byte for byte with the public Sury source packages on
24 October 2026. This project does not claim authorship of the original
vulnerability fixes or the Sury version-specific backports.

| File | Sury source package | Original patch |
| --- | --- | --- |
| `php56-php2shell.patch` | php5.6 5.6.40-105 | `0261-Avoid-state-corruption-via-bailout.patch` |
| `php72-php2shell.patch` | php7.2 7.2.34-66 | `0159-Avoid-state-corruption-via-bailout.patch` |
| `php72-prerequisite-ZSTR_MAX_LEN.patch` | php7.2 7.2.34-66 | `0135-backport-ZSTR_MAX_LEN.patch` |

The main patch headers credit **Arnaud Le Blanc** and **Ilija Tovilo**. The
prerequisite header credits **Remi Collet**, with upstream commit
`37056ad634d9c44bac0d6c8e730eafaec1344840`. PHP Group, Zend Engine contributors
and Sury packaging maintainers retain their respective credits and licenses.
The source backports are identified upstream as minimal mitigations for EOL
versions; do not describe them as the full maintained-branch implementation.

Exact public archive URLs, member names and SHA-256 values are recorded in
[`provenance.json`](provenance.json). The checked-in patch files are unchanged;
their published hashes are in [`SHA256SUMS`](SHA256SUMS).

PHP 5.6.40 and 7.2.34 release LICENSE and Zend/LICENSE files are retained under
LICENSES/. Do not remove notices when distributing modified source or binaries.
Changes to these redistributed patches should include a separate attribution and
a clear description of how they differ from the vendor originals.
