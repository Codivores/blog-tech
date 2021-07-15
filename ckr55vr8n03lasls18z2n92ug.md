## Set up Inertia, Vite, Vue3, TypeScript / Build our first Page

Now it is time to:

- Install our front-end tools
- Create needed configuration files
- Create a root template
- Build and display our first Page

## Dependencies installation

Laravel comes with a default `package.json` file in order to build the front-end assets with Laravel Mix and Webpack packages.
The aim of our project is to replace Laravel Mix with Vite, so let's clean our `package.json` to the minimal and remove all scripts and dependencies:

```
{
    "private": true
}
```

> You can also delete the `webpack.mix.js` file

We can now install all the dependencies via [yarn](https://yarnpkg.com), but you can of course use [npm](https://www.npmjs.com) if you prefer.

```
# Inertia Client Side (with Axios as a dependency) and its Vue 3 plugin
yarn add @inertiajs/inertia @inertiajs/inertia-vue3

# Vue 3
yarn add vue@next

# Tools for authoring Single File Components (SFCs) (with PostCSS as a dependency)
yarn add @vue/compiler-sfc --dev

# Vite and its Vue 3 SFC plugin
yarn add vite @vitejs/plugin-vue --dev

# TypeScript and its Vue plugin
yarn add typescript @vuedx/typescript-plugin-vue --dev
```


## Inertia configuration

We need a TypeScript file to boot our Inertia application, so let's create `app.ts` file in `/resources/js/` (you can rename the previous default `app.js` - or name it like you want and adapt the other files accordingly - and delete the `bootstrap.js` file).

You can refer to the [official documentation](https://inertiajs.com/client-side-setup) for more details.

For easy start, we will just create and mount a basic Vue / Inertia app:

```
import { createApp, h } from "vue"
import { App, plugin as inertiaPlugin } from "@inertiajs/inertia-vue3"
import "vite/dynamic-import-polyfill"

const el = document.getElementById("app")!

createApp({
  render: () =>
    h(App, {
      initialPage: JSON.parse(el.dataset.page!),
      resolveComponent: async (name: string) => {
        const page = (await import(`./Pages/${name}.vue`)).default;
        return page
      },
    }),
})
  .use(inertiaPlugin)
  .mount(el)
```

** What it does**

- Import Vue and Inertia packages (and its Vue3 plugin)
- Declare our root HTML element ID
- Create the application, looking for Pages Vue Components *(that replace standard Laravel Blade files)* in `/resources/js/Pages` when rendering from Laravel Controller
- Use the Inertia Vue3 plugin

## Vite configuration

Here is the touchy part as we are discovring Vite, so take it as a draft that should be improved later on.

So, let's create a `vite.config.ts` in our project root folder:


```
import { ConfigEnv, defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

export default defineConfig(({ command }: ConfigEnv) => {
  return {
    base: command === 'build' ? '/dist/' : '',
    publicDir: "disable",
    build: {
      manifest: true,
      outDir: "public/dist",
      rollupOptions: {
        input: {
          app: "resources/js/app.ts",
        },
      },
    },
    server: {
      strictPort: true,
      port: 3030,
      // https: true,
      hmr: {
        host: "localhost",
      },
    },
    plugins: [
      vue()
    ],
    optimizeDeps: {
      include: [
        "@inertiajs/inertia",
        "@inertiajs/inertia-vue3",
        "axios",
        "vue"
      ],
    },
  }
})
```

** What it does**

- Import Vite and its Vue 3 SFC plugin
- Define the configuration ([Official Documentation](https://vitejs.dev/config))
  - `base`: web base path, according to the Vite command that tells us in which environment (development / production) we are:
    - production (`command === 'build'`): look for `dist` folder (in our `/public` folder) - you can name it like you want and adapt the other places it is used
    - development: empty string as we use Vite server
  - `publicDir`: folder used to serve static assets, we won't use it for now
  - `build`: options when we build for production environment
    - `manifest`: generate a `manifest.json` that contains the information of the files we will need to include in our application with their hashed version
    - `outDir`: by default, Vite will empty the output folder on build, so we use a subfolder of the `/public` Laravel folder
    - `rollupOptions`: here we define the entry point of our application (it uses [Rollup.js](https://rollupjs.org/guide/en/#big-list-of-options) that offers many options)
  - `server`:
    - `strictPort`: set to `true` to exit if port is already being used
    - `port`: set the port you want for your Vite server, it will be used later
    - `hmr.host`: here we set the `host` for our Hot Module Replacement
  - `plugins`: use the Vite Vue plugin
  - `optimizeDeps.include`: force a linked package to be pre-bundled *(not sure if necessary for axios and vue packages)*

## TypeScript configuration

We create a `tsconfig.json` in our project root folder:

```
{
    "compilerOptions": {
        "target": "esnext",
        "module": "esnext",
        "moduleResolution": "node",
        "strict": true,
        "jsx": "preserve",
        "sourceMap": true,
        "resolveJsonModule": true,
        "esModuleInterop": true,
        "lib": [
            "esnext",
            "dom"
        ],
        "plugins": [
            {
                "name": "@vuedx/typescript-plugin-vue"
            }
        ]
    },
    "include": [
        "resources/js/**/*.ts",
        "resources/js/**/*.d.ts",
        "resources/js/**/*.vue"
    ]
}
```

** What it does**

- Use default compiler options given by Vue
- Use `@vuedx/typescript-plugin-vue` package to improve TypeScript features for `.vue` files
- Defines the include folders: all `.ts`, `.d.ts` and `.vue` files in our `resources/js/` folders

## Scripts definition

All the configuration is set, we now need to define our vite scripts in our `package.json`

```
    "scripts": {
        "dev": "vite",
        "build": "vite build"
    },
```

** What it does**

- `dev`: start Vite server
- `build`: build production bundles

You can name them like you want (if you rename `build`, you need to change the test on the command variable in the `vite.config.ts` file)

We can now start our development environment:

```
yarn dev 
```

![yarn dev](https://cdn.hashnode.com/res/hashnode/image/upload/v1626273103699/ysv5ZFNuK.png)

Or build for production:

```
yarn build
```

![yarn build](https://cdn.hashnode.com/res/hashnode/image/upload/v1626363239110/tyREEDVQh.png)

## Root View set up

To include Inertia and load our assets on the first page visit, we need a root view Blade file. 
So we create `app.blade.php` in `/resources/views/` folder (you can use and rename the default `welcome.blade.php`).

> `app.blade.php` is the default view used by Inertia, you can change it in the Inertia Middleware or call `Inertia::setRootView()` in your code.

```
<!DOCTYPE html>
<html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
    <head>
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width, initial-scale=1">

        @production
            @php
                $manifest = json_decode(file_get_contents(public_path('dist/manifest.json')), true);
            @endphp
            <script type="module" src="/dist/{{ $manifest['resources/js/app.ts']['file'] }}"></script>
            {{-- <link rel="stylesheet" href="/dist/{{ $manifest['resources/js/app.ts']['css'][0] }}"> --}}
        @else
            <script type="module" src="http://localhost:3030/@vite/client"></script>
            <script type="module" src="http://localhost:3030/resources/js/app.ts"></script>
        @endproduction
    </head>
    <body>
        @inertia
    </body>
</html>
```

** What it does**

- To have Vite working in both development and production environments and include the related assets in the `head`, we need some little hack *(to be improved)*:
  - Production: read the `manifest.json` file to get our `app.ts` build and versioned file location (same will apply for CSS that we will see later)
  - Development: include Vite client and call the Vite server to serve our application from the entry point `/resources/js/app.ts` (adapt here the Vite host of HMR and server port)
- Include the `@inertia` Blade directive in the `body`

## Time to play and display our first Page

All that remains to do is:

- Create a Laravel Controller with its route
- Create a Vue Page

### Laravel

We will create a simple Controller, its only task is to render the Index Page (that Inertia will resolve in our `/resources/js/app.ts` and that will look for `/resources/js/Pages/Index.vue` file).

**/app/Http/Controllers/IndexController.php**

```
<?php

namespace App\Http\Controllers;

use Inertia\Inertia;

class IndexController extends Controller
{
    public function index()
    {
        return Inertia::render('Index');
    }
}
```

**/routes/web.php**

```
<?php

use App\Http\Controllers\IndexController;
use Illuminate\Support\Facades\Route;

Route::get('/', [IndexController::class, 'index'])->name('index');
```

### Vue

We will start just displaying a welcome message and write something in the console.

**/resources/js/Pages/Index.vue**

```
<template>
  <div>
    <h1>Welcome</h1>
  </div>
</template>

<script lang="ts">
import { defineComponent } from "vue"

export default defineComponent({
  setup() {
    console.log("Page - Index")
  },
})
</script>
```

### RUUUUUUN


Start Vite

```
yarn dev
```

Go to the web root of your project in a browser (problably `http://localhost`)

and...

![TADAAAAAAAAAAA](https://cdn.hashnode.com/res/hashnode/image/upload/v1626364122365/mwQH5-p6m.png)


### And maybe should we validate production build

First we build our assets.

```
yarn build
```

To use this files, we need to do a little change to tell Laravel we want to use production assets. To do so, we change the environment in our `.env` from

```
APP_ENV=local
```

to

```
APP_ENV=production
```

Now refresh your browser and you can see that the production JavaScript files are loaded


![TADADAAAAAAAAA](https://cdn.hashnode.com/res/hashnode/image/upload/v1626376345166/jL5mUy_e8.png)

---

> We'll do our best to provide source code of the serie on [GitHub](https://github.com/Codivores/tutorial-laravel-twill-inertia-vue3-vite-tailwind) 

%[https://github.com/Codivores/tutorial-laravel-twill-inertia-vue3-vite-tailwind]
