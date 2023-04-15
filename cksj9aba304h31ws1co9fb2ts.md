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
> \--hasNesting (-N), to enable nested items in the module listing  
> \--parentModel=, to generate the route for a nested module. See ( see Nested Module)
> 
> \--bladeForm, to generate a Blade form instead of using the new Form builder (more info [here](https://twillcms.com/blog/twill-3-introducing-oop-builders.html))

Twill is amazing for this as you can choose the features you want to have available for your content, keeping simple content simple. All work with PHP Traits and additional classes, so if you forget an option, you can add it later based on sample codes.

### Let's create our first Module to handle static pages

We will call this Module *contentPage* and we will need Blocks (to have flexible content), Translations, Slugs, Medias, Position (to order them in the administration) and Revisions (we will not attach Files directly but maybe through Blocks):

```bash
php artisan twill:make:module pageContent -BTSMPR 

# If you want to handle the form through a Blade template, you can add the --bladeForm option (you still can create the file manually afterward
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
    
* Customize the fields of the Model
    
* Create the form accordingly (Blade components or OOP builder)
    
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

Since Twill 3, there is a new way to manage navigation, registering it in the AppServiceProvider (you still can use the legacy method defining you navigation in a `config/twill-navigation.php` file). More info in the [official documentation](https://twillcms.com/docs/getting-started/navigation.html)

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
                ->title(Str::ucfirst(__('pages')))
                ->forModule('pageContents')
        );
    }
```

To see our Module administration interface without triggering an error, we need to execute the migration via `php artisan migrate`. We will do it later as we want to customize the attributes of our model, but here is an example of what you will see:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1681568293228/dcfda5dc-33dd-4767-9f11-e869aed065f5.png align="center")

#### Navigation reorganization

The default configuration adds our Module to the primary level of the navigation. As we want to organize our navigation, we will modify this configuration to access our Module at a secondary level of a global `Content` entry (Twill allows you to have up to 3 levels for your navigation):

**/routes/twill.php**

```php
<?php

use A17\Twill\Facades\TwillRoutes;
use Illuminate\Support\Facades\Route;

Route::group(['prefix' => 'content'], function () {
    TwillRoutes::module('pageContents');
});
```

**/app/Providers/AppServiceProvider.php**

```php
    public function boot(): void
    {
        TwillNavigation::addLink(
            NavigationLink::make()
                ->title(Str::ucfirst(__('content')))
                ->forModule('pageContents')
                ->doNotAddSelfAsFirstChild()
                ->setChildren([
                    NavigationLink::make()
                        ->title(Str::ucfirst(__('pages')))
                        ->forModule('pageContents')
                ]),
        );
    }
```

**What it does**

For the route, using standard Laravel routing, we encapsulate our Module routes definition in a group with the `content` prefix. As Twill routes already add `twill` prefix, our Module routes names will start with `twill.content.pageContents`

For the Twill navigation, we encapsulate our module as a child of a primary `content` navigation link which has the following attributes:

* title(): the text displayed in the navigation
    
* forModule(): the module that will be displayed on click, for now, we want the index of our Module
    
* doNotAddSelfAsFirstChild(): without this method, we would have 2 entries on the secondary level: `Content` and `Pages`
    
* setChildren(): the list of the navigation links for the secondary level
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1681569399937/97f97a47-855b-4a9c-b28e-d9cf9a1f47df.png align="center")

#### Model customization

The migration, model and form files created by the generator are a template with default fields.

In our case, we want:

* a title that can be translated
    
* meta title and description that can be translated
    
* a position to sort manually our pages in the listing of the administration interface
    
* a block editor for all the content (we will focus on this part in a later article)
    

Here is what the final files look like.

