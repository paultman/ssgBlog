---
title: "Using Rollup to bundle JS assets"
date: "2021-09-10"
categories: 
  - "tech"
tags: 
  - "bundler"
  - "rollupjs"
  - "tree-shaking"
image: "/assets/blog/344a2-twitter-card.jpg"
---

At the end of this blog post you will:

- Understand some history of bundlers
- Understand why rollup is useful
- Learn how to use rollup in your own projects
- Have a working example illustrating a big benefit of Rollup, tree-shaking

When writing webapps/webpages these days, we often import 3rd party libraries or even our own self contained modules. For the import to work, we run our code through a bundler which will pull all the modules together. It would typically put all the disparate code in one file, sandboxed to avoid global namespace pollution, and linked to easily access imported modules.

They were typically used with a "task runner" which would do various tasks, one of which was the bundling of resources. Typically the task funner, and the bundler, are two separate tools, however now this isn't always the case.

For node-based, client development, I configured a task runner to do things like run tests, clean distro directories, move files, optimize images, bundle code, internationalize files, etc. My first "task runner" was Grunt, then Gulp, then a custom JS script, and now i'm considering an all-in-one task runner + bundler.

For javascript bundlers, I first use the very simple "Browserfy," then jumped on the WebPack bandwagon, and since I've played with Parcel and now Rollup.

If you'd like to see a comparison of bundlers, have a look at this [writeup](https://bundlers.tooling.report/) by Jake Archibald where be tests each for features. In my view, if you have a basic project driven by an html page, use Parcel. For a larger project, Rollup, and for a larger one where you need better support/documentation/community it's hard to go wrong with Webpack, albeit at the cost of complexity.

Ah, looks like Jake has a similar opinion:

> Parcel: Being able to go HTML-first is the best design for a bundler targeting “the web”.
> 
> Rollup: Simpler API and design makes writing plugins easy. Well documented. Small output.
> 
> Webpack: Community plugins are great. Good CSS support.

With all that out of the way, i'm going to explain to you my current setup, the last stage in my taskrunner/bundler evolution. The official rollupjs guide can be found [here](https://rollupjs.org/guide/en/) and is quite useful. I'll be giving you the condensed version, but you should refer to the guide for specifics.

First, we need to talk about the why. Why this evolution to rollupjs, what's new about it to make me change? Well, these days we use lots of third party code, but we don't typically use all the JS functions that the generic package comes with. One way to reduce the penalty of delivering this unused code was though minifying out code, basically compressing our bundles to be significantly smaller.

