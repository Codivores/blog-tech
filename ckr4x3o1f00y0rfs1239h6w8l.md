## The Why and the What

> We are not regular writers and even less in english, so please accept our apologies in advance! (and we encourage your feedback)

After some years building Laravel based applications and using more and more Vue.js for the front-end, we are now updating our stack for websites developments, trying to bring together the best tools and practices to provide the best experience for our clients, our clients' clients and of course us as developers.

For this first article of the serie, we will introduce the stack:

- Laravel
- Twill
- Inertia
- Vue 3 with Composition API and TypeScript
- Vite
- Tailwind


## Laravel

> Website: [https://laravel.com](https://laravel.com)

> Documentation: [https://laravel.com/docs/8.x](https://laravel.com/docs/8.x)

> Git : [https://github.com/laravel/laravel](https://github.com/laravel/laravel)

We are mainly PHP developers and we cannot imagine using an other framework. But if you are here, we guess your share the same feelings (let us know if not, we can provide some insights).


## Twill

> Website: [https://twill.io](https://twill.io)

> Documentation: [https://twill.io/docs](https://twill.io/docs)

> Git : [https://github.com/area17/twill](https://github.com/area17/twill)

Despite the huge Laravel ecosystem, it's not easy to find a package that offers CMS features with a good UX for our clients.
We use Twill for 2 years now and with a dozen of projects, we (and our clients) are very satisfied with the functionalities and flexibility this open source package offers:

- No front-end, that could scare a little bit at first, but it allows you to use it with Laravel or as a headless CMS
- CLI to create needed files to handle new content (model, admin controller, form, routing, admin navigation)
- Multi-language in a very easy way to work with
- Content scheduling (draft, start and end dates), responsive preview, revisions
- Media library with built-in Glide integration (smart cropping and profiles)
- Block editor, repeaters for quick and flexible content creation
- Dashboard you can customize


## Inertia

> Website / Documentation : [https://inertiajs.com](https://inertiajs.com)

> Git : [https://github.com/inertiajs](https://github.com/inertiajs)

After 3 small projects with Inertia, we are now confident this (hard to find a good definition) back-end/front-end wrapper is a game changer, allowing to build our back-end as a standard Laravel application and make it a SPA with Vue.js page components without worrying about Ajax calls, shared data, ...


## Vue 3 / TypeScript

> Website: [https://v3.vuejs.org](https://v3.vuejs.org)

> Documentation: [https://v3.vuejs.org/guide](https://v3.vuejs.org/guide)

> Git : [https://github.com/vuejs/vue-next](https://github.com/vuejs/vue-next)

After some Vue 2 / Nuxt projects, it's time to move forward and go for Vue 3, and if we are going to use it, we should do it with the best approach, meaning 

- Composition API: better and cleaner code definition, reuse and sharing
- TypeScript: good support for Composition API, and also better and cleaner code


## Vite

> Website: [https://vitejs.dev](https://vitejs.dev)

> Documentation: [https://vitejs.dev/guide](https://vitejs.dev/guide)

> Git : [https://github.com/vitejs/vite](https://github.com/vitejs/vite)

Laravel comes natively with [Laravel Mix](https://laravel.com/docs/8.x/mix) based on [Webpack](https://webpack.js.org) and we used it a lot. But we are not big fans of Webpack (surely a lack of investment on our part), and on big projects, we got some big build times, slow Hot Module Replacement and big outputs.

And here comes Vite that seems to provide faster development experience with Hot Module Replacement and Rollup for production builds. So, we decided to give it a try!


## Tailwind

> Website: [https://tailwindcss.com](https://tailwindcss.com)

> Documentation: [https://tailwindcss.com/docs](https://tailwindcss.com/docs)

> Git : [https://github.com/tailwindlabs/tailwindcss](https://github.com/tailwindlabs/tailwindcss)

Tailwind is a utility-first CSS framework. There are Pros and Cons and we will not be arguing that here. We use it for many years now, it continues to improve (responsive, dark mode, small size production builds purging unused CSS with [PostCSS](https://postcss.org)), so why change!


---

> We'll do our best to provide source code of the serie on [GitHub](https://github.com/Codivores/tutorial-laravel-twill-inertia-vue3-vite-tailwind) 

%[https://github.com/Codivores/tutorial-laravel-twill-inertia-vue3-vite-tailwind]