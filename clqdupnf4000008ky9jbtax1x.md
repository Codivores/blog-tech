---
title: "Twill / Inertia / Vue 3 block party"
seoTitle: "Laravel Twill Inertia - Block party"
seoDescription: "Deep dive into Twill, a CMS package for Laravel, create and publish dynamic content with reusable Twill and Vue blocks"
datePublished: Sun Dec 17 2023 23:00:00 GMT+0000 (Coordinated Universal Time)
cuid: clqdupnf4000008ky9jbtax1x
slug: ltivt-10-laravel-twill-inertia-vue3-block-party
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1703069681973/7b381238-77b3-4f70-8a61-348af89a4caa.avif
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1703015664580/5435a9ac-8cc2-4c0d-b80d-6798191420af.avif
tags: laravel, php, vuejs, inertiajs, twill

---

Now that we can create pages, it's time to leverage Twill's powerful content management features and construct reusable blocks.

In this article, we will focus on creating a basic `Title` Twill Block and explore ways to enhance Developer Experience for subsequent blocks.

## TLDR

To start simply, we will create a `Title` Block in the `Common` namespace with a simple translatable Twill Text Input.

To achieve this, here are the different steps

1. Create a Twill Block component and add it the the block editor of our `PageContent` Module
    
2. Improve our Twill Models to compute blocks data and make them ready to use in our Inertia Vue Page
    
3. Create a Vue 3 component for the Block
    
4. Use it in our `PageContent` page
    

## Twill Block creation

For detailed information, refer to the [**official documentation**](https://twillcms.com/docs/block-editor/creating-a-block-editor.html).

### Generation

A Block can be initiated with the CLI generator. Let's create a `Title` Block in the `Common` namespace.

```bash
# php artisan twill:make:componentBlock namespace.name
php artisan twill:make:componentBlock common.title
```

The output we can see in our terminal:

```bash
Class written to /var/www/app/View/Components/Twill/Blocks/Common/Title.php
View written to /var/www/resources/views/components/twill/blocks/common/title.blade.php
```

### Files generated

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1703065527958/318b4ed5-3190-4193-a2ad-5f2a2d3166aa.png align="left")

Since the view file is intended for rendering Blocks through Blade, it can be confidently removed.

The component class skeleton is as follows:

**/app/View/Components/Twill/Blocks/Common/Title.php**

```php
<?php

namespace App\View\Components\Twill\Blocks\Common;

use A17\Twill\Services\Forms\Fields\Wysiwyg;
use A17\Twill\Services\Forms\Form;
use A17\Twill\Services\Forms\Fields\Input;
use A17\Twill\View\Components\Blocks\TwillBlockComponent;
use Illuminate\Contracts\View\View;

class Title extends TwillBlockComponent
{
    public function render(): View
    {
        return view('components.twill.blocks.common.title');
    }

    public function getForm(): Form
    {
        return Form::make([
            Input::make()->name('title'),
            Wysiwyg::make()->name('text')
        ]);
    }
}
```

We can see that the Block component purpose is to define the form to manage the content and how render it.

As we

* won't use Blade engine for rendering, but we need to implement the `render()`method
    
* want to have just a translatable Text Input
    

we can adapt the code this way:

```php
<?php

namespace App\View\Components\Twill\Blocks\Common;

use A17\Twill\Services\Forms\Form;
use A17\Twill\Services\Forms\Fields\Input;
use A17\Twill\View\Components\Blocks\TwillBlockComponent;
use Illuminate\Contracts\View\View;

class Title extends TwillBlockComponent
{
    public function render(): View
    {
        return view();
    }

    public function getForm(): Form
    {
        return Form::make([
            Input::make()
                ->name('title')
                ->label(__('Title'))
                ->translatable(),
        ]);
    }
}
```

## **Adding the block to the BlockEditor of our Module**

We already created a [BlockEditor field in our `PageContent` Form](https://tech.codivores.com/ltivt-5-laravel-twill-first-module#heading-form-through-form-builder), clicking on the `Add content` button, we can see our `Title` block in the dropdown with default Twill blocks:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1703067178665/75d3d1d1-b492-4d1d-8057-bb839dbbee4c.png align="center")

Selecting it, a translatable Text Input is displayed.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1703067245802/2fe505f8-3f0f-483b-bfa1-23858372cca3.png align="center")

## Twill Block improvements

For now, we have seen the basics of a Block, but there are helpers that improve the experience. As we are going to create many blocks, a little refactoring would not be useless.

### **Creating a BlockComponent class for our project**

**/app/View/Components/Twill/Blocks/Base/BlockComponent.php**

```php
<?php

namespace App\View\Components\Twill\Blocks\Base;

use A17\Twill\Services\Forms\Form;
use A17\Twill\View\Components\Blocks\TwillBlockComponent;
use Illuminate\Contracts\View\View;

class BlockComponent extends TwillBlockComponent
{
    public function render(): View
    {
        return view();
    }

    public function getForm(): Form
    {
        return Form::make([]);
    }

    public static function getBlockGroup(): string
    {
        return '';
    }
}
```

