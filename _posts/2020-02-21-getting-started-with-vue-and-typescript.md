---
layout: post
title: "Getting Started with VueJs 2 and Typescript"
date: 2020-02-21
category: Programming
tags: [vue,typescript]
comments: true
featured: true
---
This is a daunting first post in the [series](http://code.rawlinson.us/2020/02/introducing-a-series-on-vue-js.html) but, I made the promise so it's time to get started.

This is written from the perspective of someone who hasn't done anything with Vue.  I won't be comparing things with how you'd do it in Javascript.  I literally just dove in with no idea what I was doing and this is how I came out the otherside.

I can't recommend starting with the [official VUE 2 docs](https://vuejs.org/v2/guide/) enough. That's where I started and where I'm starting again as I write this post.

## Disclaimer

I'm probably doing stuff wrong.  I'm probably using some anti-patterns.  I'm not an expert on Vue or Typescript.  Things "work" but if you see things that can be done better - LET ME KNOW!  Please!!!

## The Project

As I go through these posts I'm going to be building an actual project.  I'll be doing very non-atomic commits in git - meaning that I won't commit each little change; instead I'll commit at key points in the series and link to those commits so you can see the code at that point.

This project is going to be something I've built before - about 15 years ago - in Coldfusion!  It's called lootly.  Basically, it lets you track some things you own; like books, cds, video games, tools, etc, and lend them to friends.  We're not going to actually add user authentication or some of the lending/borrowing functionality.  Instead, we'll just have the one user (me) at the moment and the inventory capture.

So our models will be, at a mininum; account, book, videogame, boardgame, tool, movie, album, cd when we are done.  All of these except account will implement some common interfaces (ILoanable for instance) and maybe some others.  I didn't have some of these cool programming tricks available to me with Coldfusion 4.7 or whatever it was back then so I'll be figuring out some of this as I go.

## Tools, Tools, and More Tools

Before we get into using Vue - lets talk about tooling.  I am using Visual Studio Code as my editor for this project.  The Vue tools for it are pretty solid and will help us avoid all sorts of little gotchas!

### Visual Studio Code Extensions

* [ESLint](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint) - honestly, if you're touching any kind of javascript use ESLint.
* [Vetur](https://marketplace.visualstudio.com/items?itemName=octref.vetur) - Vue Tooling for VS Code - probably does a lot I don't know about. But it alerts you to LOTS of problems.
* [vue](https://marketplace.visualstudio.com/items?itemName=jcbuisson.vue) - a syntax highlighting extension

Later, when we get into testing, you'll reallly appreciate extensions such as:

* [JestRunner](https://marketplace.visualstudio.com/items?itemName=firsttris.vscode-jest-runner)

### Workspace Settings

I really like lint. It keeps things "clean".  Sure, there can be some really obnoxious linting rules but for the most part, what we'll be using, are pretty handy.  However, the rules can still get annoying. Fortunately, with the eslint extension we can fix a lot of things automatically.  If you're using VSCode you can go to `File->Settings->Workspace->settings.json` and add the following.  This is my entire settings.json file for the workspace.

```js
{
  "eslint.validate" : ["vue","javascript"],
  "eslint.options": {
    "extensions": [".js",".jsx",".ts",".tsx",".vue"]
  },
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  },
  "eslint.lintTask.enable": true,
  "eslint.format.enable": true,
  "eslint.workingDirectories": [
    "./src"
  ],
  "vetur.validation.template": false
}
```

## Install the Vue Cli

Install the [Vue CLI](https://cli.vuejs.org/guide/installation.html) - it will make your life eaiser in the long run.  Trust me.  I tried the `npm install vue` option but, quite frankly I floundered a round a bit trying to translate my outdated Angular 1.x (that I hadn't touched for a year) into Vue setup/build/packaging and I gave up.   The Vue Cli makes everything pretty easy.  This [link](https://cli.vuejs.org/guide/installation.html) takes you right to the installation instructions.

Pay attention; even though I'm working with VUE 2 - the Vue Cli version I'm operating with is 4.1.1 - version numbers for Vue aren't consistent across all "vue related things" - that makes sense if you think about it but I can imagine it might confuse some people.

There is also a ui for all of this - `vue ui` which I think just puts a pretty wrapper on the vue cli.   I'm not going to go through all of the UI stuff.

## Lets Create a Project

For these posts I'm going to actually create a project as I go.  I'm going to call it lootly and I'll have the entire thing put up on my github.  To get started with this I went to a directory I wanted to work in and entered:

```sh
vue create -d lootly
```

The `-d` flag means use all the default answers for things.  I don't know what options I used when I created my first vue project with Typescript.  So, I'm going with default then I'm going to copy some of the dependencies from that project into this one to get things working.

Once you run it go get a drink or something - it takes a while.  My corporate network is particulary bad so I'm going to take a nap.

Once it is done you can cd into the `lootly` directory and type `npm run serve` and you'll have a working vue website.  It isn't much, but it's a start.  You can edit files and it will "hot-reload" the page you open at http://localhost:8080.  So keep it running and we'll see what happens.  Basically, your current project will look just like this [commit](https://github.com/finalcut/vue-lootly/tree/eb86dc6380e2aa29f917072decd1b6cf05631d5a)

### Quick Overview of the Structure

When the cli finishes you end up with three primary folders inside your project folder.

* node_modules - you'll never really have to navigate around in there.  node uses those and the modules that exist in there are all defined in the package.json file that is in the root directory of your project.
* public - has just a couple files.  These serve as the "wrapper" to your cool vue application.  Don't muck with these yet.
* src - here is where the vue code you'll be writing goes.  We'll spend most of our time in there.

In addition to the directories there are a few files:

* .gitignore - used by git.  If you don't know what that is; it's beyond the scope of this sorry. But this file tells git which files to ignore.
* babel.config.js - babel in essence converts all the cutting edge language stuff we might do in javascript or typescript and converts it to plain old javascript that will work in all the modern browers.  It's awesome.  This file tells babel some rules it needs to know.
* package-lock.json - don't touch this file.  Seriously.  It is created by npm and managed by it. If you muck around with it you might cause yourself some grief.  You can delete it but the next time you build it will come back.
* package.json - lots of import configuration in here and all of our external dependencies are defined here.  The ESLint rules are also in here and, while they can stay in there, I'm not a huge fan of that so I'll be moving those into their own file.  We'll get into that in the next section.

## Some Initial Tweaks

The next steps are going to be some housecleaning.  Like I said, I don't know what my initial settings where when I did this before so I have kind of figured out what I have to do on the fly to reproduce the initial project state.

First you need to update your package.json - use this [commit](https://github.com/finalcut/vue-lootly/commit/2084baba56a50180d233657a2d682eed3b481203) to see the differences.  After you update your package.json you'll need to run.

```sh
npm install
```

This will take a bit of time and pull in the rest of the dependencies.  Note, not only did I add a bunch of other dependencies I also removed a section from the file for eslint.  I prefer to have that in it's own config file; it just makes more sense to me that way.  The eslint files `.eslintrc.js` and `.eslintignore` are in the same commit as the updated package.json; so feel free to copy them as well.

There are a few other files I ended up needing/wanting so here they are.  Again, you can probably just [copy these from my repository](https://github.com/finalcut/vue-lootly/tree/efb839f7ed3038e4954e46f1af0fb3647347a961).

```sh
# babel is a translator, this file tells babel how to do it's magic
/babel.config.js
# this is super important since we're using typescript.  One setting that I had to tweak was  "experimentalDecorators" which I set to "true".
/tsconfig.json
# this is where you can configure webpack (how vue deploys things) as well as some extra rules for your development process.
/vue.config.js
# these shim files help typescript understand various types.
/src/shims-tsx.d.ts
/src/shims-vue.d.ts
```

At this point, go ahead and delete the file in `/src/components/HelloWorld.vue` we won't use it and it will just clutter the area later.

Next, change `/src/App.vue` to be this simple bit of code.

```html
<!-- single file component; has template at the top, and logic/script at the bottom -->
<template>
  <div id="main">
    <h1>{{ AppTitle }}</h1>
  </div>
</template>
<script lang="ts">
// these decorators are used all over the place in our typescript app.
import { Component, Vue } from 'vue-property-decorator'

// almost everything we create will be a component
// vue apps are components made up of other components
@Component({
  name: 'lootly'
})

export default class extends Vue {
  // this is a property getter.  Typescript lets us say what type of
  // value will be returned (string).  It's used up in the template as {{ AppTitle }}
  get AppTitle (): string { return 'lootly' }
}
</script>
```

You can learn more about decorators at [https://medium.com/@rossbulat/get-started-with-typescript-decorators-cf3924c37f04](https://medium.com/@rossbulat/get-started-with-typescript-decorators-cf3924c37f04).  It's a medium article so you may be rate-limited; you can go incognito if you really want to read it.

And finally, you'll need to make tweaks to the `/src/main.js` file:

1. rename it to main.ts - this identifies it as a typescript file
2. edit the main.js file and remove the comma on line 7
3. update line 2 to change the . to an @

main.ts should look like this

```js
import Vue from 'vue'
import App from './App.vue'

Vue.config.productionTip = false

new Vue({
  render: h => h(App)
}).$mount('#app')
```

You can [see the entire appliation, as it stands now, at this commit point](https://github.com/finalcut/vue-lootly/commit/e7416fa33df745d81fc5f29fa6c69c516c82039f).

## Conclusion

At this point we've done quite a bit and don't have a whole lot to show for it.  But, we're set up to start making some real rapid progress.  We've created our first typescript component (the actual app); we've used property decorators, we've created a "getter", which is effectively a "computed" property the way Vue is typically described.

In the next article we'll start by including Vuetify (a nice material design based library), create a second component that has props, "data", and we'll setup routing!