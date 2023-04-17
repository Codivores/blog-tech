---
title: "Twill, Inertia and Vue improvements"
seoTitle: "Laravel Twill Inertia - First module improvements"
seoDescription: "Improve logic, performance, DX of the Laravel Twill Inertia Vue 3 application"
datePublished: Fri Apr 14 2023 17:02:33 GMT+0000 (Coordinated Universal Time)
cuid: clgjnoakq000009mf0a4nhu0v
slug: ltivt-6-twill-inertia-improvements
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1681638786326/1113a320-d4cf-4b81-85d0-4899bff7e0f4.avif
tags: laravel, vuejs, inertiajs, twill

---

We will use our previous PageContent module to make some improvements.

## Props optimization

When Inertia renders a Page, all the props are JSON encoded in a `data-page` attribute of the root `div`.

So, when we add the `$item` to Inertia in our App Controller,

```php
return Inertia::render('Page/Content', [
    'item' => $item,
]);
```

all the attributes of the Model are serialized, as you can see in the rendered HTML:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1681639480469/9ca99889-2bf2-4c81-85c5-4d61c1a2fd49.png align="center")

If you use **Vue.js devtools**, you can inspect the props:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1681639593954/cdadfa12-8c77-41bb-b016-67c0885b7a93.png align="center")

As you can see, there are a lot of data we don't need, which slows the page load and we don't want to be visible to our visitors.

To improve this, we will use the `only()` method of the Eloquent Model to return a subset of attributes. You can directly use this in the App Controller:

**/app/Http/Controllers/App/PageContentController.php**

```php
<?php

class PageContentController extends Controller
{
    public function show(string $slug): InertiaResponse
    {
        ...

        return Inertia::render('Page/Content', [
            'item' => $item->only([
                'title',
                'meta_title',
                'meta_description',
            ]),
        ]);
    }
}
```

We prefer to create a `$publicAttributes` array attribute in our Model (like existing `$translatedAttributes`, `$slugAttributes`, ...) to declare these attributes:

**/app/Models/PageContent.php**

```php
<?php

class PageContent extends Model implements Sortable
{
    ...

    public array $publicAttributes = [
        'title',
        'meta_title',
        'meta_description',
    ];
}
```

And use it in the App Controller:

**/app/Http/Controllers/App/PageContentController.php**

```php
<?php

class PageContentController extends Controller
{
    public function show(string $slug): InertiaResponse
    {
        ...

        return Inertia::render('Page/Content', [
            'item' => $item->only($item->publicAttributes),
        ]);
    }
}
```

If we inspect the props, we see there are just the needed attributes:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1681652310124/790943d6-5573-458d-a3bd-62180fa0aee7.png align="center")

## Cache

Our content will not change often and loading all the needed data implies multiple queries that necessarily takes some time:

* Search for a published Model with the given slug
    
* Load Model relations: translations, medias, files, blocks
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1681653366487/eaebf3e6-adc5-428c-8d70-d72216b49309.png align="center")

and we will see in a future article, we will rework some data (mainly for the blocks).

In our projects, we generally try to improve performance by caching database retrieved and reworked data. In the Laravel / Twill context, it can be done like that:

**/app/Http/Controllers/App/PageContentController.php**

```php
<?php

namespace App\Http\Controllers\App;

use App\Models\PageContent;
use Illuminate\Http\Response;
use Illuminate\Support\Facades\Cache;
use Inertia\Inertia;
use Inertia\Response as InertiaResponse;

class PageContentController extends Controller
{
    public function show(string $slug): InertiaResponse
    {
        $item = Cache::rememberForever(
            'page.content.' . app()->getLocale() . '.' . $slug,
            function () use ($slug) {
                $item = PageContent::publishedInListings()
                    ->forSlug($slug)
                    ->first();
                if ($item !== null) {
                    $item->load('translations', 'medias', 'blocks');
                }
                return $item;
            }
        );

        ...
    }
}
```

To clear the cache when the content is modified, it is handled in the Repository where Twill provides an `afterSave()` method we can override:

**/app/Repositories/PageContentRepository.php**

```php
<?php

namespace App\Repositories;

...
use Illuminate\Support\Facades\Cache;

class PageContentRepository extends ModuleRepository
{
    ...

    public function afterSave($object, $fields): void
    {
        // Cache clearing
        foreach (optional($object)->slugs as $slug) {
            Cache::forget('page.content.' . $slug->locale . '.' . $slug->slug);
        }

        parent::afterSave($object, $fields);
    }
}
```