**What it does**

* Defines default `render()` and `getForm()` methods, preventing to implement it in every block
    
* Overrides the `getBlockGroup()` helper, that is `app-` by default for every block and does not represent our code organization
    

> We will see later, but this class will allow us, for example, to globally define the default editor and toolbar of WYSIWYG fields.

### **Creating a Base Block class for our namespace**

**/app/View/Components/Twill/Blocks/Common/Base.php**

```php
<?php

namespace App\View\Components\Twill\Blocks\Common;

use App\View\Components\Twill\Blocks\Base\BlockComponent;

class Base extends BlockComponent
{
    public static function getBlockGroup(): string
    {
        return 'common-';
    }
}
```

Its purpose is essentially to define the group for all blocks in this namespace.

### And now our Block

**/app/View/Components/Twill/Blocks/Common/Base.php**

```php
<?php

namespace App\View\Components\Twill\Blocks\Common;

use A17\Twill\Services\Forms\Form;
use A17\Twill\Services\Forms\Fields\Input;

class Title extends Base
{
    public function getForm(): Form
    {
        return Form::make([
            Input::make()
                ->name('title')
                ->label(__('Title'))
                ->translatable(),
        ]);
    }

    public static function getBlockTitleField(): ?string
    {
        return 'title';
    }

    public static function getBlockTitle(): string
    {
        return __('Title');
    }

    public static function getBlockIcon(): string
    {
        return 'wysiwyg_header';
    }
}
```

**What it does**

* Extends our Base Block in our Common namespace
    
* Provides a dynamic title with `getBlockTitleField()` using the `title` field. It's not mandatory but can be helpful when there are many blocks in the BlockEditor
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1703068780933/aef3469a-70a0-4769-bed4-4e1ed6975b4e.png align="center")
    
* Customizes the title with `getBlockTitle()` and the icon with `getBlockIcon()`
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1703069108870/3902bf64-4831-4193-9009-21aa726c96d6.png align="left")
    

### **Filtering allowed blocks on our module**

In the BlockEditor field, it is possible to define the list of allowed blocks. Depending on the page templates and the number of blocks, it is a good practice to define exactly the list of available blocks.

Taking the controller of our `PageContent` Module, in the `getForm()` method, we will call the `blocks` method, giving it the list of blocks.

**/app/Http/Controllers/Twill/PageContentController.php**

```php
...    

    public function getForm(TwillModelContract $model): Form
    {
        $form = parent::getForm($model);

        $form->add(
            BlockEditor::make()
                ->withoutSeparator()
                ->blocks([
                    'common-title',
                ])
        );

        return $form;
    }

...
```

## Let's take a look on the frontend side

A Twill Block is a Laravel Eloquent Model and therefore contains a lot of information (attributes, relations, ...) that are not necessary.

Here is what it looks like if we inject the block as is:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1703069583342/3205f11b-7f9f-46d5-886c-a28a82fc7472.png align="center")

### Enhance our Module Models to prepare the Blocs data let's create a Base Model with some methods

We will create a Model that extends Twill's Model in which we will add a method to prepare Block data.

> For now, we will just handle the blocks, but this class will also allow us to manage Model medias, files, slug, and medias, browsers, ... in blocks

```php
<?php

namespace App\Models\Base;

use A17\Twill\Models\Model as TwillModel;
use Illuminate\Support\Arr;

class Model extends TwillModel
{
    public function computeBlocks(string $locale = null): void
    {
        $locale = $locale ?? app()->getLocale();
        $fallbackLocale = config('translatable.use_property_fallback', false) ? config('translatable.fallback_locale') : $locale;

        $blocks = $this->blocks
            ->where('parent_id', null)
            ->map(function ($block) use ($locale, $fallbackLocale) {
                $block->childs = $this->blocks
                    ->where('parent_id', $block->id)
                    ->map(function ($blockChild) use ($locale, $fallbackLocale) {
                        $blockChild->childs = $this->blocks
                            ->where('parent_id', $blockChild->id)
                            ->map(function ($blockChildChild) use ($locale, $fallbackLocale) {
                                return $this->computeBlock($blockChildChild, $locale, $fallbackLocale);
                            })
                            ->values();

                        $block = $this->computeBlock($blockChild, $locale, $fallbackLocale);
                        return $block;
                    })
                    ->values();

                $block->unsetRelation('children');

                return $this->computeBlock($block, $locale, $fallbackLocale);
            })->values();

        $this->unsetRelation('blocks');
        $this->blocks = $blocks->values();
    }

    private function computeBlock($block, string $locale, string $fallbackLocale = null): array
    {
        // Handle translated content inputs.
        if (is_array($block->content) && count($block->content) > 0) {
            $blockContent = $block->content;
            foreach ($blockContent as $field => $value) {
                if (is_array($value)) {
                    if (isset($value[$locale]) || isset($value[$fallbackLocale])) {
                        $blockContent[$field] = $block->translatedInput($field);
                    } else {
                        foreach (config('translatable.locales') as $allowedLocale) {
                            if (isset($value[$allowedLocale])) {
                                $blockContent[$field] = null;
                                break;
                            }
                        }
                    }
                }
            }
            $block->content = $blockContent;
        }

        return $block->only(Arr::collapse(
            [
                [
                    'editor_name',
                    'position',
                    'type',
                    'content',
                ],
                ($block->childs && count($block->childs) > 0) ? ['childs'] : []
            ]
        ));
    }
}
```

