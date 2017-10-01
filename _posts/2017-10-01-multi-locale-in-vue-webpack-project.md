---
layout: post
title: Multi-Locale in Vue Webpack Project
---

Developing frontend applications with [Vue.js](https://github.com/vuejs/vue) is a fairly hassle-free experience,
especially when using [vue-cli](https://github.com/vuejs/vue-cli) to bootstrap the structure.
Our usually project choice is the default [webpack template](https://github.com/vuejs-templates/webpack) provided officially.
The template has all things for local development nicely put together with a single command of `yarn dev`,
and for deployment `yarn build`.

Recently we encountered a case where the same copy of source code should be compileed into distributions of different languages.
We didn't choose [vue-i18n](https://github.com/kazupon/vue-i18n) even though it is also an awesome tool,
mainly because we are building the website under different domains,
instead of one domain serving different folders.

Here is what we did:
We use the default vue webpack setup provided by vue-cli,
and slowly adding on to it the customed i18n-ish features we need.
And this post will cover the features we built point by point.
Since our website is an internal product with confidentiality and dirty bussiness logic that hinders understanding,
I created a [project](https://github.com/fullstacker-tidbits/vue-webpack-multi-locale-demo) containing only the bare minimal to illustrate what we learned,
and it also has cleaner commits (and maybe tags later on) to mark the evolving of the project,
and yarn-lock to ensure the same version of libraries are installed for reproducible local runs.

_I'll try to make the tutorial as friendly as I can with my limited command of English,
but some basic knowledge of Vue and Webpack are preferred;
even better if you have experience with the structure vue-cli scaffolds.
Vue documents its webpack project structure nicely [here](https://vuejs-templates.github.io/webpack/)_

If you clone the repo and checkout to the very first commit `vanilla`,
you'll see almost nothing deviates from the originally generated template
(dubbed vanilla for a cause XD).
Just run

```bash
$ yarn install
$ yarn dev
```

to see a browser tab pops with a page title and a caption.


## [Stitching APIs](https://github.com/fullstacker-tidbits/vue-webpack-multi-locale-demo/commit/664924e580dfdb8ddf8247f0dd9b3db5424e1edb)

Majority of the tweaks are going to be add-ons to the commponent of `Hello.vue`,
to make it display a todo list
(rest assured it's not going to be **yet another Todo MVC** but much much simpler).
Checkout into the commit of `i18n attempt` to see it imports two modules:
`T` for translation,
and `services` for mocked API.
And it now populates its todo list from Vue component's `created` hook which executes on its ... well, creation. 
If the `async` keyword bothers you,
you can safely take it as a mechanism to automagically make `Promise` synchronise.
Function `T` (modified from a Stackoverflow answer so long ago that I can't trace its origin) is defined in `locale/index.js`.
What it does is simply take a key string,
finds the corresponding template string from a dictionary defined in `locales/{language}.js`,
sub in the rest of the arguments to the template string. 

Things look to be still 'in order' so far.
But the code smell can quickly spread if we imagine the number of strings to translate increases:
We don't want so many fields in `data` just to be used once in our template.
What we look for is something like this {% raw %} `<h1>{{ T('title') }}</h1>` {% endraw %}
Can we do that? Sure, by exposing `T` to DOM by adding it in `data` too...

"That's even worse!" You might scream.
Surely it is very dirty exposing a **function** as **data**.
Besides, if majority of the components need to be internationalized,
importing `T` again and again and registering it in `data` is just way too smelly.
The most proper way to this issue is creating a custom plugin and install it on initiating Vue runtime.
But let's not side-track that far into the Vue practices,
but keep our demo simple by using a _stripped_ mechanism of plugin: mixin,
as illustrated in the commit `i18n with mixin`.


## [Mixin](https://github.com/fullstacker-tidbits/vue-webpack-multi-locale-demo/commit/5c9d4396af2df72d86e228030b2d10c1f88db83c)

What mixin does is either defining a set of behaviors or attributes common to the components that register it.
Vue has an [intuitive way to merge duplcated attributes](https://vuejs.org/v2/guide/mixins.html#Option-Merging),
and you can even define your own merge rules if they collide in a too unwieldy manner XD.
For our case we just want to have a translation function registered for all componets.
Also per Vue community's convention,
such extended functionalities should be prefixed with a dollor sign,
let's just conform to it:

```javascript
import T from './locales';

Vue.mixin({
  created() {
    this.$T = T;
  },
});
```

And now we can happily strip off all the imports of `T` in our components,
and discard all the used-once-only fields in `data` ㋡


## [New Languages](#)


## [Building Meta](#)


## [Custom Styling](#)