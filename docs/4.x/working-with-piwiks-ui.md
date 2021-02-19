---
category: Develop
---
# JavaScript and CSS

Read this guide if you'd like to know

* **how to load JavaScript and CSS files in the Piwik UI**
* **how plugins should create JavaScript files**
* **how to work with Piwik Core's JavaScript code**

## JavaScript libraries

Matomo uses the following JavaScript libraries:

* [jQuery](https://jquery.com/) and [jQuery UI](https://jqueryui.com/)
* [AngularJS](https://angularjs.org/)
* [jqPlot](https://www.jqplot.com/)
* A couple of other libraries are used see our [npm dependencies](https://github.com/matomo-org/matomo/blob/4.x-dev/package.json)

**Include new JS libraries only if they are vital to your plugin.** If many plugins decide to use a custom library, the UI will slow down and plugins might get problems when different plugins load different versions of those libraries.

## Loading JS and CSS files in the UI

Plugins can tell Matomo about to load JS and CSS asset files.

This is done through the [`AssetManager.getStylesheetFiles`](/api-reference/events#assetmanagergetstylesheetfiles) and [`AssetManager.getJavaScriptFiles`](/api-reference/events#assetmanagergetjavascriptfiles) events. 
If you are new to events, check out our [events guide](/guides/events). These events are defined in your plugin file (if your plugin name was `YourPluginName` then they would be defined in `$matomoDir/plugins/YourPluginName/YourPluginName.php`). 

To load a file in the UI simply append the paths that should be loaded to the given `$files` array:

```php
// these methods need to be in your plugin php file
public function registerEvents()
{
    return array(
        'AssetManager.getStylesheetFiles' => 'getStylesheetFiles',
        'AssetManager.getJavaScriptFiles' => 'getJavaScriptFiles',
    );
}

public function getStylesheetFiles(&$files)
{
    $files[] = "plugins/MyPlugin/stylesheets/myStylesheet.less";
    $files[] = "plugins/MyPlugin/stylesheets/myCssStylesheet.css";
}

public function getJavaScriptFiles(&$files)
{
    $files[] = "plugins/MyPlugin/javascripts/myJavaScript.js";
}
```

### Asset merging and compiling

In production environments, Matomo will concatenate all JavaScript files into one and minify it. [LESS](https://lesscss.org/) files will be compiled into CSS and merged into one CSS file. Piwik does not merge and minify JavaScript and CSS files on every request as it takes a long time to do this. They are only merged on certain events, such as when enabling a new plugin.

To make sure your changes will be actually visible and executed you need to enable the development mode in case you have not done yet:

```
./console development:enable
```

This way JavaScript files won't be merged and you can debug the original JavaScript source code.

### AngularJS files

If you are using Angular JS, then each plugin should put these files inside an `angularjs` directory. For example the plugin "CoreHome" defines various directives etc in [plugins/CoreHome/angularjs](https://github.com/matomo-org/matomo/tree/4.1.1/plugins/CoreHome/angularjs).

We create a directory for each component which can include a directive, template, controller, etc. The naming for these files is ([click to view an example](https://github.com/matomo-org/matomo/tree/4.1.1/plugins/CoreHome/angularjs/siteselector)):

* `plugins/YourPluginName/angularjs/$component-name/$component-name.controller.js`
* `plugins/YourPluginName/angularjs/$component-name/$component-name.directive.js`
* `plugins/YourPluginName/angularjs/$component-name/$component-name.directive.html`
* `plugins/YourPluginName/angularjs/$component-name/$component-name.directive.less`
* Under circumstances separate logic may be put into a model like `plugins/YourPluginName/angularjs/$component-name/$component-name.model.js`

We prefer to have the controller in a separate file and not in the directive but this might not always be the case.

Generic filters and services that are used by multiple components are usually defined in a `angularjs/common` directory ([see this example](https://github.com/matomo-org/matomo/tree/4.1.1/plugins/CoreHome/angularjs/common)).

#### Module

* The name of our module is `piwikApp`. This means you can register for example a component using `angular.module('piwikApp').component(...)`.
* This module `piwikApp` is configured in [plugins/CoreHome/angularjs/piwikApp.js](https://github.com/matomo-org/matomo/blob/4.1.1/plugins/CoreHome/angularjs/piwikApp.js)


## Special HTML Elements

The following is a list of special elements that you should be aware of as you develop your plugin or contribution:

- `#root`: The root element of everything that is displayed and visible to the user.

- `#content`: The root element that contains everything displayed under the main reporting menu and the row of 'selector' controls (ie, the period selector, the segment selector, etc.).

- `.top_controls`: The element that contains the 'selector' controls. Only one of these elements.

- `.widgetContent`: The root element of each widget. Events are posted to this specific element.

## Global variables defined by Piwik

Piwik defines several global variables (held in `window.piwik`) regarding the current request. Here is what they are and how you should use them:

* `piwik.token_auth`: The **token_auth** of the current user. Should be used in AJAX requests, but should never appear in the URL.
* `piwik.piwik_url`: The URL to this Piwik instance.
* `piwik.userLogin`: The current user's login handle (if there is a current user).
* `piwik.idSite`: The ID of the currently selected website.
* `piwik.siteName`: The name of the currently selected website.
* `piwik.siteMainUrl`: The URL of the currently selected website.
* `piwik.language`: The currently selected language's code (for example, `en`).

## Coding conventions

When writing JavaScript for your contribution or plugin, you would ideally respect the following coding conventions.

## Important JavaScript classes

Piwik Core defines many classes that should be reused by new plugins and contributions. These classes can be used to, among other things, change the page the UI shows, load a popover and get themed color values from CSS.

### Broadcast

This class is considered deprecated and only exists for historical reasons as some components are still using it. Whenever possible you should instead use angularjs components like [$location](https://docs.angularjs.org/api/ng/service/$location). In cases where this is not possible, `broadcast` can be used. 

The `broadcast` object is stored directly in the `window` object and should be used to parse the current URL, load a page in the area below the menu and load persistent popovers.

_Note: Though the object is stored in `window` and not a JS namespace, it can still be accessed using `require` by calling `require('broadcast')`._

#### Parsing the URL

`broadcast` contains several methods for parsing the Piwik URL. Piwik's main UI stores secondary set of query parameters in a URL's hash value. This is the URL that is loaded below the menu. `broadcast`'s URL parsing functions will look for query parameter values in the hash as well as the main query string, so it is important to use them instead of directly accessing `window.location.href`.

`broadcast` provides the following functions:

- `isHashExists()`: Returns the hash of the URL if it exists, `false` if otherwise.
- `getHashFromUrl()`: Returns the hash of the URL. Can be an empty string.
- `getSearchFromUrl()`: Returns the query string.
- `extractKeyValuePairsFromQueryString()`: Converts a query string to an object mapping query parameter names with query parameter values.
- `getValuesFromUrl()`: Returns an object mapping query parameter names with query parameter values for the current URL.
- `getValueFromUrl()`: Returns one query parameter value for the current URL by name.
- `getValueFromHash()`: Returns one query parameter value in the hash of the current URL by name.
- `getHash()`: Returns the hash of the URL.

To learn more about an individual function, see the method documentation in the `plugins/CoreHome/javascripts/broadcast.js` file.

#### Loading new Piwik pages

To load a new page below the main Piwik menu, use the `propagateNewPage()` function with a URL to the controller method whose output should be displayed:

```javascript
(function (require) {

    var broadcast = require('broadcast');
    broadcast.propagateNewPage('index.php?module=MyPlugin&action=mySpecialPage', true);

})(require);
```

### ajaxHelper

This class is considered deprecated and only exists for historical reasons as some non-angular components are still using it. You should usually instead use our angularjs service [piwikApi](https://github.com/matomo-org/matomo/blob/4.x-dev/plugins/CoreHome/angularjs/common/services/piwik-api.js).

The `ajaxHelper` class should be used whenever you need to create an AJAX request. **Plugins should not use `$.ajax` directly.** `ajaxHelper` does some extra things that make it harder to write insecure code. It also keeps track of the current ongoing AJAX requests which is vital to the [UI tests](/guides/tests-ui).

To use the `ajaxHelper`, create an instance, configure it, and then call the `send()` method. To learn more, read the documentation in the source code (located in [`plugins/Morpheus/javascripts/ajaxHelper.js`](https://github.com/matomo-org/matomo/blob/master/plugins/Morpheus/javascripts/ajaxHelper.js)).

For example:

```javascript
(function (require, $) {

    var ajaxHelper = require('ajaxHelper');

    var ajax = new ajaxHelper();
    ajax.setUrl('index.php?module=Actions&action=getPageUrls&idSite=1&date=today&period=day');
    ajax.setCallback(function (response) {
        $('#myReportContainer').html(response);
    });
    ajax.setFormat('html'); // the expected response format
    ajax.setLoadingElement('#myReportContainerLoading');
    ajax.send();

})(require, jQuery);
```

## Learn more

* To learn **about creating new report visualizations** read our [Visualizing Report Data](/guides/visualizing-report-data) guide.
* To learn **more about the asset merging system** read this [blog post](https://matomo.org/blog/2010/07/making-piwik-ui-faster/) by the system's author.
* To learn **more about theming** read our [Theming](/guides/theming) guide.
* To learn **more about UI components and styles** read our [Views](/guides/views) guide.
