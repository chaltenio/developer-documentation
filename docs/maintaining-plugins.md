---
category: DevelopInDepth
title: Maintaining Plugins
---
# Matomo core - Maintaining Plugins

This guide applies to Matomo core developers. If you develop a plugin and want to share it as a contributor please check out the guide for [distributing plugins](https://developer.matomo.org/guides/distributing-your-plugin).

## How to add a new plugin 

* Get Matt or Thomas to create a new github repository under `matomo-org`. The name for the plugin usually starts with `plugin-` followed by the plugin name. You can for example ping us in our slack. We use `*.x-dev` as the main branch where `*` would be replaced by the latest major version of Matomo.
* Push the code
* Add the plugin's repository as a submodule to our main matomo repository `git submodule add {http-plugin-github-address} plugins/{pluginname}`
* Edit `.gitmodules` and move the entry for this submodule before the comment at the bottom (see description of that comment)
* It's expected that some core screenshots will fail and maybe also some integration or system tests as the plugin will be activated and the tests for this plugin run during core tests too. This is because we consider the plugins bundled w/ core to be tested this often (testing compatibility on every matomo change). Otherwise the tests would only run when the plugin changes, even though a change in core could cause problems.

### New plugin checklist

* The new plugin has a `LICENSE` file similar to https://github.com/matomo-org/plugin-QueuedTracking/blob/4.x-dev/LICENSE
* The same license is also defined in `plugin.json`
* A descriptive description and useful keywords are configured in `plugin.json`. Same for other values in the plugin JSON file.
* If any possible screenshots are defined in the `screenshots` folder.
* If `docs/index.md` or `docs/faq.md` are not needed then delete them


## Maintaining plugin FAQs

### How do I fix the error "Some Screenshots are not stored in LFS"?

The error might look like this:

```
1) Piwik\Tests\Integration\ReleaseCheckListTest::test_screenshotsStoredInLfs
   Some Screenshots are not stored in LFS: plugins/YourPluginName/tests/UI/expected-screenshots/Filename.png
   Failed asserting that an array is empty.
```

You can fix this issue using these steps:

* `cd plugins/YourPluginName`
* `git rm tests/UI/expected-screenshots/*.png`
* Add `tests/UI/expected-screenshots/*.png filter=lfs diff=lfs merge=lfs` to the file `.gitattributes` in your plugin
* `git add tests .gitattributes`
* `git commit -m 'Remove screenshots'`
* `Add the expected UI test files again`
* `git add tests`
* `git commit -m 'Add screenshots using LFS'`
* Then update the submodule in core

#### Get travis to use LFS

To get travis to checkout the screenshots correctly and use LFS you will need to add/change the `.travis.yml` within your plugin like this (eg [see this file](https://github.com/matomo-org/tag-manager/blob/4.x-dev/.travis.yml#L65-L68)):

```
before_install:

  - if [[ "${TEST_SUITE}" == "UITests" ]]; then git lfs fetch; git lfs checkout; fi
```

And you need to create a file `tests/travis/before_install.after.yml` in your plugin with the following content (eg [see this file](https://github.com/matomo-org/tag-manager/blob/4.x-dev/tests/travis/before_install.after.yml)):

```
- if [[ "${TEST_SUITE}" == "UITests" ]]; then git lfs fetch; git lfs checkout; fi
```