But, as we begin to use more and more 3rd party code, there should be a way to inspect our code, then eliminate 3rd party code which never gets used to reduce our bundle size. This process of eliminating unused code is called [tree shaking](https://en.wikipedia.org/wiki/Tree_shaking), and was initially adopted by newer bundling tools like rollupjs.

![](images/e425b-dart-tree-shaking.png)

A visual look bundling only used code

To make an example project, you can clone [this](https://github.com/paultman/rollupExampleProj.git) repo, or create it manually via the steps I'll outline below. _Note: If you use the repo, after cloning, you can update the working directory to the pre-minify output version by calling, "git checkout 91d103" which will revert your working tree so you can more easily read the bundled js file._

**To create the project from scratch**: Open your terminal, create a new project directory and CD into it. Then init a new npm project with the "y" flag to accept all defaults.

```
mkdir rollupExampleProj; cd $_; npm init -y
```

Now that we have an npm project, we want to add a few packages, namely Rollup and a plugin to move files, and another to start a local webserver. This is because we will be using Rollup as not only a bundler, but as a task runner as well. Here is a [list](https://github.com/rollup/awesome) of other useful rollup plugins.

```
npm i rollup rollup-plugin-copy rollup-plugin-serve -D
```

Now we need to add a few files, namely an html file, a javascript file, and a module which we will import containing a used function, and an unused one.

./src/index.html

```
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
  </head>
  <body>
    <h1 id="header">text to be replaced</h1>
    http://./js/index.min.js
  </body>
</html>
```

./src/js/index.js

```
import { someMethod1 } from './someModule.js';

const name = 'somebody';
document.querySelector('#header').textContent = someMethod1(name);
```

./src/js/someModule.js

```
export function someMethod1(name) {
  return `hello ${name}`;
}

export function someMethod2(name) {
  return `goodbye ${name}`;
}
```

Now need to make a rollup configuration file. You can find all the config [options](https://rollupjs.org/guide/en/#using-config-files) on the rollup website.

./rollup.config.js

```
import copy from 'rollup-plugin-copy';
import serve from 'rollup-plugin-serve';

export default {
  input: 'src/js/index.js',
  output: { file: 'dist/js/index.min.js', format: 'iife', sourcemap: 'inline' },
  plugins: [
    copy({
      targets: [{ src: 'src/index.html', dest: 'dist/' }],
    }),
    serve('dist'),
  ],
};
```

For the last step, we need to update our npm config to invoke rollup for the build, which will also invoke the serve plugin to serve out test page.

./package.json

```
...
 "scripts": {
    "build": "rollup -c"
  },
...
```

Now, run that command with "npm run build", if everything worked and you open your web browser to the default port of 10001 you should see this:

![](images/147e5-screen-shot-2021-09-10-at-11.59.25-am.png)

So what happened?

- We told Rollup to run, and use the config file we created
- The config file told Rollup where to find our JS file
- Rollup looked at the JS file and imported the parts of the import file that were actually used, someMethod1 and exported that bundled file to our dist directory

Have a look at the bundled JS file:

```
(function () {
  'use strict';

  function someMethod1(name) {
    return `hello ${name}`;
  }

  const name = 'somebody';
  document.querySelector('#header').textContent = someMethod1(name);

}());
```

First, it's wrapped in an IIFE, Immediately Invoked Function Expression, so that there are no global namespace pollution, next you'll see that the code for "someMethod1" is inserted. If you look at the original module itself, you'll see that it also had a function "someMethod2" but that did not get into our dist bundle, since we never used it. You can try changing the method you called to someMethod2, and after building you'll see that the generated bundle will only contain that method. This selective code inclusion is called "Tree Shaking."

Now that you've seen the output file and how the selective code inclusion works, we can finish optimizing our bundle by minifying it. We saved this step till now since it will be more difficult to read the obfuscated code. To do it, you'll want to use [rollup-plugin-terser](https://github.com/TrySound/rollup-plugin-terser). Instructions are in the readme, but basically you'll need to install the plugin via npm, add its import in your rollup config, and call it on the file output.

```
npm i rollup-plugin-terser -D
```

```
import copy from 'rollup-plugin-copy';
import serve from 'rollup-plugin-serve';
import {terser} from 'rollup-plugin-terser';

export default {
  input: 'src/js/index.js',
  output: { 
    file: 'dist/js/index.min.js', format: 'iife', sourcemap: 'inline', plugins: [terser()] 
  },
  plugins: [
    copy({
      targets: [{ src: 'src/index.html', dest: 'dist/' }],
    }),
    serve('dist'),
  ],
};
```

Now if you look at our output file, you'll see it's minified and you can see our inline source map file as well.

![](images/d34ff-screen-shot-2021-09-11-at-7.31.42-am.png)

In addition to optimal code bundles, you'll find that rollup can also handle tasks typically left for a task runner like gulp or grunt. You've seen above how you can use a plugin to move files. Here's a [link](https://github.com/rollup/plugins) to a list of other useful plugins.

Btw, the same can be done with CSS frameworks. If you are only styling a page layout, why do you need all the classes for form styling or buttons? Using postcss to eliminate unused css is a reason [Tailwindscss](https://tailwindcss.com) became so popular. I have some opinions on it, which you can find [here](https://paultman.wordpress.com/my-view-of-tailwind-css/).
