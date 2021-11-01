# Pagemachine Renovate Config

[Organization level presets](https://docs.renovatebot.com/config-presets/#organization-level-presets) configuration for Renovate:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": [
    "config:base",
    ":separatePatchReleases"
  ],
  "commitMessagePrefix": "[TASK] ",
  "commitMessageTopic": "{{depName}}",
  "commitMessageExtra": " ({{{displayFrom}}} => {{{displayTo}}})",
  "composerIgnorePlatformReqs": null,
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
      "matchPackageNames": [
        "phpstan/phpstan",
        "jangregor/phpstan-prophecy",
        "saschaegerer/phpstan-typo3"
      ],
      "groupName": "PHPStan"
    }
```

This rule groups all PHPStan-related packages into a single update. This avoids broken update PRs caused by major version bumps.

```json
    {
      "matchSourceUrlPrefixes": [
        "https://github.com/TYPO3-CMS/"
      ],
      "groupName": "TYPO3 CMS",
      "extends": [
        ":disableMajorUpdates"
      ]
    }
```

This rule groups [all TYPO3 packages](https://github.com/orgs/TYPO3-CMS/repositories) into a single update. Technically all TYPO3 packages must be updated at once since all depend on each other with the same version. Also major TYPO3 updates (e.g. from v9 to v10) will not be suggested. These basically always need proper preparation and migration as well as other package updates.

Finally see the Renovate docs about [Configuration Options](https://docs.renovatebot.com/configuration-options/) and [Default Presets](https://docs.renovatebot.com/presets-default/) for all features and tweaks.
