# version-matrix-php80-composer26

## Purpose

This is a version-matrix probe. Its purpose is to validate Mend's manual-versioning behavior across the supported
toolchain space. Specifically, it exercises detection of a Composer project locked with PHP 8.0 and Composer 2.6
to ensure Mend resolves the dependency tree correctly when the `.whitesource` file pins these legacy-but-supported
toolchain versions.

## Pinned Toolchain

- PHP version: 8.0
- Composer version: 2.6
- Docker image used to generate `composer.lock`: `composer:2.6` (Composer 2.6.6, PHP 8.3 in the image — `--ignore-platform-reqs` was used because no `composer:2.6-php8.0` tag exists; the lockfile platform constraint is `>=8.0`)

## Lockfile proof

The `composer.lock` contains:

```json
"plugin-api-version": "2.6.0"
```

This confirms that Composer 2.6 was used to produce the lockfile.

## Expected `.whitesource` `scanSettings.versioning` content

When the `.whitesource` file is added in the follow-up workflow step, it must contain:

```json
{
  "scanSettings": {
    "versioning": {
      "php": "8.0",
      "composer": "2.6"
    }
  }
}
```

Do NOT include `.whitesource` in this repo — it is added separately.

## Feature exercised

Lockfile-driven detection of a minimal Packagist registry project pinned to the PHP 8.0 + Composer 2.6 toolchain
cell in the version-matrix coverage plan.

## Expected dependency tree

- `monolog/monolog` 2.11.0 — direct production dependency, source: Packagist registry
  - `psr/log` 3.0.2 — transitive production dependency, source: Packagist registry

Total packages: 2 (1 direct, 1 transitive). No dev packages.

## Mend detection expectations

- Both packages must appear in Mend's dependency tree.
- Both are sourced from Packagist (registry source).
- No packages should be missing — the transitive chain is shallow and unambiguous.
- The resolved versions must match the lockfile exactly: `monolog/monolog:2.11.0`, `psr/log:3.0.2`.