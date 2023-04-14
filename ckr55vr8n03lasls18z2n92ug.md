---
title: "Set up Inertia, Vite, Vue3, TypeScript / Build our first Page"
seoTitle: "Set up Inertia, Vite, Vue3, TypeScript on our Laravel application"
seoDescription: "Install the front-end tools: Inertia client side, Vite, Vue 3 with Composition API and TypeScript"
datePublished: Thu Jul 15 2021 17:03:35 GMT+0000 (Coordinated Universal Time)
cuid: ckr55vr8n03lasls18z2n92ug
slug: ltivt-2-inertia-vite-vue3-typescript
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1627665008142/oVypqiw66.png
tags: laravel, vuejs, typescript, frontend-development, inertiajs

---

> **Updated version for Laravel 10 / Twill 3 on Apr 14, 2023**

Now it is time to:

* [Install our front-end tools](#dependencies-installation)
    
* [Create needed configuration files](#inertia-configuration)
    
* [Create a root template](#root-view-set-up)
    
* [Build and display our first Page](#time-to-play-and-display-our-first-page)
    

## Dependencies installation

Laravel comes with a default `package.json` file to build the front-end assets and already integrates Vite and the Laravel Vite packages.

We can now install all the needed dependencies via [yarn](https://yarnpkg.com), but you can of course use [npm](https://www.npmjs.com) if you prefer.

```bash
# Vue 3
yarn add vue

# Vite plugin Vue
yarn add vite @vitejs/plugin-vue --dev

# TypeScript
yarn add typescript --dev

# Inertia Vue 3 adapter (with @inertiajs/core and axios as a dependency)
yarn add @inertiajs/vue3
```

## Inertia configuration

We need a TypeScript file to boot our Inertia application, so let's create `app.ts` file in `/resources/js/` (you can rename the previous default `app.js` - or name it like you want and adapt the other files accordingly - and delete the `bootstrap.js` file).

You can refer to the [official documentation](https://inertiajs.com/client-side-setup) for more details.

For an easy start, we will just create and mount a basic Vue / Inertia app:

```javascript
import { createApp, h, type DefineComponent } from 'vue'
import { createInertiaApp } from '@inertiajs/vue3'
import { resolvePageComponent } from 'laravel-vite-plugin/inertia-helpers'

createInertiaApp({
  resolve: async (name: string) => {
    let page = await resolvePageComponent(`../views/Pages/${name}.vue`, import.meta.glob<DefineComponent>('../views/Pages/**/*.vue'))

    return page
  },
  setup({ el, App, props, plugin }) {
    createApp({ render: () => h(App, props) })
      .use(plugin)
      .mount(el)
  },
})
```

\*\* What it does\*\*

* Import Vue and Inertia packages
    
* Create the application, looking for Pages Vue Components *(that replace standard Laravel Blade files)* in `/views/Pages` when rendering from Laravel Controller
    

## Vite configuration

Laravel comes with a default `vite.config.ts` file in our project root folder with laravel plugin configured that we are going to edit:

```javascript
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import vue from '@vitejs/plugin-vue';

export default defineConfig({
    plugins: [
        laravel({
            input: ['resources/css/app.css', 'resources/js/app.ts'],
            refresh: true,
        }),
        vue({
            template: {
                transformAssetUrls: {
                    // The Vue plugin will re-write asset URLs, when referenced
                    // in Single File Components, to point to the Laravel web
                    // server. Setting this to `null` allows the Laravel plugin
                    // to instead re-write asset URLs to point to the Vite
                    // server instead.
                    base: null,
 
                    // The Vue plugin will parse absolute URLs and treat them
                    // as absolute paths to files on disk. Setting this to
                    // `false` will leave absolute URLs un-touched so they can
                    // reference assets in the public directory as expected.
                    includeAbsolute: false,
                },
            },
        }),
    ],
});
```

\*\* What it does\*\*

* Import Vite and its Vue 3 SFC plugin
    
* Define the configuration ([Official Documentation](https://vitejs.dev/config))
    
    * `plugins`:
        
        * `laravel`: adapt the location of the application entry point
            
        * `vue`: use the Vite Vue plugin
            

## TypeScript configuration

We create a `tsconfig.json` in our project root folder:

```json
{
  "compilerOptions": {
    "module": "ESNext",
    "moduleResolution": "Node",
    "resolveJsonModule": true,
    "useDefineForClassFields": true,

    // Required in Vue projects
    "jsx": "preserve",

    // `"noImplicitThis": true` is part of `strict`
    // Added again here in case some users decide to disable `strict`.
    // This enables stricter inference for data properties on `this`.
    "noImplicitThis": true,
    "strict": true,

    // Required in Vite
    "isolatedModules": true,
    // For `<script setup>`
    // See <https://devblogs.microsoft.com/typescript/announcing-typescript-4-5-beta/#preserve-value-imports>
    "preserveValueImports": true,
    // Enforce using `import type` instead of `import` for types
    "importsNotUsedAsValues": "error",

    // A few notes:
    // - Vue 3 supports ES2016+
    // - For Vite, the actual compilation target is determined by the
    //   `build.target` option in the Vite config.
    //   So don't change the `target` field here. It has to be
    //   at least `ES2020` for dynamic `import()`s and `import.meta` to work correctly.
    // - If you are not using Vite, feel free to override the `target` field.
    "target": "ESNext",

    // Recommended
    "esModuleInterop": true,
    "forceConsistentCasingInFileNames": true,
    // See <https://github.com/vuejs/vue-cli/pull/5688>
    "skipLibCheck": true,

    "types": ["vite/client", "node"],
  
    "baseUrl": ".",
    "paths": {
      "@/*": ["resources/*"]
    },
    "include": [
      "resources/js/**/*.ts",
      "resources/js/**/*.d.ts",
      "resources/**/*.vue"
    ]
  }
}
```

\*\* What it does\*\*

* Use default compiler options given by Vue
    
* Defines the include folders: all `.ts`, `.d.ts` and `.vue` files in our `resources` folders
    

## Scripts definition

All the configuration is set, let's have a look at the Vite scripts in our `package.json`

```json
    "scripts": {
        "dev": "vite",
        "build": "vite build"
    },
```

\*\* What it does\*\*

* `dev`: start Vite server
    
* `build`: build production bundles
    

Of course, you can name them like you want and adapt the scripts according to your build process.

We can now start our development environment:

```bash
yarn dev
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1681468373313/51926191-6ed1-40e2-9a7d-0a7ed7bc632f.png align="left")

Or build for production:

```bash
yarn build
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1681468483638/04f0b161-b704-4dcb-bec6-fc1aaf4ba14a.png align="left")

## Root View set up

To integrate Inertia and load our assets on the first page visit, we need a standard view Blade file. So we create `app.blade.php` in `/resources/views/` folder (you can use and rename the default `welcome.blade.php`).

> `app.blade.php` is the default view used by Inertia, you can change it in the Inertia Middleware or call `Inertia::setRootView()` in your code.

```xml
<!DOCTYPE html>
<html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
    <head>
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width, initial-scale=1">
        @vite('resources/js/app.ts')
        @inertiaHead
    </head>
    <body>
        @inertia
    </body>
</html>
```

\*\* What it does\*\*

* Add the `@vite()` Blade directive in the `head` to reference our Vite entry point
    
* Add the `@inertiaHead` Blade directive in the `head`
    
* Add the `@inertia()` Blade directive in the `body`
    

## Time to play and display our first Page

All that remains to do is:

* Create a Laravel Controller with its route
    
* Create a Vue Page
    

### Laravel

We will create a simple Controller, its only task is to render the Index Page (that Inertia will resolve in our `/resources/js/app.ts` and that will look for `/resources/views/Pages/Index.vue` file).

**/app/Http/Controllers/IndexController.php**

```php
<?php

namespace App\Http\Controllers;

use Inertia\Inertia;
use Inertia\Response as InertiaResponse;

class IndexController extends Controller
{
    public function __invoke(): InertiaResponse
    {
        return Inertia::render('Index');
    }
}
```

**/routes/web.php**

```php
<?php

use App\Http\Controllers\IndexController;
use Illuminate\Support\Facades\Route;

Route::get('/', IndexController::class)->name('index');
```

### Vue

We will start displaying a welcome message and write something in the console.

**/resources/views/Pages/Index.vue**

```xml
<template>
  <div>
    <h1>Welcome</h1>
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue'

export default defineComponent({
  setup() {
    console.log('Page - Index')
  },
})
</script>
```

Or with the SFC `<script setup>` (compile-time syntactic sugar for using Composition API inside Single File Components: [https://vuejs.org/api/sfc-script-setup.html](https://vuejs.org/api/sfc-script-setup.html))

```xml
<script setup lang="ts">
  console.log('Page - Index')
</script>

<template>
  <h1>Welcome</h1>
</template>
```

### RUUUUUUN

Start Vite

```bash
yarn dev
```

Go to the web root of your project in a browser (probably `http://localhost`)

and...

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1681474424942/ae4ba4a4-fd14-43f8-beee-c42fa10ce617.png align="center")

### And maybe should we validate production build

First we build our assets.

```bash
yarn build
```

Now refresh your browser and you can see that the production JavaScript files are loaded

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1681474474730/7a85c63d-9425-4d70-9945-4d2a03e06c3d.png align="center")

---

> We'll do our best to provide source code of the serie on [GitHub](https://github.com/Codivores/tutorial-laravel-twill-inertia-vue3-vite-tailwind)

%[https://github.com/Codivores/tutorial-laravel-twill-inertia-vue3-vite-tailwind]