### **Adaptations of the Model of our module**

It must now:

* extend our Base Model
    
* declare the `blocks` attribute in its `$publicAttributes` array attribute
    

```php
<?php

// use A17\Twill\Models\Model;
use App\Models\Base\Model;

class PageContent extends Model implements Sortable
{
    public array $publicAttributes = [
        'title',
        'meta_title',
        'meta_description',
        'blocks',
    ];
}
```

### **Adapt our existing Module controllers**

On both Back and Frontend controllers, we will call the `computeBlocks()` method on our `item`.

**/app/Http/Controllers/Admin/PageContentController.php**

```php
<?php

...

use App\Models\Base\Model;

class PageContentController extends BaseModuleController
{
    ...

    /**
     * @param Model $item
     * @return array
     */
    protected function previewData($item)
    {

        $item->computeBlocks();


        return $this->previewForInertia($item->only($item->publicAttributes), [
            'page' => 'Page/Content',
        ]);
    }
}
```

**/app/Http/Controllers/App/PageContentController.php**

```php
<?php

...

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


                    $item->computeBlocks();


                }
                return $item;
            }
        );

        ...
    }
}
```

Here is what we have now on the Vue side:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1703070694430/6ae5aa15-d9b0-4374-9bc2-547a0060eb35.png align="center")

> We could clean up more by removing `editor_name` and `position`, but we decided to keep them because it happens to us to have several BlockEditor fields on the same Module and also to have display behaviors depending on the position (for example, for our `Title` block, we could put an `h1` if position is 1 and an `h2` otherwise).

## Vue Block creation

Now we have an array of blocks that we can process in ou Inertia Page as a Vue Single-File Component (SFC)

### Block component

We will create a Vue component equivalent of the Twill Block.

**/resources/views/Components/Theme/Block/Common/Title.vue**

```javascript
<script setup lang="ts">
defineOptions({
  name: 'BlockCommonTitle',
})

interface Props {
  block: Model.Block & PropsBlock
}

type PropsBlock = {
  content: {
    title?: string | null
  }
}

defineProps<Props>()
</script>

<template>
  <h1
    v-if="block.content?.title"
    v-html="block.content.title"
    class="text-center text-4xl font-semibold text-gray-900"
  ></h1>
</template>
```

### TypeScript types enrichment

We create a `Block` type and add `blocks` to our `Page` type:

**/resources/js/types/models.d.ts**

```javascript
declare namespace Model {
  export type Page = {
    ...
    blocks?: Array<Block> | null
  }

  export type Block = {
    editor_name: string
    position: number
    type: string
    content: {} | null
  }
}
```

### Adding a `@Block` alias

**/vite.config.js**

```javascript
import { defineConfig } from 'vite'

export default defineConfig({
  resolve: {
    alias: [
      {
        find: '@Block',
        replacement: '/resources/views/Components/Theme/Block',
      },
      
      ...
    ],
  },
})
```

**/tsconfig.json**

```javascript
{
  "compilerOptions": {
    ...

    "paths": {
      "@Block/*": ["resources/views/Components/Theme/Blocks/*"],

      ...
    },

    ...
  }
}
```

## Integration into our Inertia Vue Page

* We will load our blocks asynchronously, avoiding unnecessary component loading.
    
* In the template, if our item contains blocks, we will loop through the list and load the appropriate Vue component based on its type value.
    

**/resources/views/Pages/Page/Content.vue**

```javascript
<script setup lang="ts">
import Head from '@Theme/Head.vue'

interface Props {
  item: Model.Page
}

defineProps<Props>()

const BlockCommonTitle = defineAsyncComponent(() => import('@Block/Common/Title.vue'))
</script>

<template>
  <Head :item="item"></Head>

  <div
    v-if="item?.blocks && Array.isArray(item.blocks) && item.blocks.length > 0"
    class="mx-auto w-full max-w-6xl"
  >
    <div
      v-for="(block, index) in item.blocks"
      :key="index"
    >
      <BlockCommonTitle
        v-if="block.type == 'common-title'"
        :block="block"
      ></BlockCommonTitle>
    </div>
  </div>
</template>
```

The rendering is as follows:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1703076901807/3c85f4b3-b0c8-484e-9871-9b94583fb1d5.png align="center")

---

**We now have our first Block, we will see in a later article how to create other blocks with medias, links, HTML, ...**

---

> We'll do our best to provide source code of the serie on [GitHub](https://github.com/Codivores/tutorial-laravel-twill-inertia-vue3-vite-tailwind)

%[https://github.com/Codivores/tutorial-laravel-twill-inertia-vue3-vite-tailwind]