---
layout: post
title: Multi-Locale in Vue Webpack Project
---

Developing frontend applications with [Vue.js](https://github.com/vuejs/vue) is mostly a painless experience,
especially when using [vue-cli](https://github.com/vuejs/vue-cli) to bootstrap the structure.
Our usually project choice is the default [webpack template](https://github.com/vuejs-templates/webpack) provided officially.
The template has all things for local development nicely put together with a single command of `yarn dev`,
and for deployment `yarn build`.

Recently we encountered a case whereby the same copy of source code should be compileed into multiple distributions of different languages.
We didn't choose [vue-i18n](https://github.com/kazupon/vue-i18n) even though it is also an awesome tool,
mainly because we are building the website under different domains,
instead of one domain serving different folders.

So here is what we did:
We use the default vue webpack setup provided by vue-cli,
and slowly adding on to it the customed i18n-ish features we need.
And this post will cover the features we built point by point.
Since our website is an internal product with confidentiality and dirty bussiness logic that hinders understanding,
I created a [project](https://github.com/fullstacker-tidbits/vue-webpack-multi-locale-demo) containing only the bare minimal to illustrate what we learned,
and it also has cleaner commits (and maybe tags later on) to mark the evolving of the project.