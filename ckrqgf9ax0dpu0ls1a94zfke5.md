## Give some style with Tailwind

In this article, we will
- [Install and configure Tailwind](#heading-installation-and-configuration)
  - [Tailwind 2](#heading-tailwind-2-configuration)
  - [Tailwind 3](#heading-tailwind-3-configuration)
- [Integrate it in our application](#heading-integration-in-our-application)
- [Play with Inertia and a basic Tailwind component](#heading-time-to-play)

## Installation and configuration

For this steps, we will mainly follow the [official documentation](https://tailwindcss.com/docs/guides/vue-3-vite).

### Dependencies installation

```
# Tailwind and its dependencies: PostCSS, Autoprefixer
yarn add tailwindcss@latest postcss@latest autoprefixer@latest --dev
```

### PostCSS configuration

Let's create a `postcss.config.js` in our project root folder with default configuration.

**/postcss.config.js**
```
module.exports = {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
}
```

### Tailwind 2 Configuration

Let's create a `tailwind.config.js` in our project root folder with default configuration except for the `mode` and `purge` options.

**/tailwind.config.js**
```
module.exports = {
    mode: 'jit',
    purge: [
        './resources/**/*.blade.php',
        './resources/**/*.vue',
    ],
    darkMode: false,
    theme: {
      extend: {},
    },
    variants: {
      extend: {},
    },
    plugins: [],
  }

```

** What it does**

The `purge` option is where we set the paths of our application files which have HTML classes, so that PostCSS can tree-shake unused styles for production builds (also used if JIT mode is enabled for development builds - explanations in next paragraph):
- our Blade files (probably just our root view `app.blade.php`)
- our Vue Pages and Components

The `mode` set to `jit` for Just-in-Time which is a new compiler currently in preview (you can remove this option if you have troubles, we may have an issue where this mode breaks the Vite HMR for `.vue` files - ongoing investigations). It uses your `purge` option configuration in development environment to generate your CSS faster and identically to production build.

You can see the difference without JIT


![Tailwind without JIT](https://cdn.hashnode.com/res/hashnode/image/upload/v1626377561392/YFOwPK56S.png)

and with JIT activated


![Tailwind with JIT](https://cdn.hashnode.com/res/hashnode/image/upload/v1626377625064/1AWXjcTKi.png)

### Tailwind 3 Configuration

The JIT compiler is now enabled by default and the `purge` option has been replaced by `content`.

**/tailwind.config.js**
```
module.exports = {
  content: [
    './resources/**/*.blade.php',
    './resources/**/*.vue',
  ],
  theme: {
    extend: {},
  },
  plugins: [],
}
```

## Integration in our application

- Create a CSS file with Tailwind directives
- Import it in our application
- Use it in our root view

We can use the default `/resources/css/app.css` that comes with Laravel (or feel free to do like you want) and we add the standard Tailwind directives.

**/resources/css/app.css**
```
@tailwind base;
@tailwind components;
@tailwind utilities;

```

Then we import it in our TypeScript application file.

**/resources/js/app.ts**
```
import { createApp, h } from "vue"
import { App, plugin as inertiaPlugin } from "@inertiajs/inertia-vue3"
import "vite/dynamic-import-polyfill"

import '../css/app.css'

const el = document.getElementById("app")!
...
```

In our Root View, we left a commented line that we can now uncomment, to include our CSS for the production build.

**/resources/views/app.blade.php**
```
        @production
            @php
                $manifest = json_decode(file_get_contents(public_path('dist/manifest.json')), true);
            @endphp
            <script type="module" src="/dist/{{ $manifest['resources/js/app.ts']['file'] }}"></script>
            <link rel="stylesheet" href="/dist/{{ $manifest['resources/js/app.ts']['css'][0] }}">
        @else
            <script type="module" src="http://localhost:3030/@vite/client"></script>
            <script type="module" src="http://localhost:3030/resources/js/app.ts"></script>
        @endproduction
```



## Time to play

### Just to see if it works

To do so, we will use a simple component taken from [Tailwind UI](https://tailwindui.com), and we replace the template part of our Index Page with this:

**/resources/js/Pages/Index.vue**

```
<template>
  <div class="bg-indigo-700">
    <div class="max-w-2xl mx-auto text-center py-16 px-4 sm:py-20 sm:px-6 lg:px-8">
      <h2 class="text-3xl font-extrabold text-white sm:text-4xl">
        <span class="block">Boost your productivity.</span>
        <span class="block">Start using Workflow today.</span>
      </h2>
      <p class="mt-4 text-lg leading-6 text-indigo-200">Ac euismod vel sit maecenas id pellentesque eu sed consectetur. Malesuada adipiscing sagittis vel nulla nec.</p>
      <a href="#" class="mt-8 w-full inline-flex items-center justify-center px-5 py-3 border border-transparent text-base font-medium rounded-md text-indigo-600 bg-white hover:bg-indigo-50 sm:w-auto">
        Sign up for free
      </a>
    </div>
  </div>
</template>
``` 

If Vite is started you just have to save and have a look to your browser to see


![Woaw](https://cdn.hashnode.com/res/hashnode/image/upload/v1626379957289/zZPxiy24S.png)

### Why not add some logic to play around

At this time we have a static Vue application, maybe we could use some data from our Laravel Controller and play with CSS classes.

In our Laravel Controller, we will compute a `darkMode` random boolean that we will use in our Page.

> The Inertia render function takes a second parameter which is an associative array that will be passed as Vue props in our Page.

**/app/Http/Controllers/IndexController.php**

```
<?php

namespace App\Http\Controllers;

use Inertia\Inertia;

class IndexController extends Controller
{
    public function index()
    {
        return Inertia::render('Index', [
            'darkMode' => (bool)random_int(0, 1),
        ]);
    }
}
``` 

And now in our Page, we register our `darkMode` prop (as a Boolean) in our component props and use it in the first `div` of the template to alternatively set its class to `bg-gray-700` or `bg-indigo-700`.

**/resources/js/Pages/Index.vue**

```
<template>
  <div :class="darkMode ? 'bg-gray-700' : 'bg-indigo-700'">
    <div class="max-w-2xl mx-auto text-center py-16 px-4 sm:py-20 sm:px-6 lg:px-8">
      <h2 class="text-3xl font-extrabold text-white sm:text-4xl">
        <span class="block">Boost your productivity.</span>
        <span class="block">Start using Workflow today.</span>
      </h2>
      <p class="mt-4 text-lg leading-6 text-indigo-200">Ac euismod vel sit maecenas id pellentesque eu sed consectetur. Malesuada adipiscing sagittis vel nulla nec.</p>
      <a href="#" class="mt-8 w-full inline-flex items-center justify-center px-5 py-3 border border-transparent text-base font-medium rounded-md text-indigo-600 bg-white hover:bg-indigo-50 sm:w-auto">
        Sign up for free
      </a>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent } from "vue"

export default defineComponent({
  props: {
    darkMode: Boolean,
  },
  setup() {
    console.log("Page - Index")
  },
})
</script>
``` 

If we go back to our browser, we can now either see this


![indigo mode](https://cdn.hashnode.com/res/hashnode/image/upload/v1626382693663/MdgeUzxNf.png)

or this


![gray mode](https://cdn.hashnode.com/res/hashnode/image/upload/v1626382667300/EI195jDxH.png)

---

**All is now set to build the front-end of our application, in the next article we will prepare the back-end**

---

> We'll do our best to provide source code of the serie on [GitHub](https://github.com/Codivores/tutorial-laravel-twill-inertia-vue3-vite-tailwind) 

%[https://github.com/Codivores/tutorial-laravel-twill-inertia-vue3-vite-tailwind]