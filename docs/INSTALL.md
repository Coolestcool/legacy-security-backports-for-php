# Apply, build, verify and roll back

## 1. Identify the actual runtime

Inventory the CLI, CGI, Apache module, FPM pools and containers that serve your
applications. Record exact package revisions, executable paths, build options,
INI locations, loaded extensions and service definitions. `php -v` describes
only the CLI selected by PATH; it does not prove what Apache or FPM uses.

Useful read-only commands, adjusted to your environment:

```sh
php -v
php --ini
php -m
php-config --configure-options
```

Treat phpinfo/configuration output as private. Do not post credentials or service
configuration in an issue. Obtain the source corresponding to your installed
package or image and retain all existing distribution/security patches.
The tested source releases are **5.6.40** and **7.2.34**, not every 5.6.x/7.2.x.

## 2. Download a pinned release

```sh
git clone --branch v0.1.0 https://github.com/Coolestcool/legacy-security-backports-for-php.git
cd legacy-security-backports-for-php
sha256sum -c SHA256SUMS
```

Alternatively download and extract the source archive from the release, then
verify the SHA256SUMS inside it. The helper requires the repository layout; keep
patches/ and tools/ together. Standalone patch assets have a separate
`SHA256SUMS.assets` manifest listing their flat download filenames. On macOS use `shasum -a 256 -c SHA256SUMS`.

For an official pristine-source lab, the archives and hashes are in
[sources.json](../sources.json). For a distribution runtime, start from the
matching distribution source instead. Archive checksums are pinned bytes, not
proof of provenance by themselves; verify the upstream signatures as available.

## 3. Check and apply to a disposable source tree

Requirements: Python 3.8+, a working `patch` command and a writable temporary
directory. The default command modifies only temporary copies of patch target
files; it leaves your source tree unchanged:

```sh
python3 tools/prepare-source.py /path/to/php-5.6.40
python3 tools/prepare-source.py /path/to/php-7.2.34
```

Choose the command for your release, then explicitly apply:

```sh
python3 tools/prepare-source.py /path/to/php-5.6.40 --apply
# OR:
python3 tools/prepare-source.py /path/to/php-7.2.34 --apply
```

PHP 7.2's ZSTR_MAX_LEN prerequisite is applied first if neither prerequisite macro
exists. It is skipped if both are already defined. A partial prerequisite,
checksum mismatch, unexpected version, rejected hunk or fuzzy application stops
the operation. An already patched main tree is rejected, not silently accepted.
Do not use `--force`, manually ignore rejects or apply the two main version
patches to the same tree. Use a clean working tree and preserve its original
source snapshot; this helper is a source preparation tool, not a deployment tool.

Manual equivalent, from the corresponding source directory after inspecting
patch contents and validating their checksums:

```sh
# PHP 5.6.40 only; set PATCH_REPO to the absolute clone path.
patch --batch --forward --fuzz=0 --dry-run -p1 < "$PATCH_REPO/patches/php56-php2shell.patch"
patch --batch --forward --fuzz=0 -p1 < "$PATCH_REPO/patches/php56-php2shell.patch"

# PHP 7.2.34 only, if both prerequisite macros are absent:
patch --batch --forward --fuzz=0 --dry-run -p1 < "$PATCH_REPO/patches/php72-prerequisite-ZSTR_MAX_LEN.patch"
patch --batch --forward --fuzz=0 -p1 < "$PATCH_REPO/patches/php72-prerequisite-ZSTR_MAX_LEN.patch"
# Then check and apply the main 7.2 patch:
patch --batch --forward --fuzz=0 --dry-run -p1 < "$PATCH_REPO/patches/php72-php2shell.patch"
patch --batch --forward --fuzz=0 -p1 < "$PATCH_REPO/patches/php72-php2shell.patch"
```

## 4. Rebuild in a matching environment

Use your distribution's package build process or your reproducible image build.
Preserve compiler flags, prefix, config paths, thread safety, architecture, PHP
module ABI, every required extension and linked libraries. Record new hashes.
For example, Debian source packages are normally rebuilt with `dpkg-buildpackage`
using their recorded build dependencies and quilt series. Add the patch through
that packaging workflow; a regenerated package source must include it too.
If a vendor already shipped the fix, use that package rather than applying it
again. Review vendor changes instead of attempting to remove them.

For a **minimal laboratory build only**, not a replacement for a packaged PHP:

```sh
./configure --disable-all --enable-cli --enable-cgi --enable-fpm   --enable-session --enable-json --enable-hash --enable-filter
make -j2
python3 "$PATCH_REPO/tests/check-runtime.py" sapi/cli/php sapi/cgi/php-cgi
```

This example intentionally omits production extensions and Apache integration.
It must not be installed over the existing runtime. PHP 5.6's Apache module,
CGI and CLI are distinct artifacts; FPM/CLI and extension artifacts must likewise
match the PHP 7.2 image. Merely compiling CLI leaves the web runtime vulnerable.

## 5. Stage and verify before deployment

Back up the original executables, Apache module, extensions, INI files, wrappers,
service definitions and image/Compose configuration. Preserve ownership, modes,
exact image IDs and checksums. Test restore in an isolated copy. Keep independent
console access and a maintenance window appropriate to your system.

Validate in staging:

- PHP version, extension list, INI paths and dependencies match the original.
- Real requests exercise each active SAPI and expected application route.
- GET/POST, persistent sessions and normal multipart uploads work.
- Upload-progress tracking is intentionally disabled by the backport. An INI
  value of `1` does not mean tracking is active; check behavior in isolation.
- Authentication and representative read/write workflows pass with test data.
- OOM termination behavior is checked only in a bounded laboratory process.
- Logs show no new loader errors, missing symbols, fatal/parse errors or crashes.

The CLI/CGI fixture is deliberately local and uses no HTTP listener. It is not
an FPM/Apache integration test and does not certify full application behavior.

## 6. Deploy the built artifacts, then replace every old worker

Use your reviewed package/image deployment process. For Docker, deploy a pinned
new image, retain the prior image and configuration, and recreate only affected
services. Avoid rebuilding arbitrary floating tags in production. Revalidate any
reverse-proxy upstream resolution if a recreated container changes address.

Check that Apache masters, CGI processes and every FPM pool use the new artifacts.
A service manager's success status is not proof: inspect the running executable
and loaded module paths, including old/deleted mappings where supported. Restart
or gracefully retire stale workers using your platform's service procedure.
Re-run the application's agreed smoke checks and review logs after activation.

No universal binary replacement or root installer is supplied because package
layouts, libraries, SAPIs and deployment controls differ between installations.

## 7. Rollback

If agreed checks fail, restore the saved packages/binaries, module, configuration
and prior container image; recreate/restart the affected services and remove
stale workers. Check runtime hashes and real application responses again.
A source-level reverse patch is **not** a rollback of already installed binaries.
Do not reset databases or repair tables as part of this PHP rollback.

Rollback restores service compatibility but also restores the previous security
exposure. Investigate in staging before another deployment attempt. Keep a
record of which artifact is currently serving each application.
