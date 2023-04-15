---
title: "What about building a CMS, let's Twill"
seoTitle: "Laravel Twill CMS package installation, configuration and basics"
seoDescription: "Discover Twill, a CMS package for Laravel, get the basic configuration tips and build with us your first content pages"
datePublished: Thu Aug 19 2021 18:27:05 GMT+0000 (Coordinated Universal Time)
cuid: cksj9aba304h31ws1co9fb2ts
slug: ltivt-4-laravel-twill-basics
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1629392987770/Vblh721Vm.png
tags: laravel, php, cms, twill

---

> **Updated version for Laravel 10 / Twill 3 in progress**

In this article, we will see

* [Twill installation and configuration](#installation-and-configuration)
    
    * [Dependencies installation](#dependencies-installation)
        
    * [Twill installation](#twill-installation)
        
    * [URL configuration](#where-is-my-twill)
        
    * [Multi-language configuration](#multi-language-configuration)
        
* [Creation and customization of a first content module](#ready-to-create-content)
    
    * [Content Module creation](#ready-to-create-content)
        
    * [Route and menu customization](#route-and-menu-management)
        
    * [Model customization](#model-customization)
        
    * [Admin Controller customization](#admin-controller-customization)
        

## Installation and configuration

### Dependencies installation

Twill comes as a Laravel package that can be installed via Composer. You can refer to the [official documentation](https://twillcms.com/docs/) for more information.

```bash
composer require area17/twill:"^3.0"
```

### Twill installation

#### What does that mean?

Twill installation will

* Create configuration files
    
    * `twill.php`: global configuration of the package
        
    * `translatable.php`: Twill uses [astrotomic/laravel-translatable](https://github.com/Astrotomic/laravel-translatable) package to handle multi-language content
        
* Create a `twill.php` route file to declare the routes of our modules
    
* Publish the assets of the administration interface (Twill is built with Vue.js 2)
    
* Migrate all the package tables: twill users, settings, blocks, medias, files, ...
    
* Prompt for a superadmin account creation (you can create it or new ones later with this command `php artisan twill:superadmin`)
    

If you want to change the default table names, create the `twill.php` config file before proceeding to the installation and define your custom names.

**Default table names**

```php
    'blocks_table' => 'twill_blocks',
    'features_table' => 'twill_features',
    'fileables_table' => 'twill_fileables',
    'files_table' => 'twill_files',
    'mediables_table' => 'twill_mediables',
    'medias_table' => 'twill_medias',
    'password_resets_table' => 'twill_password_resets',
    'related_table' => 'twill_related',
    'settings_table' => 'twill_settings',
    'tagged_table' => 'twill_tagged',
    'tags_table' => 'twill_tags',
    'users_oauth_table' => 'twill_users_oauth',
    'users_table' => 'twill_users',
```

We generally change the table names to set `admin` prefix instead of `twill` for admin users and add `content` prefix for all other tables.

**/config/twill.php**

```php
<?php

return [

    /*
    |--------------------------------------------------------------------------
    | Twill default tables naming configuration
    |--------------------------------------------------------------------------
    |
    */

    'blocks_table' => 'content_blocks',
    'features_table' => 'content_features',
    'fileables_table' => 'content_fileables',
    'files_table' => 'content_files',
    'mediables_table' => 'content_mediables',
    'medias_table' => 'content_medias',
    'password_resets_table' => 'admin_password_resets',
    'related_table' => 'content_related',
    'settings_table' => 'content_settings',
    'tagged_table' => 'content_tagged',
    'tags_table' => 'content_tags',
    'users_oauth_table' => 'admin_users_oauth',
    'users_table' => 'admin_users',

];
```

#### Ok, let's do it

We have an email address *(no mail will be sent)* and a password, we are ready for the installation process, so let's execute the following command:

```bash
php artisan twill:install
```

### Where is my Twill

The administration interface is available by default as `admin` path of your `APP_URL` environment variable (defined when you installed Laravel in your `/.env` file): `http://localhost/admin`.

Twill allows you to have it available where you want with environment variables:

* For a fully dedicated domain: set an `ADMIN_APP_URL` environment variable with the URL (ex: `ADMIN_APP_URL=tech.codivores.com`)
    
* As a path of your main domain: set an `ADMIN_APP_PATH` environment variable with the name of the subdirectory (ex: `ADMIN_APP_PATH=/administration`).
    

You can find more configuration options in the [official documentation](https://twillcms.com/docs/getting-started/installation.html#content-admin-path-and-domain) like strict domain handling when using a path.

In most cases, we make the administration available as a path of the main domain. And depending on our clients, we set the `ADMIN_APP_PATH` environment variable in the `.env` file (or as an OS environment variable) with the name they want and that is not too obvious for an administration interface.

/!\\ If you are using a path, there's a little buggy behavior: if you try to login through `http://localhost/admin/login.php` instead of `http://localhost/admin`, you are well logged in but redirected to the `APP_URL`... To prevent this, you can edit the default `auth_login_redirect_path` config:

**/config/twill.php**

```php
return [

    /*
    |--------------------------------------------------------------------------
    | Twill Auth related configuration
    |--------------------------------------------------------------------------
    |
     */
    'auth_login_redirect_path' => env('ADMIN_APP_PATH', '/admin'),

];
```

We can now go to `http://localhost/admin` in a browser and see the login page.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1681485807160/f3df7f2b-397c-419f-87ca-ecd7b14e1e40.png align="center")

Filling the login form with the superadmin email and password we set earlier, we can see a beautiful empty administration interface 🙌.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1681485842850/7401a84c-f3bf-4345-9e7f-e7ed9a7b04c4.png align="center")

### Multi-language configuration

Twill allows you to manage your content in multiple languages and also have the administration in different languages.

#### Multi-language in the Front

Just add your locales in the `locales` array of the [astrotomic/laravel-translatable](https://github.com/Astrotomic/laravel-translatable) package configuration file. We will just add French beside the default English locale for the tutorial.

**/config/translatable.php**

```php
<?php

return [

    /*
    |--------------------------------------------------------------------------
    | Application Locales
    |--------------------------------------------------------------------------
    |
    | Contains an array with the applications available locales.
    |
    */

    'locales' => [
        'en',
        'fr',
    ],
```

#### Multi-language in the Back

Twill handles some locales for the administration interface (at this time: `ar, bs, de, en, fr, nl, pl, pt, ru, tr, zh-Hans` - feel free to contribute and add yours).

Admin users can choose their language on their profile page, but you can change the default `en` locale in the Twill configuration file if you prefer.

**/config/twill.php**

```php
<?php

return [

    /*
    |--------------------------------------------------------------------------
    | Twill app locale
    |--------------------------------------------------------------------------
    |
    */

    'locale' => 'fr',
```

## Ready to create content?

In Twill terminology, a Module represents all the files needed to manage a content type: Model (with migration), Repository, Controller, Request and Form view.

A Module can be initiated with the CLI generator, if we look at the [official documentation](https://twillcms.com/docs/modules/cli-generator.html), we see it's an artisan command that takes options:

```bash
php artisan twill:make:module moduleName {options}
```

The `moduleName` is the singular name of your content type and Twill generates some files with plural name (so avoid `news` as it already ends with a `s` and will break some functionalities).

The options are (taken from the [official documentation](https://twill.io/docs/#cli-generator)):

> \--hasBlocks (-B), to use the block editor on your module form

> \--hasTranslation (-T), to add content in multiple languages

> \--hasSlug (-S), to generate slugs based on one or multiple fields in your model

> \--hasMedias (-M), to attach images to your records

> \--hasFiles (-F), to attach files to your records

> \--hasPosition (-P), to allow manually reordering of records in the listing screen

> \--hasRevisions (-R), to allow comparing and restoring past revisions of records
> 
>   
> \--hasNesting (-N), to enable nested items in the module listing  
> \--parentModel=, to generate the route for a nested module. See ( see Nested Module)
> 
>   
> \--bladeForm, to generate a Blade form instead of using the new Form builder (more info [here](https://twillcms.com/blog/twill-3-introducing-oop-builders.html))

Twill is amazing for this as you can choose the features you want to have available for your content, keeping simple content simple. All work with PHP Traits and additional classes, so if you forget an option, you can add it later based on sample codes.

### Let's create our first Module to handle static pages

We will call this Module *contentPage* and we will need Blocks (to have flexible content), Translations, Slugs, Medias, Position (to order them in the administration) and Revisions (we will not attach Files directly but maybe through Blocks):

```bash
php artisan twill:make:module pageContent -BTSMPR --bladeForm
```

The output we can see in our terminal:

```php
Migration created successfully! Add some fields!
Models created successfully! Fill your fillables!
Repository created successfully! Control all the things!
Controller created successfully! Define your index/browser/form endpoints options!
Form request created successfully! Add some validation rules!
Form view created successfully! You can now include your form fields.

 Do you also want to generate the preview file? [no]:
  [0] no
  [1] yes
 > 

The following snippet has been added to routes/twill.php:
-----
TwillRoutes::module('pageContents');
-----
To add a navigation entry add the following to your AppServiceProvider BOOT method.
-----
use A17\Twill\Facades\TwillNavigation;
use A17\Twill\View\Components\Navigation\NavigationLink;
TwillNavigation::addLink(
    NavigationLink::make()->forModule('pageContents')
);
-----
Do not forget to migrate your database after modifying the migrations.

Enjoy.
```

It creates the following files in your application:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1681566941833/8602ad02-6aa6-4308-a5f6-cf0d0e22de9b.png align="left")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1681566962922/23e66d76-9ad3-44e9-876c-c919497db3a9.png align="left")

### What next

* Check or edit the Routes according to the structure your want
    
* Add a navigation entry
    
* Customize the fields of the Model and the form accordingly
    
* Customize the admin Controller if needed
    

#### Routes configuration

Since Twill 3, the module routes are automatically added at the end of the file as a root entry (if you are still using Twill 2, you need to add it manually).

**/routes/twill.php**

```php
<?php

use A17\Twill\Facades\TwillRoutes;

TwillRoutes::module('pageContents');
```

#### Navigation entry

Since Twill 3, there is a new way to manage navigation, registering it in the AppServiceProvider (you still can use the legacy method defining you navigation in a `config/twill-navigation.php` file). More info in the [official documentation](https://twillcms.com/docs/getting-started/navigation.html))

You can copy/paste what the CLI output suggested:

**/app/Providers/AppServiceProvider.php**

```php
<?php

namespace App\Providers;

use A17\Twill\Facades\TwillNavigation;
use A17\Twill\View\Components\Navigation\NavigationLink;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        TwillNavigation::addLink(
            NavigationLink::make()->forModule('pageContents')
        );
    }
}
```

If you want to customize the title in the navigation, you can chain it with the call of `title()` method:

```php
    public function boot(): void
    {
        TwillNavigation::addLink(
            NavigationLink::make()
                ->forModule('pageContents')
                ->title(Str::ucfirst(__('pages')))
        );
    }
```

To see our Module administration interface without triggering an error, we need to execute the migration via `php artisan migrate`. We will do it later as we want to customize the attributes of our model, but here is an example of what you will see:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1681568293228/dcfda5dc-33dd-4767-9f11-e869aed065f5.png align="center")

This configuration add our Module on the global navigation (first level of the menu), but we have to organize our menu, so we will modify this configuration to access to our Module as a primary entry of a global menu `Content` (with Twill, you can have up to 3 levels for your menu):

**/routes/admin.php**

```xml
<?php

use Illuminate\Support\Facades\Route;

Route::group(['prefix' => 'content'], function () {
    Route::module('pages');
});
```

**/config/twill-navigation.php**

```xml
<?php

return [

    'content' => [
        'title' => 'Content',
        'route' => 'admin.content.pages.index',
        'primary_navigation' => [
            'pages' => [
                'title' => 'Pages',
                'module' => true,
            ],
        ],
    ],

];
```

**What it does**

For the route, using standard Laravel routing, we encapsulate our Module routes definition in a group with the `content` prefix. As Twill routes already add `admin` prefix, our Module routes names will start with `admin.content.pages`

For the Twill navigation, we encapsulate our menu definition in a `content` block which has the following attributes:

* title: the text displayed in the menu
    
* route: the route name of the page that will be displayed on click, for now, we want the index of our Module
    
* primary\_navigation: the list of the submenus
    

![Twill navigation organized](https://cdn.hashnode.com/res/hashnode/image/upload/v1626446745343/lgnJ_CqGh.png align="left")

#### Model customization

The migration, model and form files created by the generator is a template with default fields.

In our case we want:

* a title that can be translated
    
* meta title and description that can be translated
    
* the position to order our pages in the listing of the administration interface
    
* a block editor for all the content (we will focus on this part in a later article)
    

Here is what final files look like.

**/database/migration/...\_create\_pages\_table.php**

```xml
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;

class CreatePagesTables extends Migration
{
    public function up()
    {
        Schema::create('pages', function (Blueprint $table) {
            createDefaultTableFields($table);
        });

        Schema::table('pages', function (Blueprint $table) {
            $table->integer('position')->unsigned()->nullable()->after('id');
        });

        Schema::create('page_translations', function (Blueprint $table) {
            createDefaultTranslationsTableFields($table, 'page');
        });

        Schema::table('page_translations', function (Blueprint $table) {
            $table->string('title', 1000)->nullable()->after('page_id');
            $table->string('meta_title', 100)->nullable()->after('title');
            $table->text('meta_description', 200)->nullable()->after('meta_title');
        });

        Schema::create('page_slugs', function (Blueprint $table) {
            createDefaultSlugsTableFields($table, 'page');
        });

        Schema::create('page_revisions', function (Blueprint $table) {
            createDefaultRevisionsTableFields($table, 'page');
        });
    }

    public function down()
    {
        Schema::dropIfExists('page_revisions');
        Schema::dropIfExists('page_translations');
        Schema::dropIfExists('page_slugs');
        Schema::dropIfExists('pages');
    }
}
```

> As the `createDefault...()` functions create the timestamp and soft delete columns, we use an update of the structure to add our columns in a more pleasant order but it's not a requirement, you can define your columns in the create method

We can now run our migration to create the tables for our Module:

```xml
php artisan migrate
```

**/app/Models/Page.php**

```xml
// ...
    protected $fillable = [
        'published',
        'title',
        'meta_title',
        'meta_description',
        'position',
    ];

    public $translatedAttributes = [
        'title',
        'meta_title',
        'meta_description',
        'active',
    ];
// ...
```

> Twill uses Laravel Eloquent mass assignment, so we need to declare all our attributes in the `fillable` property, and the translatable attributes in the `translatedAttributes` property (it's a common mistake to forget to add our attributes or a new one created after as Twill won't trigger an error when you edit your content, but your value will not be saved)

**/resources/views/admin/pages/form.blade.php**

```xml
@extends('twill::layouts.form')

@section('contentFields')
    @formField('block_editor', [
        'withoutSeparator' => true,
        // 'blocks' => []
    ])
@stop

@section('sideFieldsets')
    @formFieldset([ 'id' => 'seo', 'title' => 'SEO'])
        @formField('input', [
            'name' => 'meta_title',
            'label' => 'Title',
            'translated' => true,
            'maxlength' => 100,
        ])

        @formField('input', [
            'name' => 'meta_description',
            'label' => 'Description',
            'translated' => true,
            'maxlength' => 200,
        ])
    @endformFieldset
@stop
```

> In the `contentFields` section, we remove the title and add a `block_editor` field (the `withoutSeparator` option just removes a separator displayed before it on the edit page and the `blocks` option allows to make available a specific list of blocks, otherwise all blocks can be added)

> We create a `sideFieldsets` section where we add our SEO fields

Now we can manage our content in the administration interface. Let's click on the *Add new* button to see a modal asking us the title of our page (the permalink aka slug is automatically generated but you can edit it). You can fill the information for each language and also change the publication status of your content or for each language):

![Content creation - Modal](https://cdn.hashnode.com/res/hashnode/image/upload/v1626447569529/q6LdgkTSX.png align="left")

And here is the form for editing our content:

![Content creation - Form](https://cdn.hashnode.com/res/hashnode/image/upload/v1626448839004/tU22hf5qy.png align="left")

#### Admin Controller customization

Maybe have you seen on the form screenshot an URL under the title. Twill displays a link to our front-end content based on the domain name, the module name and then the slug of our page. This behavior, and many more things (like the columns displayed in the listing, the default order, ...) can be customized in the admin Controller.

Here is some basic customization:

**/app/Http/Controllers/Admin/PageController.php**

```xml
<?php

namespace App\Http\Controllers\Admin;

use A17\Twill\Http\Controllers\Admin\ModuleController;

class PageController extends ModuleController
{
    protected $moduleName = 'pages';

    protected $permalinkBase = '';

    protected $indexOptions = [
        'reorder' => true,
    ];

    protected $indexColumns = [
        'title' => [
            'title' => 'Page',
            'field' => 'title',
        ],
    ];
}
```

> We add an empty `permalinkBase` property to tell Twill our pages will be availble directly under the Web root path.

> We add `indexOptions` array property to allow us to order our pages in the administration listing interface (this property allows you to activate or desactivate many features like creation, publication, duplication, deletion, ...).

> We add `indexColumns` array property (even if it's not necessary here as Twill uses `title` attribute as the default and only column to display). We can later add thumbnails, computed, relationship or presented fields that can be sortable, not visible by default, ...

---

**We now have a basic (for now) but working administration interface to manage content, we will see in later articles how to display it on our front-end with Inertia and get more complex content structure**

---

> We'll do our best to provide source code of the serie on [GitHub](https://github.com/Codivores/tutorial-laravel-twill-inertia-vue3-vite-tailwind)

%[https://github.com/Codivores/tutorial-laravel-twill-inertia-vue3-vite-tailwind]