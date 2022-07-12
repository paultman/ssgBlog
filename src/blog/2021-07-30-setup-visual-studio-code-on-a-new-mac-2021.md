---
title: "Setup Visual Studio Code on a new Mac in 2021"
date: "2021-07-30"
categories: 
  - "tech"
tags: 
  - "airbnb-style-guide"
  - "eslint"
  - "npm"
  - "prettier"
  - "visual-studio-code"
image: "a58f5-images.png"
---

updates:

May 31, 2022 - Added CSS lint tool to project, and to Visual Studio.

Dec 23, 2021 - Removed "Bracket Pair Colorizer"extension, now included in VSC by default

## Intro

As a longtime developer, my most important tool is my code editor. I started with vi, then when I started using a Mac, to [TextMate](https://macromates.com), then [Sublime Text](https://www.sublimetext.com), now finally [Visual Studio Code](https://code.visualstudio.com) (VSC). The final move was mainly for all the extensions and because all the cool kids were doing it :p It's from Microsoft, and is a free version which borrows from their Visual Studio suite. I also used VS back in the day, but not as long as the other editors.

This post is going to start with general usage, then will go into settings/configs as a full stack developer mainly using Javascript. At the end, I'll include two bonus sections. First, I'll show you how to use VSC with [Amazon Web Services](https://aws.amazon.com) (AWS), and second I'll how you how you can use VSC to help with competitive programming, specifically on [Leetcode](https://leetcode.com/paultman/).

You can find a screencast of the setup on [YouTube](https://youtu.be/8UrCpyUs6a8). Also, you can check out the associated code repo on [GitHub](https://github.com/paultman/npm-starter-vcs).

## Basics

VSC matches most visual Integrated Development Environments (IDE's) in that it's divided into three sections, left for files, right for the currently opened document and bottom for an optionally opened terminal window.

![screenshot of default VCS window with an open project](images/Screen-Shot-2022-05-31-at-9.31.58-AM-1024x881.png)

screenshot of default VCS window with an open project

The left sidebar (activity bar) defaults to the file explorer but can also be used for Search, Version Control/Git, Debugging, and for Extensions.

The three most useful keyboard shortcuts are: the Command Pallet using **shift-cmd-p** on a mac, or **shift-ctrl-p** on windows, the Quick Open to open files, **cmd-p** on mac or **ctrl-p** on windows and the Toggle Terminal command, **ctrl-\`** for both . Here are links to keyboard shortcuts: [mac](https://code.visualstudio.com/shortcuts/keyboard-shortcuts-macos.pdf) and [windows](https://code.visualstudio.com/shortcuts/keyboard-shortcuts-windows.pdf). The links can also be opened by typing **cmd-kr** for mac and **ctrl-kr** on windows.

Themes. To preview and change your color theme, you can use the shortcut cmd-kt (mac) or cmd-kt (win). Use the down arrow to preview and enter/return to select.

Last, to really learn more then simple text edits, check out the Interactive Editor Playground where you'll learn more about things like Multi-Line editing and Emmet. Open it using Help in the menubar, then Interactive Playground.

## Extensions

There are nearly 30k [extensions](https://marketplace.visualstudio.com) available for VSC, here I will call out the ones I find most useful. I've also annotated each's popularity as of July 29, 2021. The first two extensions need npm packages to be installed.

[ESLint](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint) (15.6m) - As a Javascript developer, this extension helps to keep my code clean and adhere to a set standard. Out of the box, eslint comes with certain rules, or you can opt to follow others like [Airbnb's](https://github.com/airbnb/javascript) or [Google's](https://google.github.io/styleguide/jsguide.html) style guides.

[Prettier](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode) (14.1m) - VSC comes with a generic code formatter, but for a more web-dev focused version, this is the most popular. It is especially useful when combined with eslint. Btw, another code formatter is [Beautify](https://marketplace.visualstudio.com/items?itemName=HookyQR.beautify), however I don't recommend it as the last update was May 2019, and it supports less file types then Prettier (which is readily updated).

[GitLens](https://marketplace.visualstudio.com/items?itemName=eamodio.gitlens) (10m) - VSC supports source control already, but this plugin brings even more capabilities. If your code is in git version control, this plugin makes branches, diffs, history, much easier.

Update: Dec23, 2021. Colored brackets are now natively included in VCS, simply add the config found below to turn it on.  
[Br](https://marketplace.visualstudio.com/items?itemName=CoenraadS.bracket-pair-colorizer)[acket Pair Colorizer 2](https://marketplace.visualstudio.com/items?itemName=CoenraadS.bracket-pair-colorizer-2) (3m) - When looking though code, you often need to look for the end of a block, by having different brackets of different colors makes it easier to spot. It supports all typical bracket types like \[\],{}, () and custom types.

Update: May31, 2022  
[Stylelint](https://marketplace.visualstudio.com/items?itemName=stylelint.vscode-stylelint) Similar to ESlint for javascript, this lints our css/scss files. We will install it's dependencies and the standard ruleset via NPM.

### Less Common

[Vim](https://marketplace.visualstudio.com/items?itemName=vscodevim.vim) (2.8m) - If you have a background in Vi or you want to do as much as possible from your keyboard, this is the extension for you. To enable key repeat, turn it on for visual studio code by running this command once from the command line. This will disable the default Apple action.

```
> defaults write com.microsoft.VSCode ApplePressAndHoldEnabled -bool false
```

[AWS Toolkit](https://marketplace.visualstudio.com/items?itemName=AmazonWebServices.aws-toolkit-vscode) (404k)- Besides using the CLI for amazon web services, or the web console, you can also access the resources in VSC using this extension.

[LeetCode](https://marketplace.visualstudio.com/items?itemName=LeetCode.vscode-leetcode) (349k) - If you have used LeetCode, this is a great extension. It allows you to access all the leetcode exercises, run test cases, and even step through your code. Highly recommended if you use LC.

### Per Project Setup using ESLint + Airbnb's Style guide and Prettier

Even with the above extensions installed, you will still need to install the actually binary files to use eslint and prettier. Rather than installing globally, I (and eslint.org) recommend to install these as local dev dependencies on each project. That way you can check them into source control and other devs can work from the same configuration, and your global settings will not interfere. Speaking of, here is a [basic project](https://github.com/paultman/npm-starter-vcs) from the code below.

First, setup a new npm based project if you haven't already created one  
`> mkdir new_project && cd "$_" && npm init -y`  
That will create the project.json file for your project.

Next add ESLint as a dev dependency  
`> npm install -D eslint`

Configure ESLint using a script  
`> npx eslint --init`  
Select the options that apply to your project. When asked how you would like to use ESLint, select the last option, "To check syntax, find problems, and enforce code style," then select airbnb. The script will also install the airbnb rules, and the ESLint plugin importer.

![](images/ce0fc-setupvsc_eslint.png)

Next, install Prettier locally as a dev dependency and the exact latest version (which will not later be changed) with this command  
`> npm install -D -X prettier`

Then set the [configuration for Prettier](https://prettier.io/docs/en/configuration.html), create the file .prettierrc.json in your proj directory. Here is a basic config, setting a wide width (default is 80chars), single quotes, and trailing commas. Btw, i used to hate trailing commas because I thought it looked incomplete, however it does help with remembering them when adding properties later, and helps with git diffs since it does not show the previous line comma when adding new props.

```
{
  "printWidth": 120,
  "singleQuote": true,
  "trailingComma": "es5"
}
```

With Prettier installed, we need to configure ESLint to use it and not conflict with its' own rules:  
\> `npm install -D eslint-config-prettier eslint-plugin-prettier`

With that installed, we need to add "prettier" of the extends section of our [.eslintrc.json](https://eslint.org/docs/user-guide/configuring/) file, be sure to put it after airbnb since we want its' rules to trump those of airbnb.

```
{
    "env": {
        "browser": true,
        "es2021": true
    },
    "extends": [
        "airbnb-base",
        "prettier"
    ],
    "parserOptions": {
        "ecmaVersion": 12,
        "sourceType": "module"
    },
    "rules": {
    }
}
```

Now, you should be able to open Visual Studio Code in the current project directory  
`> code .`  
and check for problem in the console. When VSC opens, the plugins will read their configs. If there is a problem, you will see it here.

![](images/40dd8-setupvsc_console.png)

One last configuration is to have VSC use Prettier as the default code formatter, and to format/fix code on file save.

Open your workspace settings via command pallet and typing "Preferences: Open Workspace Settings" There, add this configuration

```
{
    "editor.formatOnSave": true,
    "editor.defaultFormatter": "esbenp.prettier-vscode",
    "editor.codeActionsOnSave": {
      "source.fixAll.eslint": true
    },
  "editor.bracketPairColorization.enabled": true,
  "editor.guides.bracketPairs":"active"
}
```

May 30, 2022 Check our css styles using Stylelint

Besides JS formating, we also need to be concerned with our HTML and CSS. Here's a Google [style guide](https://google.github.io/styleguide/htmlcssguide.html) for good conventions.

For CSS, we can also add a tool to our environment to alert us when we have formatting/style issues.

The origins of css linting were from the same guy who gave us eslint, [Nicholas C. Zakas](https://github.com/nzakas) and [Nicole Sullivan's](http://www.stubbornella.org/content/) with their project, [CSSLint](https://github.com/CSSLint/csslint). Later, css-tricks made their own and called it [stylelint](https://css-tricks.com/stylelint/). It's now the more popular of the two, and is what I recommend.

We have already added the extension to VSC, and to add the associated resources use:

`npm install -D stylelint stylelint-config-standard stylelint-config-prettier`

That will install stylelint along with the [standard configuration](https://github.com/stylelint/stylelint-config-standard) which is more popular and has more rules than the [recommended configuration](https://github.com/stylelint/stylelint-config-recommended). We also install a package to allow stylelint to [work well](https://github.com/prettier/stylelint-config-prettier) with prettier, which we installed earlier.

To tell stylelint to use those two packages, create the .stylelintrc.json config file in your project root directory and reference them.

{
  "extends": \["stylelint-config-standard", "stylelint-config-prettier"\]
}

BTW, this process used to be a lot tougher before the extension was released last year, here's some [history](https://stackoverflow.com/questions/58836612/how-to-use-vscode-prettier-3-formatting-with-stylelint/59752415#59752415) on the process.

\---

Now is a good idea to commit the base version of your project to source control. If you haven't already, init a git repo:  
\> `git init`  
You should also add a gitignore, we will just pull a standard one from github for node.  
\> `curl https://raw.githubusercontent.com/github/gitignore/master/Node.gitignore >> .gitignore`

That's it, add any other files and commit your changes. You should be all ready to create your next masterpiece! Or at least a properly formatted "Hello World!" :p

#### Extras

To have install a local static web server that can be invoked from any directory, use NPM to globally install the [serve](https://github.com/vercel/serve#readme) package.

```
> npm i -g serve
```

If you're not using an M1 Mac, and might be concerned about fan noise or heat, check out this [article](https://paultman.wordpress.com/save-battery-and-processing-by-disabling-autoplay-ads-videos/?preview_id=2562&preview_nonce=0804af15d5&preview=true&_thumbnail_id=2571) to reduce system load while web browsing.

## References / Inspiration

[Getting started guide](https://code.visualstudio.com/learn/get-started/basics) from Visual Studio Code/Microsoft
