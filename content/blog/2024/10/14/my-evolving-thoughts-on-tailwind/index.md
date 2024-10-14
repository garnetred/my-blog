---
title: "my evolving thoughts on tailwind versus native css"
date: "2024-10-14"
categories:
  - ""
tags:
  - "op-ed"
description: "Discussing the pros and cons of tailwind and other css utility frameworks from a new perspective."
---

if you had asked me a couple of years ago, "what do you think of tailwind?" i would've probably said that i liked it. "it makes writing css much easier," i might have said. however, after a full year of writing browser extensions using vanilla javascript, html, and css, my opinion has virtually 180'd. i've started noticing pitfalls and shortcomings that weren't obvious before.

examples:

- can't fully replace css with tailwind or bootstrap so you end up having to use both
  - ex. i downloaded an eleventy template (not for this blog) that was great, except it used bootstrap. the template by default had a lot of patterns that violate common accessibility rules. for example, buttons would use an animation to change the background color on hover. the contrast was poor and all of the colors needed to be updated. if you hovered over a random element, it was likely to move for no reason. the only way for me to override this successfully was either to edit the minified bootstrap file (not recommended) or _write more css to manually override bootstrap_. in no world does that make sense. in some code bases this results in a confusing mix of css and a css utility framework. worst case scenario it's hard to tell why the css isn't behaving as it should and there are a lot of `!important` attributes everywhere.
- clutters up the html
  - since i write a lot of browser extensions that involve dom manipulation, i spend an inordinate amount of time looking at random websites's html. what i've noticed is that the html has become so cluttered with utility classes that it's not incredibly difficult to read. in some cases, there aren't any regular css classes so it becomes all the more difficult to select a specific element. this is an issue for me, a browser extension developer, but it's also an issue for the developers who work on this codebase, too. (it feels like it should be an anti-pattern.)
- largely inefficient
  - with traditional css, you can use one class to determine the text color. everywhere you want to use that color, you can add that class to the respective html element. if later you change your mind, all you have to do is go to that one spot in the css and change the color - it'll be updated everywhere. if, on the other hand, you use tailwind, you'll have to manually change the tailwind class everywhere it appears. (unless you want to rename the class entirely.)
  - if you want to remove an element from the dom, you can use the `display: none` property. you can create a class and then add this as a css rule, then add that class to any elements you want. in bootstrap, you can add the utility class `display-none` to multiple elements. is this really an improvement? if you change your mind and instead just want to hide the element from the dom, you can't just modify the css rule to be `visibility: hidden`. you have to change it everywhere.

---
