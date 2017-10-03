---
layout: post
title: Multi-Locale in Vue Webpack Project
---

Developing frontend applications with [Vue.js](https://github.com/vuejs/vue) is a fairly hassle-free experience,
especially when using [vue-cli](https://github.com/vuejs/vue-cli) to bootstrap the structure.
When I work on my project,
my usual choice is simply the default [webpack template](https://github.com/vuejs-templates/webpack) provided officially.
The template has all things for local development nicely put together with a single command of `yarn dev`,
and for deployment `yarn build`.

Recently we came across a case where the same copy of source code should be compiled into distributions of different languages.
We didn't choose [vue-i18n](https://github.com/kazupon/vue-i18n) even though it is also an awesome tool,
mainly because we are building the website under different domains,
instead of one domain serving different languages by different folders.

Here is what we did:
We used the default vue webpack setup provided by vue-cli,
and slowly adding to it the customed i18n-ish features we need.
And this post will cover the features we built point by point.

Since our website is an internal product with confidentiality and dirty bussiness logic that hinders understanding,
I created a [project](https://github.com/fullstacker-tidbits/vue-webpack-multi-locale-demo) containing only the bare minimal to illustrate what we learned.
It also has cleaner commits to mark the evolution of the project,
and yarn-lock to ensure the same version of libraries are installed for reproducible local runs.
Each section of this post will explain a commit to the demo project,
and the title is wired with the link that leads to the commit in GitHub.

-----

_I'll try to make the tutorial as friendly as I can with my limited command of English,
but some basic knowledge of Vue and Webpack are preferred;
even better if you have experience with the structure vue-cli scaffolds.
Vue documents its webpack project structure nicely [here](https://vuejs-templates.github.io/webpack/)_


### [Very Beginning](https://github.com/fullstacker-tidbits/vue-webpack-multi-locale-demo/commit/16f1da58ad0c9fbd1cf63af7a82cc15f6b95a751)

If you clone the repo and checkout to the very first commit `vanilla`,
you'll see almost nothing deviates from the originally generated template
(dubbed vanilla for a cause XD).
Just run

```bash
$ yarn install
$ yarn dev
```

to see a browser tab pops with a page title and a caption.


### [Stitching API](https://github.com/fullstacker-tidbits/vue-webpack-multi-locale-demo/commit/664924e580dfdb8ddf8247f0dd9b3db5424e1edb)

Majority of the tweaks are going to be enhancements to the commponent of `Hello.vue`,
to make it display a todo list
(rest assured it's not going to be **yet another Todo MVC** but much much simpler).
Checkout the commit of `i18n attempt` to see it imports two modules:
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
you can safely take it as a mechanism to automagically make `Promise` synchronous.
Function `T` (modified from a Stackoverflow answer so long ago that I can't trace its origin) is defined in `locales/index.js`.

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
Can we do that? Sure, by adding `T` to `data` so that it's exposed to DOM...

"That's even worse!" You might scream.
Surely it's very dirty exposing a **function** as **data**.
Besides, if majority of the components need to be internationalized,
importing `T` again and again and registering it in `data` is just way too smelly.
The most proper way to this issue is creating a custom plugin and install it on initializing Vue runtime.
But let's not side-track that far into the Vue practices,
but keep our demo simple by using a _stripped_ mechanism of plugin: mixin,
as illustrated in the commit `i18n with mixin`...


### [Mixin](https://github.com/fullstacker-tidbits/vue-webpack-multi-locale-demo/commit/5c9d4396af2df72d86e228030b2d10c1f88db83c)

What mixin does is defining a set of behaviors or attributes common to the components that register it.
Vue has an [intuitive way to merge duplcated attributes](https://vuejs.org/v2/guide/mixins.html#Option-Merging),
and you can even define your own merge rules if they collide in a too unwieldy manner XD.

For our case we just want to have a translation function registered for _all_ componets,
so we directly modify the behavior of the imported `Vue` instance.
Also per Vue community's convention,
such extended functionalities should be prefixed with a dollor sign.
So let's just conform to it:

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

1. `prod.js` later serves as the basis of environment variable dictionary,
and it **discards all other environment variables**
2. `JSON.stringify` is needed as `process.env` variables are directly **substituted** just like macro in C

I think throwing all environment variables could be out of a safety concern;
but IMHO this is quite counter-intuitive.
My first impression over a glimpse of the folder structure would be that
the environment variables are **added on top of existing ones**,
never anticipating it to discard all the unspecified.
And the second gotcha pull so much of my hair out due to the mysterious
"Uncaught ReferenceError: cn is not defined" error.

In this commit, just export `LOCALE` as either `cn` or `en`, `yarn build` it,
and serve `dist` folder in Nginx/Apache/Caddy/Python http.server/Node http,
it should behave just as you would expect it to do.


### [Dark Corner...](https://github.com/fullstacker-tidbits/vue-webpack-multi-locale-demo/commit/577817e8290a5a35937302891e3520363c29120c)

Up until now everything is indeed quite decent,
and I did celebrate in the project at work after I put all that was introduced so far.

However, there is a very sneaky corner that I overlooked:
After I ran `yarn build` over the project and deployed my website,
the built application seemed slightly larger than usual...

The issue was that even I needed only one language support in a build,
Webpack put all languages into my compiled `app.whateverhash.js`.

This can be harder to spot in this demo as the locale files are so tiny,
and it would cause hardly any serious harm anyway;
but in our case, with so many languages to support and so many text to substitute,
accidentally including all the translations to the production build should not be taken easy-peasy.
I won't paste the uglified code here but you can quickly search it to realize the redundancy.

So what now?

Obviously we need a way to conditionally import the locale.
But the issue is that `import` [can't be done conditionally](https://stackoverflow.com/questions/36367532/how-can-i-conditionally-import-an-es6-module).
Fortunately we can fallback to Node's `require` for this need.
Just change everything in `locales/index.js` before `export` line into the following:

```javascript
// eslint-disable-next-line
const properties = require(`./${process.env.LOCALE}`).default;
```

ESLint line ignoring is needed to appease my Linter configured using Airbnb standard.
That line aside, we essentially required only one module per specified by `LOCALE`.
Therefore, with this change,
`dev` and `build` and `{whatever-daemon} serve` as last commit,
and be glad at a production build not polluted with unnecessary bloat.


### [Building Meta](https://github.com/fullstacker-tidbits/vue-webpack-multi-locale-demo/commit/7e0b564c439bd357cba6ead8c417a3d366a3d3b2)

The core features are roughly complete with all the steps so far,
but there is one thing important that all PM would require - customized meta header for SEO.
It's just not acceptable to have English audiences reading Chinese meta in search result and vice versa.

So now suppose `title` tag and `og:title` meta tag should take value from `title` of `{locale}.js`.
How do we cope with it?
An easy _libraried_ solution is to use some `*-meta` package and it does work good for Google and Bing.
The only trouble is that those are pretty much the only two search engines execute synchronous JS code,
and for the case of sharing to Facebook,
the `index.html` page would be parsed as it is,
resulting in an empyt page no room for customizing the behavior later on.

Yet luckily, this is still a standard requirement that Googling gives almost direct answer:
Customize the entry of `index.html` with [ejs](http://ejs.co),
so that it works with parameterized generation;
and parameters are slotted in from [html-webpack-plugin](https://github.com/jantimon/html-webpack-plugin) config.

Another cool thing is that with the current setup,
no extra packages are needed - `ejs` has is installed with `webpack-bundle-analyzer`.

Below is the new head tag in `index.ejs`, renamed from `index.html`.

```handlebars
<head>
  <meta charset="utf-8">
  <title><%= htmlWebpackPlugin.options.title %></title>
  <meta property="og:title" content="<%= htmlWebpackPlugin.options.title %>" />
</head>
```

With the entry renamed of its suffix,
naturally we should update the config files too:

```javascript
// webpack.dev.conf.js L28
new HtmlWebpackPlugin({
  filename: 'index.html',
  template: 'index.ejs',  // <- here
  inject: true
})

// webpack.prod.conf.js L52
new HtmlWebpackPlugin({
  filename: config.build.index,
  template: 'index.ejs',  // <- here
  inject: true
  // ... some more conf continues
```

Now the dev page is populated with a value for title - `Webpack App`.
Honestly I have no clue where it comes but let's just ignore and put in the value we need.
So we happily add `var T = require('../src/locales')` at the top of two webpack config files modified above,
and `title: T('title)` in the config for `HtmlWebpackPlugin`, and restart yarn dev server ...
then it screamed and exited with an error saying `export` in `locales/index.js` is unexpected.

Experienced Node & JS developers may quickly identify the issue of mixing two styles of module standards in our locale files:
CommonJS and ES6.
The dev server and building process are Node based (CommonJS), but `import / export` is from ES6 standard.
But fortunately an environment supporting ES6 module should usually support CommonJS too.
So let's just have the entire `locales` module conform to CommonJS
by changing all `export default ...` to `module.exports = ...`,
also remove the `.default` from the requiring of individual locale.


### [Custom Styling](https://github.com/fullstacker-tidbits/vue-webpack-multi-locale-demo/commit/991b9fdfdbdc10cc6e65295357cdf2d05b5491ab)

This section is more of a bonus feature that we built.
Our regional PM reported that some languages don't render nicely under our universal font setting,
and suggested us of corresponding "better" fonts to change.
For our example project, without loss of generality,
let's just imagine we want to change Chinese compile to be rendered in dark gray,
and the English to black.

Following the same rationale we have been applying,
we create two stylesheets in `assets` folder to define the default color of body as follows:

```css
/* en.css */
body {
  color: black;
}

/* cn.css */
body {
  color: darkgray;
}
```

And to apply the style conditionally to our environment variable,
we `require` the corresponding sheet in global entry...

```javascript
// main.js L5

// eslint-disable-next-line
require(`./assets/${process.env.LOCALE}.css`);
```

And sure enough, it's working as we expect!

And stop the dev server, export another locale value,
restart the dev server again ... to see it works too!