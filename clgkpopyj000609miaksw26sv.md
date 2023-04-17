---
title: "Application level improvements"
seoTitle: "Laravel Twill Inertia - Application level improvements"
seoDescription: "Improve your application structure using layouts, global head component"
datePublished: Mon Apr 17 2023 10:47:10 GMT+0000 (Coordinated Universal Time)
cuid: clgkpopyj000609miaksw26sv
slug: ltivt-7-application-level-improvements
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1681665253746/8a644a6b-bcaf-4ca9-9d08-91609d0698ea.avif
tags: laravel, vuejs, inertiajs, twill

---

## Layouts

For the moment, we have just one Twill module with one Inertia page, but we will probably have more page templates with different modules in our application (home, blog, portfolio, contact, customer account, ...).

It may be a good idea to create one (or more) layouts that will be used by our different pages and will be persistent (not destroyed and recreated every visit). For this article, we will create 2 layouts and see how to use it on our pages:

* a Default for the common pages (which may have a header with a menu, a footer, ...)
    
* a FullPage for custom pages (like authentication)
    

### Layouts creation

We create our 2 templates in a new directory `/resources/views/Components/Layout`:

**/resources/views/Components/Layout/Default.vue**

```xml
<template>
  <div class="min-h-full">
    <header class="bg-white shadow">
      <div class="mx-auto max-w-7xl px-4 py-6">
        <h1 class="text-3xl font-bold tracking-tight text-gray-900">My Application</h1>
      </div>
    </header>
    <main>
      <div class="mx-auto max-w-7xl py-6">
        <slot></slot>
      </div>
    </main>
  </div>
</template>
```

**/resources/views/Components/Layout/FullPage.vue**

```xml
<template>
  <main class="w-full flex flex-col h-screen content-center justify-center">
    <slot></slot>
  </main>
</template>
```

### Configuration

For the moment, our layouts are just Vue components, to make Inertia aware of the layout every page has to load, we need to do some changes in our application entry point in the `resolve()` method:

**/resources/js/app.ts**

**before:**

```typescript
  resolve: async (name: string) => {
    let page = await resolvePageComponent(`../views/Pages/${name}.vue`, import.meta.glob<DefineComponent>('../views/Pages/**/*.vue'))

    return page
  },
```

**after:**

```typescript
  resolve: async (name: string) => {
    let page = await resolvePageComponent(`../views/Pages/${name}.vue`, import.meta.glob<DefineComponent>('../views/Pages/**/*.vue'))

    page = page.default

    page.layout = (await import(`../views/Components/Layout/${page.layoutName || 'Default'}.vue`)).default

    return page
  },
```

**What it does**

To set the `page.layout` value, it will

* look if the page has a `layoutName` option defined and then load the Component with that name which would be in `/resources/views/Components/Layout` directory
    
* if not, load the `Default` layout
    

More info in the [official documentation](https://inertiajs.com/pages#creating-layouts).

> We don't test if the layout Component file exists, if a wrong name is defined, it will trigger an error:
> 
> ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1681721891269/63c78f7e-e8c8-4f70-acc7-8a723e04561a.png align="left")

### Define the Layout of the page

Now we can choose our layout on each page.

If you use the `unplugin-vue-define-options` plugin, it can be done like that:

**/resources/views/Pages/Page/Content.vue**

```javascript
<script setup lang="ts">
defineOptions({
  layoutName: 'FullPage',
})

interface Props {
  item: Model.Page
  locale: string
}

defineProps<Props>()
</script>
```

If not, we need to use Option API like that:

**/resources/views/Pages/Page/Content.vue**

```javascript
<script lang="ts">
export default {
  layoutName: 'FullPage',
}
</script>

<script setup lang="ts">
interface Props {
  item: Model.Page
  locale: string
}

defineProps<Props>()
</script>
```

You should see:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1681722777316/5a7e0239-4c06-4c09-8ef8-34ea1e229743.png align="center")

And if you remove the configuration or set `layoutName` to `Default`, you should see:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1681722736612/f2fdcd63-d97d-426c-a68b-4056c388dca0.png align="center")

## Head

In Web applications, it's a good practice to define a page `title`, and maybe `meta tags` (like description). Inertia provides a `Head` component that helps us for this purpose. More info in the [official documentation](https://inertiajs.com/title-and-meta).

### Our Head component

To have a better organization and more flexibility, we will create our `Head` component that will extend the Inertia's Head component.

**/resources/views/Components/Theme/Head.vue**

```typescript
<script setup lang="ts">
import { Head as InertiaHead } from '@inertiajs/vue3'

defineOptions({
  name: 'ThemeHead',
})

interface Props {
  item?: any
  title?: string
  description?: string
}

const props = withDefaults(defineProps<Props>(), {
  item: () => ({}),
  title: '',
  description: '',
})

const appName = 'My Application'

const title =
  props.item && props.item.meta_title && props.item.meta_title != ''
    ? props.item.meta_title
    : props.item && props.item.title && props.item.title != ''
    ? props.item.title
    : props.title

const description =
  props.item && props.item.meta_description && props.item.meta_description != ''
    ? props.item.meta_description
    : props.item && props.item.description && props.item.description != ''
    ? props.item.description
    : props.description
</script>

<template>
  <InertiaHead :title="title ? `${title} | ${appName}` : appName">
    <template v-if="description !== ''">
      <meta
        head-key="description"
        name="description"
        :content="description"
      />
    </template>
  </InertiaHead>
</template>
```

**What it does**

The component defines `title` and `meta-description` tags from props

* if an `item` prop is provided (from PageContent, PageHome, BlogArticle, ...), it looks for:
    
    * `meta_title` and `meta_description` item properties
        
    * then `title` and `description` properties if not defined
        
* if not, looks for `title` and `description` props
    

The `title` is filled out by the `appName` variable (here it is static text `My Application`, you can make it dynamic or add your logic according to your project needs).

### Usage

**/resources/views/Pages/Page/Content.vue**

```javascript
<script setup lang="ts">
import Head from '@Theme/Head.vue'

...
</script>

<template>
  <Head :item="item"></Head>
  ...
</template>
```

If we go back to our Twill module, if we define the SEO information:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1681727190157/59f0ec14-028e-4602-8d45-37d345534987.png align="center")

They will be rendered on the HTML page:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1681727258929/9efc5534-a049-40df-8b5a-0604103ab3a0.png align="left")

---

> We'll do our best to provide source code of the serie on [GitHub](https://github.com/Codivores/tutorial-laravel-twill-inertia-vue3-vite-tailwind)

%[https://github.com/Codivores/tutorial-laravel-twill-inertia-vue3-vite-tailwind]