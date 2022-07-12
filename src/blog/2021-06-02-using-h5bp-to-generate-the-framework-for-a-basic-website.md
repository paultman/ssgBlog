---
title: "Using H5BP to Generate the Framework for a Basic Website"
date: "2021-06-02"
categories: 
  - "aws"
  - "tech"
tags: 
  - "h5bp"
  - "website"
image: "/assets/blog/88b42-html5boilerplate.png"
---

_Target Reader: Someone who knows Git basics and wants to generate a bare-bones, current-standards compliant, website with a typical npm/package-based directory layout._

Related to another guide I'm writing, hosting your website on Amazon, readers need to have a website ready, so I'm creating this a guide to quickly generate and customize one.  
  
It is based on the HTML5 Boilerplate (H5BP), "The web's most popular front-end template." It has over 50k stars on [github](https://github.com/h5bp/html5-boilerplate) and although it has been around forever, its still regularly updated.

There are alternatives. If you are looking for the most basic index.html template, including a detailed explanation of each line, have a look at Manuel Matuzovic's recent (April 9, 2021) blog [post](https://www.matuzo.at/blog/html-boilerplate/). If on the other extreme, you would like to start with a dynamic website template, which has many defaults and sets you up with React, check out [create-react-app](https://github.com/facebook/create-react-app). If you are new to React apps, I'd recommend you set things up manually first, so you learn how it works. Two good guides are a [writeup](https://medium.com/@tim.givois.mendez/create-a-react-project-from-scratch-without-create-react-app-f02fce4e05b) by Tim Givois or this [one](https://www.srijan.net/resources/blog/setup-react-app-from-square-one-without-using-create-react-app) by Rawat Ak.

In this guide, we will be going "middle of the road" and will be using the "create-html5-boilerplate" npx script. It leverages the H5BP to generate the basic files for a website. It does more than Manuel's single index.html file, by creating a full directory structure along with a package.json. The later leverages NPM to easily import other libraries/resources. It also comes with Parcel, a task runner and compiler to optionally transform resources like, sass or typescript, to be ready for web browsers. It does all this, in a standards compliant way, and should be considered as the minimum baseline for a website which can be improved over time without locking you into a specific implementation.

What this guide will do is:

\- Use the HTML5 Boilerplate. A bare bones website setup that includes a standard website structure, basic css files, etc. I will explain each part, and offer suggestions for customization.  
\- The generated template will use normalizecss and other configuration to minimize web browser variations, so that your webpage will look the same on most common browsers, in a standards compliant way.  
\- Setup an extendable site based on a typical package.json/npm structure to easily leverage 3rd party node\_modules libraries.  
\- Leave the choice to use a layout manager like Bootstrap, Foundation, or Materialize, up to you.

We will start with a script which will generate all the directories and files using the latest version of the HTML5 Boilerplate temple.

I'm using a mac, though the windows setup should be similar. You should have node installed on your machine. See this [guide](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm) is you don't.

From the command line, browse to the parent directory of where you will be generating your website directory. Here we will envoke the [create-html5-boilerplate](https://github.com/h5bp/create-html5-boilerplate) script. For our example, I'm going to call the website myPersonalWebsite1, replace that name in the subsequent text with whatever you want to call your website.

```
> npx create-html5-boilerplate personalWebsite1
```

If you have Visual Studio Code installed open the newly created project. It should look like this:

![](images/4c962-h5bp_dir_structure.png)

That simple command generated all those files. We will look into a few important ones, but first lets remove a few unnecessary files.

The files generated include lots of [markdown](https://en.wikipedia.org/wiki/Markdown) files in the doc directory. You can read them there, or see the rendered markdownon the h5bp github [page](https://github.com/h5bp/html5-boilerplate/blob/v8.0.0/dist/doc/TOC.md). For our basic website, it's unnecessary to track them. You can either delete the directory, or add it to our .gitignore file (in your root directory). Speaking of which, delete the .gitignore in the img directory. I found an old mention of it on [github](https://github.com/h5bp/html5-boilerplate/issues/1162) and think it's unnecessary.

Next is the .htaccess file at the root of the project. If you are not hosting your website on an apache webserver, delete it.

Two other . files at the root of the directory are [.gitattributes](https://dev.to/deadlybyte/please-add-gitattributes-to-your-git-repository-1jld) which tells git have to save certain files, and [.gitignore](https://git-scm.com/docs/gitignore/en) which tells git which files not to save. Do not delete them :-)

There are also a few unnecessary images, but rather than deleting them now and having invalid references in files, we will save that for later.

Now, before we make any other changes, it's recommended to save your current project state to source control. I'm using Git for source control (recommended).

```
> git init
> git add .
> git commit -m "pruned files generated for personal website using h5bp generator"
```

I recommend reading some docs from the H5BP readme, starting with the [usage](https://github.com/h5bp/html5-boilerplate/blob/v8.0.0/dist/doc/usage.md) page. That will walk though all the parts of the project. As the guide says,

> HTML5 Boilerplate is a starting point, not a destination.

Meaning it provides the initial framework to later add custom content and even other frameworks on top of.

Site icons. This is one area that I would replace the generic h5pb ones, and overwrite their references in the browserconfig.xml and index.html files. If you don't already have them ready, go to an online icon generator website like [realfavicongenerator](https://realfavicongenerator.net). Click "Select your Favicon image" and upload a square image you have. You can customize the appearance on the following screen, and/or scroll to the bottom and click the button, "Generate your Favicons and HTML code"

With those files, copy them to your website folder, overwriting the previous browserconfig.xml, site.manifest (update name and short\_name fields) and favicon.ico files, and deleting: tile.png, title-wide.png, icon.png.

Last, copy the html from the website and replace the 3 lines in your index.html (around 15-17) from <link rel="manifest" to the comment telling you to place the favicon in your root directory. Also update the og:image line with a url to one of the images. You could also make a copy for the largest, the android-chrome-512x512 one, or use your original logo file and name it icon.\[original ext\]. For example:

![](images/78167-faviconupdates-1-1024x300-1.png)

There are two other generic configuration files which you should update with you particular website details, they are: humans.txt, and package.json. You can read about them in the [misc](https://github.com/h5bp/html5-boilerplate/blob/v8.0.0/dist/doc/misc.md#humanstxt) section of the h5bp readme.

Google Analytics support. (Optional)

While it's unnecessary for a personal website, the boilerplate also provides code for google analytics. It can slow down your website, but it can provide you with interesting data about your viewers. You can read more about what's captured and how it works [here](https://developers.google.com/analytics/devguides/collection/analyticsjs). If you want to skip it, simply delete the two script sections at the end of the html body.

```
  <!-- Add your site or application content here -->
  <p>Hello world! This is HTML5 Boilerplate.</p>
  http://js/vendor/modernizr-3.11.2.min.js
  http://js/main.js

  <!-- Google Analytics: change UA-XXXXX-Y to be your site's ID. -->
  <script>
    window.ga = function () { ga.q.push(arguments) }; ga.q = []; ga.l = +new Date;
    ga('create', 'UA-XXXXX-Y', 'auto'); ga('set', 'anonymizeIp', true); ga('set', 'transport', 'beacon'); ga('send', 'pageview')
  </script>
  https://www.google-analytics.com/analytics.js
```

Google now had a newer analytics package called Google Analytics 4 (using gtag.js) which still uses the previously Universal Analytics (analytics.js) under the covers. The new version has been [discussed](https://github.com/h5bp/html5-boilerplate/issues/2014) with members of the h5bp team. And the consensus is to still use the UA version. Incidentally I setup a wordpress site last week and the most popular plugin also recommends the UA version. When [creating](https://analytics.google.com/analytics/web/#/a182708740p252390553/admin/property/create) your property with google analytics, click "Show advanced options" click the toggle to enable, "Create a Universal Analytics property" and select the second option box for "Create Universal Analytics property only". After putting in your property name, url, etc, you will get a tracking ID which you can use on your index.html page in near the bottom in the analytics section.

Miscellaneous

.editorconfig // this is for consistent settings across editors. If using VSC, add this [extension](https://marketplace.visualstudio.com/items?itemName=EditorConfig.EditorConfig) to use.  
js/plugins.js // unless you are going to install jQuery plugins, or are worried about console log support in non-mainstream browsers, you can delete this file. Also remove the reference from the bottom of the index.html file.

The last thing to do is the actual website/presentation itself. That means updating the index.html, js/mail.js, css/main.css files.

While updating the html in your site, you can run

```
> npm run dev
```

that will compile all your local files, save those to the dist directory, start a local webserver and open your default browser to your compiled index.html page.

Once you are happy with how the first version of your site looks, run git status, it should look like this:

![](images/1738f-gitstatusafterchanges-1024x845-1.png)

Those are all the changes you have made. Last, add and commit them with

```
> git add .
> git commit -m "update h5bp config and add first version of website"
```

That's it. Now that you have a website, and code repo, you can deploy it to amazon, free!
