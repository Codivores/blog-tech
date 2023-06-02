---
title: "Twill preview with Inertia"
seoTitle: "Laravel Twill - Preview with Inertia"
seoDescription: "Deep dive into Twill, a CMS package for Laravel, and preview your content in an Inertia page in your admin"
datePublished: Fri Jun 02 2023 06:00:39 GMT+0000 (Coordinated Universal Time)
cuid: clie5pfqv0d1wyanvb25y2jut
slug: ltivt-9-laravel-twill-preview-with-inertia
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1685630161377/cee34abe-e15b-4395-a75f-b39431e40c8e.avif
tags: laravel, php, vuejs, inertiajs, twill

---

If a Twill module uses Revisions, you can benefit from an awesome Twill feature: preview your changes before publishing and compare old revisions. By default, it uses Blade views, so working with Inertia needs some adjustments.

## The principle

For now, Inertia pages cannot be rendered in an iframe through `srcdoc` attribute (that is the way Twill manages the preview): [https://github.com/inertiajs/inertia/issues/1008](https://github.com/inertiajs/inertia/issues/1008).

We need to implement a workaround to get our Inertia pages rendered by the Twill Blade view:

1. Prepare the data for Twill `preview()` method in our Module Controller
    
2. Render the Twill preview Blade view that will redirect to a custom PreviewController
    
3. Retrieve and bind the data in the PreviewController to render the requested Inertia page
    

## Prepare the data for the Blade view

The aim is to

* put all the needed data in session with a specific key
    
* provide to the Blade view this session key and the route to the PreviewController
    

To achieve this, we will create a Trait with our custom logic and use it in our Module controllers.

### Trait creation to prepare the data

**/app/Http/Controllers/Twill/Traits/HasPreview.php**

```php
<?php

namespace App\Http\Controllers\Twill\Traits;

use Illuminate\Support\Str;

trait HasPreview
{
    protected function previewForInertia($item, array $config): array
    {
        if (in_array('blocks', $config)) {
            $item->computeBlocks();
        }

        if (isset($config['medias']) && is_array($config['medias'])) {
            foreach ($config['medias'] as $role) {
                $item->computeMedias($role);
            }
        }

        $sessionKey = 'adminPreview_' . Str::random(40);

        request()->session()->flash($sessionKey, [
            'module' => Str::singular($this->moduleName),
            'page' => $config['page'] ?? null,
            'itemPreview' => $item,
            'props' => $config['props'] ?? [],
        ]);

        return [
            'route' => route('twill.preview'),
            'sessionKey' => $sessionKey,
        ];
    }
}
```

**What it does**

* Computes `blocks` and `medias` of the item for Inertia rendering (we will see this part in detail in a further article).
    
* Puts in session:
    
    * the module's singular name in `module`
        
    * the Inertia page to render in `page`
        
    * the Model item mapped to `itemPreview`
        
    * the potential additional `props` that will be injected into the Inertia page
        
* Gives the Twill `preview()` method:
    
    * the `route` to the PreviewController
        
    * the `sessionKey` to retrieve our data
        

### Using the Trait in our Module controller

**/app/Http/Controllers/Admin/PageContentController.php**

```php
<?php

use App\Http\Controllers\Twill\Traits\HasPreview;

class PageContentController extends TwillModuleController
{
    use HasPreview;

    protected $previewView = 'twill.preview';

    protected function previewData($item)
    {
        return $this->previewForInertia($item->only($item->publicAttributes), [
            'page' => 'Page/Content',
        ]);
    }
}
```

## Create the preview Blade view

**/resources/views/twill/preview.blade.php**

```xml
<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0">
    </head>

    <body onload="window.location='{{ $route }}?sessionKey={{ $sessionKey }}'">
    </body>
</html>
```

## Render the Inertia page through the PreviewController

### Route

**/routes/twill.php**

```php
<?php

use App\Http\Controllers\Twill\Base\PreviewController;

Route::get('preview', PreviewController::class)->name('preview');
...
```

### Controller

**/app/Http/Controllers/Twill/Base/PreviewController.php**

```php
@@ -0,0 +1,47 @@
<?php

namespace App\Http\Controllers\Twill\Base;

use A17\Twill\Http\Controllers\Admin\Controller;
use App\Http\Controllers\Twill\Base\ModuleController;
use Illuminate\Contracts\Foundation\Application;
use Illuminate\Http\Response;
use Illuminate\Http\Request;
use Inertia\Inertia;
use Inertia\Response as InertiaResponse;

class PreviewController extends ModuleController
{
    public function __construct(Application $app, Request $request)
    {
        Controller::__construct($app, $request);
        $this->app = $app;
        $this->request = $request;
    }

    public function __invoke(Request $request): InertiaResponse
    {
        abort_if(!$request->has('sessionKey'), Response::HTTP_NOT_FOUND);

        $sessionKey = $request->input('sessionKey');

        abort_if(!$request->session()->has($sessionKey), Response::HTTP_NOT_FOUND);

        $input = $request->session()->get($sessionKey);

        abort_if(!isset($input['module']) || !isset($input['itemPreview']), Response::HTTP_NOT_FOUND);

        $item = $input['itemPreview'];

        $props = $input['props'] ?? [];
        if (!isset($props['locale'])) {
            $props['locale'] = app()->getLocale();
        }

        $page = $input['page'] ?? ucfirst($input['module']) . '/Item';

        return match ($input['module']) {
            default => Inertia::render($page, ['item' => $item] + $props),
        };
    }
}
```

**What it does**

* Checks if the `sessionKey` is provided in the request
    
* Checks if the session exists and has the required data
    
* Retrieves the props and adds the `locale` props
    
* Computes the Inertia page to render.  
    If the page is not set in your `previewData()` data, it uses the `Item.vue` SFC located in the `ModuleName` directory (for example, for a `Projects` Twill module, it will look for `/resources/views/Pages/Project/Item.vue`)
    
* Renders the Inertia page with the provided props (we use a `match()` on the module name as we may have specific logic for some modules)
    

## Improvement

We can create our base Twill controllers (for Module and Singleton) that use the Trait and define the previewView and make our controllers extend those to avoid defining everything in every Module controller that needs preview.

To do so, we need base controllers to extend the:

* Twill Module controller
    
* Twill Singleton Module controller
    

### Create Base controllers in the application

**/app/Http/Controllers/Admin/Base/ModuleController.php**

```php
<?php

namespace App\Http\Controllers\Twill\Base;

use A17\Twill\Http\Controllers\Admin\ModuleController as TwillModuleController;
use App\Http\Controllers\Twill\Traits\HasPreview;

class ModuleController extends TwillModuleController
{
    use HasPreview;

    protected $previewView = 'twill.preview';
}
```

**/app/Http/Controllers/Admin/Base/SingletonModuleController.php**

```php
<?php

namespace App\Http\Controllers\Twill\Base;

use A17\Twill\Http\Controllers\Admin\SingletonModuleController as TwillSingletonModuleController;
use App\Http\Controllers\Twill\Traits\HasPreview;

class SingletonModuleController extends TwillSingletonModuleController
{
    use HasPreview;

    protected $previewView = 'twill.preview';
}
```

### Adapt our existing Module controllers

**/app/Http/Controllers/Admin/PageContentController.php**

```php
<?php

// use A17\Twill\Http\Controllers\Admin\ModuleController as BaseModuleController;
use App\Http\Controllers\Twill\Base\ModuleController as BaseModuleController;

class PageContentController extends BaseModuleController
{
    protected function previewData($item)
    {
        return $this->previewForInertia($item->only($item->publicAttributes), [
            'page' => 'Page/Content',
        ]);
    }
}
```

**/app/Http/Controllers/Admin/PageHomeController.php**

```php
<?php

// use A17\Twill\Http\Controllers\Admin\SingletonModuleController as BaseModuleController;
use App\Http\Controllers\Twill\Base\SingletonModuleController as BaseModuleController;

class PageHomeController extends BaseModuleController
{
    protected function previewData($item)
    {
        return $this->previewForInertia($item->only($item->publicAttributes), [
            'page' => 'Page/Home',
        ]);
    }
}
```

---

> We'll do our best to provide source code of the serie on [GitHub](https://github.com/Codivores/tutorial-laravel-twill-inertia-vue3-vite-tailwind)