**/database/migrations/...\_create\_page\_contents\_tables.php**

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreatePageContentsTables extends Migration
{
    public function up()
    {
        Schema::create('page_contents', function (Blueprint $table) {
            createDefaultTableFields($table);
        });

        Schema::table('page_contents', function (Blueprint $table) {
            $table->after('id', function ($table) {
                $table->integer('position')->unsigned()->nullable();
            });
        });

        Schema::create('page_content_translations', function (Blueprint $table) {
            createDefaultTranslationsTableFields($table, 'page_content');
        });

        Schema::table('page_content_translations', function (Blueprint $table) {
            $table->after('page_content_id', function ($table) {
                $table->string('title', 200)->nullable();
                $table->string('meta_title', 100)->nullable();
                $table->text('meta_description', 200)->nullable();
            });
        });

        Schema::create('page_content_slugs', function (Blueprint $table) {
            createDefaultSlugsTableFields($table, 'page_content');
        });

        Schema::create('page_content_revisions', function (Blueprint $table) {
            createDefaultRevisionsTableFields($table, 'page_content');
        });
    }

    public function down()
    {
        Schema::dropIfExists('page_content_revisions');
        Schema::dropIfExists('page_content_translations');
        Schema::dropIfExists('page_content_slugs');
        Schema::dropIfExists('page_contents');
    }
}
```

> As the `createDefault...()` functions create the timestamp and soft delete columns, we use an update of the structure to add our columns in a more pleasant order but it's not a requirement, you can define your columns in the create method

We can now run our migration to create the tables for our Module:

```bash
php artisan migrate
```

**/app/Models/PageContent.php**

```php
// ...
    protected $fillable = [
        'title',
        'meta_title',
        'meta_description',
        'published',
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

#### Form creation

In Twill 2, forms were Blade components, Twill 3 introduces a Form builder allowing you to define the form in PHP in the module controller.

Blade components give you more flexibility for very complex forms or if you want to integrate custom HTML. For standard forms, the Form builder seems to do the job. We will see both ways.

#### Form through Blade components

If you set the `--bladeForm` option on the module creation, the file already exists, if not you have to create it manually.

**/resources/views/twill/pageContents/form.blade.php**

```scss
@extends('twill::layouts.form')

@section('contentFields')
    
    <x-twill::block-editor
        :withoutSeparator="true"
    />

@stop

@section('sideFieldsets')
    @formFieldset([ 'id' => 'seo', 'title' => 'Référencement'])

        <x-twill::input
            label="Title"
            name="meta_title"
            :translated="true"
            :maxlength="100"
        />

        <x-twill::input
            label="Description"
            name="meta_description"
            :translated="true"
            :maxlength="200"
        />

    @endformFieldset
@stop
```

> In the `contentFields` section, we remove the title and add a `block_editor` field (the `withoutSeparator` option just removes a separator displayed before it on the edit page)

> We create a `sideFieldsets` section where we add our SEO fields

#### Form through Form builder

The configuration is made directly in the module controller.

**/app/Http/Controllers/Admin/PageContentController.php**

```php
<?php

namespace App\Http\Controllers\Twill;

use A17\Twill\Http\Controllers\Admin\ModuleController as BaseModuleController;
use A17\Twill\Models\Contracts\TwillModelContract;
use A17\Twill\Services\Forms\Fields\BlockEditor;
use A17\Twill\Services\Forms\Fields\Input;
use A17\Twill\Services\Forms\Fieldset;
use A17\Twill\Services\Forms\Form;

class PageContentController extends BaseModuleController
{
    public function getForm(TwillModelContract $model): Form
    {
        $form = parent::getForm($model);

        $form->add(
            BlockEditor::make()
                ->withoutSeparator()
        );

        return $form;
    }

    public function getSideFieldsets(TwillModelContract $model): Form
    {
        $form = parent::getSideFieldsets($model);

        $form->addFieldset(
            Fieldset::make()
                ->title('SEO')
                ->id('seo')
                ->fields([
                    Input::make()
                        ->name('meta_title')
                        ->label('Title')
                        ->translatable()
                        ->maxLength(100),

                    Input::make()
                        ->name('meta_description')
                        ->label('Description')
                        ->translatable()
                        ->maxLength(200),
                ])
        );

        return $form;
    }
}
```

> The `getForm()` method defines the fields or fieldsets in the left column
> 
> The `getSideFieldsets()` method defines the additional fields or fieldsets in the side/right column

#### Form in action

Now we can manage our content in the administration interface. Let's click on the *Add new* button to see a modal asking us for the title of our page (the permalink aka slug is automatically generated but you can edit it). You can fill in the information for each language and also change the publication status of your content globally and for each language):

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1681569793440/ec3f55d8-ee83-4be3-8372-a980e6219252.png align="center")

And here is the form for editing our content:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1681570266696/8d092b35-1649-4982-8e90-3e855fa7526d.png align="center")

#### Admin Controller customization

Maybe have you seen on the form screenshot an URL under the title. Twill displays a link to our front-end content based on the domain name, the module name and then the slug of our page. This behavior, and many more things (like the columns displayed in the listing, the default order, ...) can be customized in the admin Controller.

Here is some basic customization:

**/app/Http/Controllers/Admin/PageContentController.php**

```php
<?php

namespace App\Http\Controllers\Twill;

use A17\Twill\Http\Controllers\Admin\ModuleController as BaseModuleController;

class PageContentController extends BaseModuleController
{
    protected $moduleName = 'pageContents';

    protected function setUpController(): void
    {
        $this->setPermalinkBase('');
        $this->enableReorder();
    }
}
```

> We set an empty `permalinkBase` property to tell Twill our pages will be available directly under the Web root path.

> We enable reordering

More info in the [official documentation](https://twillcms.com/docs/modules/controllers.html#content-controller-setup)

---

**We now have a basic (for now) but working administration interface to manage content, we will see in later articles how to display it on our front-end with Inertia and handle more complex content structure**

---

> We'll do our best to provide source code of the serie on [GitHub](https://github.com/Codivores/tutorial-laravel-twill-inertia-vue3-vite-tailwind)

%[https://github.com/Codivores/tutorial-laravel-twill-inertia-vue3-vite-tailwind]