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
    "config:base",
    ":disableDependencyDashboard",
    ":separatePatchReleases"
  ],
  "commitMessagePrefix": "[TASK] ",
  "commitMessageTopic": "{{depName}}",
  "commitMessageExtra": " ({{{displayFrom}}} => {{{displayTo}}})",
  "composerIgnorePlatformReqs": null,
  "platformAutomerge": true,
  "rangeStrategy": "update-lockfile",
  "packageRules": [
    // See below
  ]
}
```

The various parts explained:

- [`config:base`](https://docs.renovatebot.com/presets-config/#configbase) is the default base configuration for all languages.
- [`:disableDependencyDashboard`](https://docs.renovatebot.com/presets-default/#disabledependencydashboard) disables the Dependency Dashboard since it implicitly requires the GitLab _Issues_ feature to be enabled
- [`:separatePatchReleases`](https://docs.renovatebot.com/presets-default/#separatepatchreleases) separates minor (x.Y.z) and patch (x.y.Z) updates. Patch updates are usually safe to apply right away. Minor updates provide new features and should be reviewed more thoroughly.
- [`commitMessagePrefix`](https://docs.renovatebot.com/configuration-options/#commitmessageprefix), [`commitMessageTopic`](https://docs.renovatebot.com/configuration-options/#commitmessagetopic), [`commitMessageExtra`](https://docs.renovatebot.com/configuration-options/#commitmessageextra) slightly reformat the Renovate commit messages:

      [TASK] Update <package> (<old-version> => <new-version>)

- [`composerIgnorePlatformReqs`](https://docs.renovatebot.com/configuration-options/#composerignoreplatformreqs) lets Composer respect platform requirements (e.g. PHP version)
- [`platformAutomerge`](https://docs.renovatebot.com/configuration-options/#platformautomerge) enables the platform-native auto-merge capabilities so that update PRs can be merged automatically without interaction.
- [`rangeStrategy`](https://docs.renovatebot.com/configuration-options/#rangestrategy) prefers lockfile updates (e.g. `composer.lock`), otherwise updates version constraints (e.g. `^1.0` to `^2.0`)
- [`packageRules`](https://docs.renovatebot.com/configuration-options/#packagerules):

```json
    {
      "matchDepTypes": [
        "devDependencies",
        "require-dev"
      ],
      "matchUpdateTypes": [
        "minor",
        "patch"
      ],
      "automerge": true
    }
```

This rule enables automerge for all kind of development dependencies, namely for NPM/Yarn (`devDependencies`) and Composer (`require-dev`). This means that patch and minor updates of development packages will be merged automatically if possible.

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
      "matchPackageNames": ["php"],
      "enabled": false
    }
```

This rule disables all PHP update suggestions. Updating the PHP version or widening the range of supported PHP versions is almost always accompanied by further changes like Docker image updates and CI adjustments. Thus it currently does not make much sense to have Renovate suggest such updates.

```json
    {
      "matchPackagePrefixes": [
        "phpstan/"
      ],
      "matchPackageNames": [
        "jangregor/phpstan-prophecy",
        "saschaegerer/phpstan-typo3",
        "rector/rector",
        "slevomat/coding-standard"
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
      "extends": [
        ":disableMajorUpdates"
      ]
    }
```

This rule groups [all TYPO3 packages](https://github.com/orgs/TYPO3-CMS/repositories) into a single update. Technically all TYPO3 packages must be updated at once since all depend on each other with the same version. Also major TYPO3 updates (e.g. from v9 to v10) will not be suggested. These basically always need proper preparation and migration as well as other package updates.

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
      "matchDepTypes": [
        "require"
      ],
      "rangeStrategy": "widen"
    }
```

This rule overrides the [`rangeStrategy`](https://docs.renovatebot.com/configuration-options/#rangestrategy) for Composer dependencies via [`require`](https://getcomposer.org/doc/04-schema.md#require). By default new major version updates of dependencies would lead to a replaced version constraint:

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
