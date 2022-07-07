# Upgrade Guide

## Migrating from ColdBox Elixir to Vite

> **Note** This upgrade guide does not cover all possible ColdBox Elixir use cases, such as Sass compilation. Please consult the [Vite documentation](https://vitejs.dev/guide/) for information on configuring Vite for these scenarios.

### Install the ColdBox Vite module

TODO: write this module and docs

### Install Vite and the ColdBox Vite Plugin

Next, you will need to install [Vite](https://vitejs.dev/) and the [ColdBox Vite Plugin](https://www.npmjs.com/package/coldbox-vite-plugin) using your npm package manager of choice:

```shell
npm install --save-dev vite coldbox-vite-plugin
```

You may also need to install additional Vite plugins for your project, such as the Vue or React plugins:

```shell
npm install --save-dev @vitejs/plugin-vue
```

```shell
npm install --save-dev @vitejs/plugin-react
```

### Configure Vite

Create a `vite.config.js` file in the root of your project:

```js
import { defineConfig } from "vite";
import coldbox from "coldbox-vite-plugin";
// import react from "@vitejs/plugin-react";
// import vue from "@vitejs/plugin-vue";

export default defineConfig({
    plugins: [
        coldbox(["resources/assets/css/app.css", "resources/assets/js/app.js"]),
        // react(),
        // vue({
        //     template: {
        //         transformAssetUrls: {
        //             base: null,
        //             includeAbsolute: false,
        //         },
        //     },
        // }),
    ],
});
```

If you are building an SPA, you will get a better developer experience by removing the CSS entry point above and [importing your CSS from Javascript](#importing-your-css-from-your-javascript-entry-points).

#### Update Aliases

If you are migrating aliases from your ColdBox Elixir's `webpack.config.js` file to your `vite.config.js` file, you should ensure that the paths start with `/`. For example, `resources/assets/js` would become `/resources/assets/js`:

```js
export default defineConfig({
    plugins: [
        coldbox(["resources/assets/css/app.css", "resources/assets/js/app.js"]),
    ],
    resolve: {
        alias: {
            "@": "/resources/assets/js",
        },
    },
});
```

For your convenience, the ColdBox Vite plugin automatically adds an `@` alias for your `/resources/assets/js` directory. If you do not need to customize your aliases, you may omit this section from your `vite.config.js` file.

### Update NPM scripts

Update your NPM scripts in `package.json`:

TODO: update to match ColdBox Elixir's scripts

```diff
  "scripts": {
-     "dev": "webpack --hide-modules",
-     "watch": "npm run dev -- --watch",
-     "prod": "npm run dev -- -p"
+     "dev": "vite",
+     "build": "vite build"
  }
```

### Vite compatible imports

Vite only supports ES modules, so if you are upgrading an existing application you will need to replace any `require()` statements with `import`.

#### Inertia

Inertia makes use of a `require()` call that is more complex to replicate with Vite.

The following function can be used instead:

```diff
+ import { resolvePageComponent } from "coldbox-vite-plugin/inertia-helpers";

  createInertiaApp({
      title: (title) => `${title} - ${appName}`,
-     resolve: (name) => require(`./Pages/${name}.vue`),
+     resolve: (name) => resolvePageComponent(`./Pages/${name}.vue`, import.meta.glob('./Pages/**/*.vue')),
      setup({ el, app, props, plugin }) {
          return createApp({ render: () => h(app, props) })
              .use(plugin)
              .mixin({ methods: { route } })
              .mount(el);
      },
  });
```

### Importing your CSS from your JavaScript entry point(s)

If you are building an SPA, you will get a better experience by importing your CSS from your JavaScript entry point(s), such as your `resources/js/app.js` entry point:

```diff
  import './bootstrap';
+ import '../css/app.css';
```

In development mode, Vite will automatically inject your CSS into the page. In production, a dedicated stylesheet will be generated that the `@vite` directive will load from the manifest.

### Replace `elixir()` and `elixirPath()` with `vite()` from the `coldbox-vite` module

When using Vite, you will need to use the `vite()` function provided by the `coldbox-vite` module instead of the `elixir()` or `elixirPath()` helper.

This will automatically detect whether you are running in serve or build mode and include all of the required `<script>` and `<link rel="stylesheet">` for you:

```diff
- <link rel="stylesheet" href="#elixirPath( "css/app.css" )#">
- <script src="#elixirPath( "js/app.js" )#" defer></script>
+ #vite([ "resources/assets/css/app.css", "resources/assets/js/app.js" ] )#
```

The entry points should match those used in your `vite.config.js`.

#### React

If you are using React and hot-module replacement, you will need to include an additional directive _before_ the `@vite` directive:

TODO: Are we supporting this react thing?

```html
@viteReactRefresh #vite( "resources/assets/js/app.jsx" )#
```

This loads a React "refresh runtime" in development mode only, which is required for hot module replacement to work correctly.

### JavaScript files containing JSX must use a `.jsx` extension

You will need to rename any `.js` files containing JSX to instead have a `.jsx` extension.

See [this tweet](https://twitter.com/youyuxi/status/1362050255009816577) from Vite's creator for more information.

> **Note** If you are using Tailwind, remember to update the paths in your `tailwind.config.js` file.

### Vue imports must include the `.vue` extension

```diff
- import Button from "./Button";
+ import Button from "./Button.vue";
```

### Remove ColdBox Elixir

The ColdBox Elixir package can now be uninstalled:

```shell
npm remove coldbox-elixir
```

And you may remove your Webpack configuration file:

```shell
rm webpack.config.js
```

### Optional: Configure Tailwind

If you are using Tailwind you will need to create a `postcss.config.js` file. Tailwind can generate this for you automatically:

```shell
npx tailwindcss init -p
```

Or, you can create it manually:

```js
module.exports = {
    plugins: {
        tailwindcss: {},
        autoprefixer: {},
    },
};
```

If you are using other PostCSS plugins, such as `postcss-import`, you will need to include them in your configuration.

### Optional: Git ignore the build directory

Vite will place all of your build assets into a `build` subdirectory inside your `includes` directory. If you prefer to build your assets on deploy instead of committing them to your repository, you may wish to add this directory to your `.gitignore` file:

```gitignore
/includes/build
```

### Optional: Update SSR configuration

// TODO: do we support this?

You may remove your dedicated Laravel Mix SSR configuration:

```shell
rm webpack.ssr.mix.js
```

In most cases, you won't need a dedicated SSR configuration file when using Vite. You can specify your SSR entry point by passing a configuration option to the Laravel plugin:

```js
import { defineConfig } from "vite";
import laravel from "laravel-vite-plugin";

export default defineConfig({
    plugins: [
        laravel({
            input: "resources/js/app.js",
            ssr: "resources/js/ssr.js",
        }),
    ],
});
```

You may wish to add the following additional scripts to your `package.json`:

```diff
  "scripts": {
      "dev": "vite",
-     "build": "vite build"
+     "build": "vite build && vite build --ssr"
  }
```

If you prefer to build your assets on deploy instead of committing them to your repository, you may wish to add the SSR output directory to your `.gitignore` file:

```gitignore
/storage/ssr
```

You may start the SSR server using `node`:

```sh
node storage/ssr/ssr.js
```

### Wrapping up

You should now be able to build your assets using `dev` command. This will also invoke the Vite server and Vite will watch for file changes:

```shell
npm run dev
```

Alternatively, if you need to build files without watching or if you need to build them for production, you may use the `build` command:

```shell
npm run build
```
