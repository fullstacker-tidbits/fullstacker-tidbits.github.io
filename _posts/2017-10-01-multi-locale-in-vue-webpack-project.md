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

-----

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

And from here on, all the section titles point to the github commit history.
You are encouraged to read the post with either the code running or the commit diffs.


### [Stitching APIs](https://github.com/fullstacker-tidbits/vue-webpack-multi-locale-demo/commit/664924e580dfdb8ddf8247f0dd9b3db5424e1edb)

Majority of the tweaks are going to be add-ons to the commponent of `Hello.vue`,
to make it display a todo list
(rest assured it's not going to be **yet another Todo MVC** but much much simpler).
Checkout into the commit of `i18n attempt` to see it imports two modules:
`T` for translation,
and `services` for mocked API.
And it now populates its todo list from Vue component's `created` hook which executes on its ... well, creation.

```javascript
// ...
async created() {
  this.todos = await services.todoList();
},
// ...
```

If the `async` keyword bothers you,
you can safely take it as a mechanism to automagically make `Promise` synchronise.
Function `T` (modified from a Stackoverflow answer so long ago that I can't trace its origin) is defined in `locale/index.js`.

```javascript
import en from './en';
const properties = en;

export default function T(key, ...args) {
  return properties[key].replace(
    /{(\d)}/g,
    (match, num) => (typeof args[num] !== 'undefined' ? args[num] : match),
  );
}
```

What it does is simply take a key string,
finds the corresponding template string from a dictionary defined in `locales/{language}.js`,
sub in the rest of the arguments to the template string. 

Things look to be still 'in order' so far.
But the code smell can quickly spread if we imagine the number of strings to translate increases:
We don't want so many fields in `data` just to be used once in our template.
What we look for is something like {% raw %} `<h1>{{ T('title') }}</h1>` {% endraw %}
Can we do that? Sure, by exposing `T` to DOM by adding it in `data` too...

"That's even worse!" You might scream.
Surely it is very dirty exposing a **function** as **data**.
Besides, if majority of the components need to be internationalized,
importing `T` again and again and registering it in `data` is just way too smelly.
The most proper way to this issue is creating a custom plugin and install it on initiating Vue runtime.
But let's not side-track that far into the Vue practices,
but keep our demo simple by using a _stripped_ mechanism of plugin: mixin,
as illustrated in the commit `i18n with mixin`.


### [Mixin](https://github.com/fullstacker-tidbits/vue-webpack-multi-locale-demo/commit/5c9d4396af2df72d86e228030b2d10c1f88db83c)

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


### [New Locale](https://github.com/fullstacker-tidbits/vue-webpack-multi-locale-demo/commit/f3d0a46969ad799b911667d8b8c675e797aa4449)

Now let's add another language! How about Chinese?

You don't have to worry about `cn.js` (nor do I think you will at all),
but just take a look at the changed history and things should be rather clear.
The locale entry now looks like:

```javascript
import cn from './cn';
import en from './en';

const properties = { cn, en }[process.env.LOCALE];
```

And in `config/prod.js`, we added an environment variable

```javascript
LOCALE: JSON.stringify(process.env.LOCALE)
```

Easy it may appear, there are two major gotchas in this line that cost me hours of debugging:

1. `prod.js` later serves as the basis of environment variable dictionary and it **overwrites all other environment variables**
2. `JSON.stringify` is needed as `process.env` variables are directly **substituted** just like macro in C

I think overwriting all environment variables could be out of safety concern;
but honestly this is quite counter-intuitive IMHO.
My first impression over a glimpse of the folder structure would be that
the environment variables are **added on top of existing ones**,
never anticipate that as an overwrite.
And the second gotcha pull so much of my hair out due to the mysterious
"Uncaught ReferenceError: cn is not defined" error.

In this commit, just export `LOCALE` as either `cn` or `en`, `yarn build` it,
and serve `dist` folder in Nginx/Apache/Caddy/Python http.server/Node http,
it should behave just as you would expect it to be.


### [Dark Corner...](https://github.com/fullstacker-tidbits/vue-webpack-multi-locale-demo/commit/577817e8290a5a35937302891e3520363c29120c)

Up until now everything is indeed quite decent,
and I did celebrate in the project at work after I've put all that was introduced so far.

However, there is a very sneaky corner that I overlooked:
After I run `yarn build` over the project and deployed my website,
the built application seems slightly larger than usual...

The issue is that even I needed only one language support in a build,
Webpack puts all languages into my compiled app.whateverhash.js.

This can be harder to spot in this demo as the locale files are so tiny,
and it would cause hardly any serious harm anyway;
but in our case, with so many languages to support and so many text to substitute,
accidentally including all the translations to the production build is no easy-peasy.
I won't paste the uglified code here but you can quickly search it to realize the redundancy.

So what now?

Obviously we need a way to conditionally import the locale.
But the issue is that `import` [doesn't support conditional import](https://stackoverflow.com/questions/36367532/how-can-i-conditionally-import-an-es6-module).
Fortunately we can fallback to Node's `require` for this need.
Just change everything in `locales/index.js` before `export` into the following 2 lines:

```javascript
// eslint-disable-next-line
const properties = require(`./${process.env.LOCALE}`).default;
```

ESLint ignoring line is needed to appease my Linter configured user Airbnb standard.
That line aside, we essentially required only one module per specified by `LOCALE`.
Therefore, with this change,
`dev` and `build` and `{whatever-daemon} serve` as last commit,
and be glad at a production build not polluted with unnecessary bloat.


### [Building Meta](#)
WIP


### [Custom Styling](#)
WIP