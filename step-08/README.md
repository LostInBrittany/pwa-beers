# Step 08 - Putting the *P* into PWA

In this step we are adding the *progressive enhancement* to our webapp, to make it a PWA like the one at step-00.

We begin by configuring the application entrypoint, `index.html`

## Configuring the entrypoint

In order to make our app *progressive*, we need service workers to manage the offline mode. The Polymer CLI will generate a script `service-worker.js` when we build the app for production at the end of this step, but we need to register it to be used by the browser. We can start by opening the `index.html` file and adding the following script to the head element:

```html
<script>
  if ('serviceWorker' in navigator) {
    window.addEventListener('load', function() {
      navigator.serviceWorker.register('/service-worker.js');
    });
  }
</script>
```

## Making all your imports relative

Polymer CLI doesn't like absolute paths in imports, and in some configurations it blocks the build.

Go to each one of your components and make the links relative, using `app` as base directory.

Example, for `src/beer-list/beer-list.html`:
```html
<link rel="import" href="../../bower_components/iron-ajax/iron-ajax.html">
<link rel="import" href="../../bower_components/iron-flex-layout/iron-flex-layout-classes.html">

<link rel="import" href="../../bower_components/paper-dropdown-menu/paper-dropdown-menu.html">
<link rel="import" href="../../bower_components/paper-input/paper-input.html">
<link rel="import" href="../../bower_components/paper-item/paper-item.html">
<link rel="import" href="../../bower_components/paper-listbox/paper-listbox.html">
<link rel="import" href="../../bower_components/paper-material/paper-material.html">
```

## Making sure that everything is served by your server

If we want to be able to use Service Worker to locally cache all the resources of your app, you need to serve all the resources from your webserver.

Right now in our app there is only a non-local resource, as you can see if you use browser's developer tools: the Roboto font files, that are served from Google's CDN. In order to make it local, we are going to use a variation of the `font-roboto` element, `font-roboto-local`.  

Let's add it to `bower.json` (and don't forget the `bower install` to get the package):

```json
{
  "name": "pwa-beers",
  "description": "A tutorial on PWA with Polymer 1.x",
  "main": "index.html",
  "dependencies": {
    "polymer": "Polymer/polymer#^1.4.0",
    "app-route": "PolymerElements/app-route#^0.9.3",
    "iron-ajax": "PolymerElements/iron-ajax#^1.4.3",
    "iron-pages": "PolymerElements/iron-pages#^1.0.8",
    "iron-selector": "PolymerElements/iron-selector#^1.5.2",
    "paper-toolbar": "PolymerElements/paper-toolbar#^1.1.7",
    "paper-material": "PolymerElements/paper-material#^1.0.6",
    "paper-input": "PolymerElements/paper-input#^1.1.23",
    "paper-dropdown-menu": "PolymerElements/paper-dropdown-menu#^1.5.0",
    "paper-listbox": "PolymerElements/paper-listbox#^1.1.2",
    "paper-item": "PolymerElements/paper-item#^1.2.1",
    "font-roboto": "PolymerElements/font-roboto-local#^1.0.1"
  },
  "devDependencies": {
    "iron-component-page": "PolymerElements/iron-component-page#^1.0.0",
    "iron-demo-helpers": "PolymerElements/iron-demo-helpers#^1.0.0",
    "web-component-tester": "^4.0.0",
    "webcomponentsjs": "webcomponents/webcomponentsjs#^0.7.0"
  }
}
```

We are calling it `font-roboto` in order to make it a transparent replacement of the precedent package.


## Configure the Web App Manifest

The manifest file `manifest.json` is needed to tell the browser that the app can be *installed* like a PWA.

```json
{
  "name": "PWA Beers",
  "short_name": "pwa-beers",
  "description": "A tutorial on PWA with Polymer 1.x",
  "start_url": "./",
  "display": "standalone",  
  "theme_color": "#9C27B0",
  "background_color": "#9C27B0",
}
```

Let's look at some of its properties:

- `name`: The name of your app. Pick any name you want! I called my app "PWA Beers", of course!

- `short_name`: A short name for the app

- `start_url`: The URL that should be opened when your app starts. Since we only have one entry point, this should just be `index.html`

- `display`: How the app should be presented to the user when launched from the home screen. Using the value standalone will make your Web App look like a native app (without any browser chrome) when launched

- `theme_color`: The primary theme color of your app. This will be used to adjust the highlight color of native OS chrome (where applicable) when your app launches

- `background_color`: The color that will be used for the splash screen as the app is launching from a mobile device's home screen

### Icon configuration

Every great mobile app needs its signature launcher icon. Your Web App Manifest will let you configure icons just like a native app: you can specify different resolutions and styles of icon for different use cases and screen densities.

```json
"icons": [{
   "src": "/img/logo-25px.png",
   "sizes": "25x25",
   "type": "image/png"
 },
 {
    "src": "/img/logo-40px.png",
    "sizes": "40x40",
    "type": "image/png"
 },
 {
    "src": "/img/logo-250px.png",
    "sizes": "250x250",
    "type": "image/png"
 },
 {
    "src": "/img/logo-500px.png",
    "sizes": "500x500",
    "type": "image/png"
 }]
}
```

