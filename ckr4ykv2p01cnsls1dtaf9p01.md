---
title: "Set up Laravel with Inertia"
seoTitle: "Set up Laravel with Inertia"
seoDescription: "First steps of the project, install Laravel and Inertia server side"
datePublished: Thu Jul 15 2021 13:35:02 GMT+0000 (Coordinated Universal Time)
cuid: ckr4ykv2p01cnsls1dtaf9p01
slug: ltivt-1-laravel-with-inertia
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1627664994987/ZCHKnMW8s.png
tags: laravel, php, inertiajs

---

> **Updated version for Laravel 10 / Twill 3 on Apr 13, 2023**

## Install Laravel

If you've never worked on a Laravel project, you can find all the ways to install it and start a new project on the [official documentation](https://laravel.com/docs/10.x/installation#your-first-laravel-project), and there are many good tutorials you can find depending on your development environment.

Here we work with Docker containers and use Composer to install Laravel in our project folder.

```php
composer create-project laravel/laravel app
```

That will create the base structure of the project, install all the dependencies and set the default configuration (we generally make a first Git commit after the fresh install).

## Install Inertia - Server Side

### Dependencies installation

Inertia comes as a Laravel package that can be installed via Composer. You can refer to the official setup on the [official documentation](https://inertiajs.com/server-side-setup).

```plaintext
composer require inertiajs/inertia-laravel
```

### Middleware set up

To have Inertia intercept the application requests and do its magic (shared data, assets versioning), we need to add a middleware.

This can be done by executing an artisan command which will publish the `HandleInertiaRequests` middleware in our middleware folder `/app/HTTP/Middleware`.

```plaintext
php artisan inertia:middleware
```

Once created, we need to register it manually in our `/app/Http/Kernel.php` file, in the last position of our web route middleware groups.

```plaintext
    /**
     * The application's route middleware groups.
     *
     * @var array
     */
    protected $middlewareGroups = [
        'web' => [
            // ...

            \App\Http\Middleware\HandleInertiaRequests::class,
        ],
    ];
```

*Of course, we need more steps to see Inertia in action, but we now have it ready to handle our application requests.*

---

> We'll do our best to provide source code of the serie on [GitHub](https://github.com/Codivores/tutorial-laravel-twill-inertia-vue3-vite-tailwind)

%[https://github.com/Codivores/tutorial-laravel-twill-inertia-vue3-vite-tailwind]