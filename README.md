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
  "gitLabAutomerge": true,
  "rangeStrategy": "update-lockfile",
  "packageRules": [
    // See below
  ]
}
```

The various parts explained:

- [`config:base`](https://docs.renovatebot.com/presets-config/#configbase) is the default base configuration for all languages.
- [`:separatePatchReleases`](https://docs.renovatebot.com/presets-default/#separatepatchreleases) separates minor (x.Y.z) and patch (x.y.Z) updates. Patch updates are usually safe to apply right away. Minor updates provide new features and should be reviewed more thoroughly.
- [`commitMessagePrefix`](https://docs.renovatebot.com/configuration-options/#commitmessageprefix), [`commitMessageTopic`](https://docs.renovatebot.com/configuration-options/#commitmessagetopic), [`commitMessageExtra`](https://docs.renovatebot.com/configuration-options/#commitmessageextra) slightly reformat the Renovate commit messages:

      [TASK] Update <package> (<old-version> => <new-version>)

- [`gitLabAutomerge`](https://docs.renovatebot.com/configuration-options/#gitlabautomerge) enables automerge on GitLab using the [_Merge when pipeline succeeds_](https://docs.gitlab.com/ee/user/project/merge_requests/merge_when_pipeline_succeeds.html) feature.
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
      "packagePatterns": ["^typo3/cms-*"],
      "groupName": "TYPO3"
    }
```

This rule groups all TYPO3 packages into a single update. Technically all TYPO3 packages must be updated at once since all depend on each other with the same version.

Finally see the Renovate docs about [Configuration Options](https://docs.renovatebot.com/configuration-options/) and [Default Presets](https://docs.renovatebot.com/presets-default/) for all features and tweaks.