### Testing the `manifest.json`

If now you reload the app, you can use Chrome Developer Tools (*Application* tab) to see if the browser can understand it:

[![Screenshot](/img/step-08_01.t.jpg)](/img/step-08_01.jpg)


## Adding a `polymer.json`

This is a file that lives at the top-level of the project and contains the build configuration. We need to add the configuration to the `polymer.json` file to indicate the main entrypoint, the app shell and a few more:

```json
{
  "entrypoint": "index.html",
  "shell": "src/pwa-app/pwa-app.html",
  "fragments": [
    "src/beer-list/beer-list.html",
    "src/beer-details/beer-details.html"
  ],
  "sourceGlobs": [
    "data/**/*",
    "img/**/*",
    "bower_components/font-roboto/fonts/**",
    "bower.json"
  ],
  "includeDependencies": [
    "manifest.json",
    "bower_components/webcomponentsjs/webcomponents-lite.js"
  ]
}
```

Let's look at it:

- `entrypoint`: The main entrypoint into your application for all routes. Often times this is your index.html file. This file should import the app shell file specified in the shell option. It should be minimal since it's loaded and cached for each route.

- `shell`: The app shell file containing common code for the app.

- `fragment`: An array of any HTML files that are not synchronously loaded from the app shell, such as async imports or any imports loaded on-demand (e.g. by importHref).

- `sources`: An optional array of globs matching your application source files. This will default to all files in your project src/ directory, but configuring your own list of sources can be useful when your source files live in other directories.

- `includeDependencies`: An optional array of globs matching any additional dependencies you'd like to include with your build. If your application loads any files dynamically they can be missed by the analyzer, but you can include them here to make sure that they are always added to your build.

> Note: If you ever use polymer-build to define your own build process you can decide to handle sources & dependencies differently. But Polymer CLI currently treats additional files included in sources & includeDependencies the same, so place any additional files wherever you think makes the most sense.

## Adding a `sw-precache-config.js`

`sw-precache-config.js` is a file that stores the configuration for the service worker. Configuration properties such as static files, navigation fallbacks and the network strategies are just a few of the options that are supported.

```js
module.exports = {
  staticFileGlobs: [
    '/index.html',
    '/manifest.json',
    '/bower_components/webcomponentsjs/webcomponents-lite.js'
  ],
  navigateFallback: 'index.html',
  navigateFallbackWhitelist: [/^(?!.*\.html$|\/data\/).*/],
  runtimeCaching: [
    {
      urlPattern: /\/img\/.*/,
      handler: 'cacheFirst',
      options: {
        cache: {
          maxEntries: 200,
          name: 'items-cache'
        }
      }
    },
    {
      urlPattern: /\/data\/.*json/,
      handler: 'fastest',
      options: {
        cache: {
          maxEntries: 100,
          name: 'data-cache'
        }
      }
    }
  ]
};
```

In this file the configuration goes as follows:

- `staticFileGlobs`: The files `/index.html`, `/manifest.json` and `/bower_components/webcomponentsjs/webcomponents-lite.js` should be automatically precached by the service worker.

- `navigateFallback` and `navigateFallbackWhitelist`: If a user lands in a URL not found in the cache that doesn't match any of the patterns in `navigateFallbackWhitelist` then `index.html` is served. This option allows us to serve the app shell while the app is offline for any client-side route.

- `runtimeCaching`: The runtime caching configuration for dynamic content. Every entry allows to specify a caching strategy. [Check this link](https://googlechrome.github.io/sw-toolbox/docs/master/tutorial-api.html) to learn more about this API.`

## Lazily loading fragments

In our main element, `pwa-app`, we are currently front-loading the two fragments, `beer-list` and `beer-details`. If we want to respect the App Shell pattern, those fragments should be loaded asynchronously. Let's do it!

We begin by deleting the imports for `beer-list.html` and `beer-details.html`.

Delete these lines:
```html
<link rel="import" href="../beer-list/beer-list.html">
<link rel="import" href="../beer-details/beer-details.html">
```  

We add an observer to the `page` property, and we use `Polymer.importHref` to import the fragments:

```js
properties: {
  page: {
    type: String,
    reflectToAttribute: true,
    observer: '_pageChanged'
  },
},
_pageChanged: function(page) {
  console.debug("[pwa-beers] _pageChanged", page);
  // load page import on demand.
  this.importHref(
    this.resolveUrl('../beer-' + page + '/beer-' + page + '.html'), null, null, true);
},
```

## Using the CLI

As we will serve the application from the 'step-08' folder, you will have to create some symlinks.

```
cd step-08
ln -sfn ../bower_components/ .
ln -sfn ../data/ .
ln -sfn ../img/ .
```

Now, we are ready to build the app by running the following command:

```bash
polymer build
```

If it was built successfully, stop any previously run polymer serve and serve the bundled build:

```bash
polymer serve build/bundled
```

Now, if you visit http://localhost:8080, you will be looking at the built version. This means a few things:

A service worker has been automatically configured, so the app can work offline. To test the service worker you can open devtools and mark the offline checkbox in the network panel. You will see that the app is still available even in offline mode.

Changes to any of the source files would require clearing the service worker and a creating a new build (running polymer build again) in order for those changes to be reflected.