## Vue props type alias

As we use TypeScript, we can benefit from static type checking and many IDE support this language providing auto-complete and warnings. It really improves your DX and we will see how.

### Defining types

We create a `models.d.ts` file in `/resources/js/types` to define the Eloquent-based objects used in Inertia pages in a `Model` namespace. For our module, we will create a `Page` type:

**/resources/js/types/models.d.ts**

```typescript
declare namespace Model {
  export type Page = {
    title: string
    meta_title?: string
    meta_description?: string
  }
}
```

### Using types

In the previous article, we were using `object` for our `item` type, in our IDE (VS Code), that triggers TypeScript warnings:

**/resources/views/Pages/Page/Content.vue**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1681663295680/58e12921-91c3-40b4-a05f-7ee4a0a27473.png align="left")

Now we can change the type of our `item` prop from `object` to `Model.Page`:

```javascript
<script setup lang="ts">
interface Props {
  item: Model.Page
}

defineProps<Props>()
</script>

<template>
  <div class="bg-gray-200 w-full h-screen flex flex-col justify-center items-center">
    <h1 class="text-center text-4xl font-semibold text-gray-900">
      {{ item.title }}
    </h1>
  </div>
</template>
```

No more warnings, and we can have information in our IDE about the properties of the object and their types:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1681663503601/4f7a0c33-5d89-474e-b25c-64009742c891.png align="left")

## Vite / TypeScript alias

As our application grows, we will have more and more components, split into different directories to keep a clean architecture (and maybe at some point we will create a new version of the theme, components, ...).

To simplify how we import components into other components and avoid relative paths like `../../Theme/Head.vue`, we can benefit from `alias` feature to map an alias to a path and make it available in code and IDE.

We generally define 3 default aliases on a new project:

* `@Composable`: stores our Vue Composition API composables
    
* `@Form`: stores our form components (input, select, switch, ...)
    
* `@Theme`: stores our theme components (head, blocks, ...)
    

Feel free to create yours according to your project structure.

### Configuration

The aliases have to be known by Vite (Rollup) and the TypeScript compiler.

**/vite.config.js**

```typescript
import { defineConfig } from 'vite';

export default defineConfig({
    resolve: {
        alias: [
          {
            find: '@Composable',
            replacement: '/resources/js/Composables'
          },
          {
            find: "@Form",
            replacement: `/resources/views/Components/Form`,
          },
          {
            find: '@Theme',
            replacement: '/resources/views/Components/Theme'
          }
        ]
    },

    ...
});
```

**/tsconfig.json**

```typescript
{
  "compilerOptions": {
    ...

    "paths": {
      "@/*": ["resources/*"],
      "@Composable/*": ["resources/js/Composables/*"],
      "@Form/*": ["resources/views/Components/Form/*"],
      "@Theme/*": ["resources/views/Components/Theme/*"]
    },

    ...
  }
}
```

### Usage

In your component, you can now import other components more easily.

In the example, the `Head` component is resolved from `@Theme/Head.vue` to `/resources/views/Components/Theme/Head.vue`

```typescript
<script setup lang="ts">
import Head from '@Theme/Head.vue'
</script>

<template>
  <Head></Head>
</template>
```

## Vue plugins

Vue provides many plugins that can improve DX.

### Define Options

As we use Vue Composition API, some features are not available in `<script setup>` like defining component name and other properties, ... and that are available in Vue Options API.

There is a plugin that provides a `defineOptions` macro that can be used in `<script setup>`

**Installation**

```bash
yarn add unplugin-vue-define-options --dev
```

**Configuration of the plugin in /vite.config.js**

```typescript
import DefineOptions from 'unplugin-vue-define-options/vite';

export default defineConfig({
    plugins: [
        ...
        DefineOptions(),
    ],
});
```

**Usage**

```typescript
<script setup lang="ts">
defineOptions({
  name: 'PageContent',
  layoutName: 'FullPage',
})
</script>
```

---

> We'll do our best to provide source code of the serie on [GitHub](https://github.com/Codivores/tutorial-laravel-twill-inertia-vue3-vite-tailwind)

%[https://github.com/Codivores/tutorial-laravel-twill-inertia-vue3-vite-tailwind]