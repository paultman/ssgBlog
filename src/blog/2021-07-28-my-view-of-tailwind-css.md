---
title: "My view of Tailwind CSS"
date: "2021-07-28"
categories: 
  - "tech"
tags: 
  - "css"
  - "css-framework"
  - "tailwindcss"
image: "/assets/blog/97129-tailwindcss-1.png"
---

I have now used Tailwind CSS v2.2 to update two websites. They were both fairly basic landing pages, the first was originally written by another developer, the second, was an update to one I personally had written. After that experience, I'd like to jot down all my thoughts and mostly likely review this post over time as I forget, and as Tailwindcss itself improves.

For those who don't know, Tailwinds is a css utility library. In a nutshell: It gives you low level classes so you can use a chain of css property classes to express your css inline in the html class of an element. The compiler has the ability to inspect the html/css to only build styles that are used. Last, it includes the ability to auto generate specific attribute values for themes, so you have a custom bundle of css properties and theme specific values.

It has exploded on the web dev scene with the first version, 1.0 being released in May of 2019. It already has over 45k github stars at the time of this article (v2.2.6) at the end of July 2021.

When I first started using it, I was enamored and though it could be the missing piece in my web dev toolkit.

I'll start by going over the positives:

- Easy to learn. There are many resources to learn Tailwinds: the website, cheatsheet, and tons of Youtube [videos](https://www.youtube.com/watch?v=qYgogv4R8zg). In addition there are discussion forums and discord groups.
- It's easy to use. The utility class names are representative, nearly all encompassing, and low level/simplistic.
- It's customizable, you can define "theme" or branding values which you can use as tailwind utility classes or as values in a larger composition. With Tailwind2.0 you can even define custom values right in your html and have them generated on the fly into your css.
- On a team, it forces everyone to be on the same page and do things in a similar way. Each is forced into the Tailwindcss way and slightly restricted into using that framework and classnames.
- It's easy to read and understand page presentation/structure all from the html file. No flipping back and forth between html and css files. Also when coding, you can stay in just the single html file for the majority of the cases.
- It's being updated constantly. Besides Adam Wathan, the creator, there are dozens of other developers working on the framework and it keeps improving. V2.0 was a big upgrade.
- Last, unlike some other general purpose frameworks, tailwind leverages postcss to strip out unnecessary css that's not used, so you are left with lean css files which only have classes that you use.

That's a lot of positives. And thinking back to my own usage, I have to admit, it felt clean/simple. Even though I only used Tailwindcss for two landing pages, I could do nearly everything I needed. But there are a few drawbacks to the framework:

- I'll start with the biggest knock against frameworks or higher level tools, it adds another layer, between the html and css. This can be good and bad, but it does mean an extra thing to learn, and a changing thing, one that will be needed to be updated and maintained.
- It's doing what you could just do with CSS today. CSS itself has gotten more powerful with things like transform & translate, media queries, flexbox/grid, responsive containers, etc. The hallmark comparison is doing layouts with bootstrap grids over doing it yourself with combinations of floats, min-width, etc. Leveraging a framework for page layouts is no longer necessary, [plain css](https://youtu.be/qm0IfG1GyZU) can do it easily.
- Some custom things are not easy to do. For example, if you dynamically add a class, which isn't in the html statically, you have to handle it in a special way so the style doesn't get stripped by post-css. Or trying to do the Tailwindcss processing in an html file with @apply inside a local style tag doesn't work. By having to do certain things outside your html file, some of the html readability benefits are reduced.
- Speaking of the @apply to make custom Tailwindcss compositions, the creator of tailwind himself hates them and wishes they were never put into the project. See his [rant](https://twitter.com/adamwathan/status/1226511611592085504?lang=en). In my view, it gives us back a huge benefit of CSS (signatures), while still leveraging atomic tailwinds classes.
- Last, tailwinds fundamentally goes against how CSS itself was designed to work. It was designed to define certain element signatures to take certain css properties. You define which element(s) via a selector, should have certain properties. If you make a new element that fits that selector, automatically it has those css properties. You define it once, and it applies for that selector, even dynamically added ones. Tailwinds on the other had needs class properties for each element, so you get tons of repeated classes on similar items.

In the end, it's hard to say if I'll continue with using Tailwindcss. For more than a simple landing page, it's unlikely. Without it, my current methodology is to come up with a style guide and branding upfront for each project, set rules, and consistently follow them for new product design.

Is it worth learning? Depends. I think everyone should learn at least one framework and experience how consistent the development/usage can be, especially on a project with other developers. Now that I already know Tailwinds, I'd likely use it on smaller projects like a landing page, however for custom applications, I'd most likely do it with custom css or scss.
