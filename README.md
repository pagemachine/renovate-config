# Pagemachine Renovate Config

[Organization level presets](https://docs.renovatebot.com/config-presets/#organization-level-presets) configuration for Renovate.

See the Renovate docs about [Configuration Options](https://docs.renovatebot.com/configuration-options/) and [Default Presets](https://docs.renovatebot.com/presets-default/) for all features and tweaks.

## Default preset

Usage in `renovate.json`:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["local>pagemachine/renovate-config"]
}
```

The `default.json`:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": [
    "config:recommended",
    ":separateMultipleMajorReleases",
    ":separatePatchReleases"
  ],
  "commitMessagePrefix": "[TASK] ",
  "commitMessageTopic": "{{depName}}",
  "commitMessageExtra": " ({{{displayFrom}}} => {{{displayTo}}})",
  "platformAutomerge": true,
  "rangeStrategy": "update-lockfile",
  "packageRules": [
    // See below
  ]
}
```

The various parts explained:

- [`config:recommended`](https://docs.renovatebot.com/presets-config/#configrecommended) is the default configuration for all languages.
- [`:separateMultipleMajorReleases`](https://docs.renovatebot.com/presets-default/#separatemultiplemajorreleases) separates multiple major (X.y.z) updates. Skipping over major releases usually requires more work than updating in sequence.
- [`:separatePatchReleases`](https://docs.renovatebot.com/presets-default/#separatepatchreleases) separates minor (x.Y.z) and patch (x.y.Z) updates. Patch updates are usually safe to apply right away. Minor updates provide new features and should be reviewed more thoroughly.
- [`commitMessagePrefix`](https://docs.renovatebot.com/configuration-options/#commitmessageprefix), [`commitMessageTopic`](https://docs.renovatebot.com/configuration-options/#commitmessagetopic), [`commitMessageExtra`](https://docs.renovatebot.com/configuration-options/#commitmessageextra) slightly reformat the Renovate commit messages:

      [TASK] Update <package> (<old-version> => <new-version>)

- [`platformAutomerge`](https://docs.renovatebot.com/configuration-options/#platformautomerge) enables the platform-native auto-merge capabilities so that update PRs can be merged automatically without interaction.
- [`rangeStrategy`](https://docs.renovatebot.com/configuration-options/#rangestrategy) prefers lockfile updates (e.g. `composer.lock`), otherwise updates version constraints (e.g. `^1.0` to `^2.0`)
- [`packageRules`](https://docs.renovatebot.com/configuration-options/#packagerules):

```json
    {
      "matchDepTypes": [
        "devDependencies",
        "require-dev"
      ],
      "automerge": true
    }
```

This rule enables automerge for all kind of development dependencies, namely for NPM/Yarn (`devDependencies`) and Composer (`require-dev`).

```json

    {
      "matchDepTypes": [
        "require"
      ],
      "matchPackageNames": [
        // ...
      ],
      "matchUpdateTypes": [
        "minor",
        "patch"
      ],
      "automerge": true
    }
```

This rule enables automerge for a selected list of reqular Composer dependencies.

```json
    {
      "matchPackageNames": [
        // ...
      ],
      "minimumReleaseAge": "1 day"
    },
```

This rule enforces a delay of 1 day for update processing like automerge for a selected list of packages.

```json
    {
      "matchDatasources": ["docker"],
      "matchPackageNames": [
        // ...
      ],
      "enabled": false
    },
```

This rule disables update suggestions for a selected list of Docker images. Usually these updates could not be completed without further changes.

```json
    {
      "matchDatasources": ["docker"],
      "matchPackageNames": [
        // ...
      ],
      "matchUpdateTypes": [
        "patch"
      ],
      "automerge": true
    },
```

This rule enables automerge for patch updates of a selected list of Docker images.

```json
    {
      "matchDepNames": ["php"],
      "enabled": false
    }
```

This rule disables all PHP update suggestions. Updating the PHP version or widening the range of supported PHP versions is almost always accompanied by further changes like Docker image updates and CI adjustments. Thus it currently does not make much sense to have Renovate suggest such updates.

```json
    {
      "matchPackagePrefixes": [
        // ...
      ],
      "matchPackageNames": [
        // ...
      ],
      "groupName": "PHPStan",
      "separateMajorMinor": false
    }
```

This rule groups all PHPStan-related packages into a single update. This avoids broken update PRs caused by major version bumps.

```json
    {
      "matchPackagePrefixes": [
        "typo3/cms-"
      ],
      "excludePackageNames": [
        "typo3/cms-cli",
        "typo3/cms-composer-installers"
      ],
      "groupName": "TYPO3 CMS",
      "labels": [
        "typo3"
      ],
      "prPriority": 5,
      "extends": [
        ":disableMajorUpdates"
      ]
    }
```

This rule groups [all TYPO3 packages](https://github.com/orgs/TYPO3-CMS/repositories) into a single update. Technically all TYPO3 packages must be updated at once since all depend on each other with the same version.

A label is added to update PRs for easier identification across projects.

The priority is raised over other updates to `5` (arbitrary, higher than the default `0`).

Major TYPO3 updates (e.g. from v9 to v10) will not be suggested. These basically always need proper preparation and migration as well as other package updates.

## TYPO3 Extension preset

Usage in `renovate.json`:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["local>pagemachine/renovate-config:typo3-cms-extension"]
}
```

The `typo3-cms-extension.json`:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["local>pagemachine/renovate-config"],
  "packageRules": [
    // See below
  ]
}
```

The various parts explained:

- [`local>pagemachine/renovate-config`](#default-preset) is the default preset explained above.
- [`packageRules`](https://docs.renovatebot.com/configuration-options/#packagerules):

```json
    {
      // ...
      "rangeStrategy": "widen"
    }
```

These rules override the [`rangeStrategy`](https://docs.renovatebot.com/configuration-options/#rangestrategy) for some selected Composer dependencies.

By default new major version updates of dependencies would lead to a replaced version constraint:

```diff
-"typo3/cms-core": "^10.4",
+"typo3/cms-core": "^11.5",
```

With `widen`, the new major version is added instead:


```diff
-"typo3/cms-core": "^10.4",
+"typo3/cms-core": "^10.4 || ^11.5",
```

This is a good practice for TYPO3 extensions to support at least 2 consecutive TYPO3 major versions for smooth upgrades.

The same goes for a few other dependencies indirectly related to TYPO3, e.g. PHPStan.
