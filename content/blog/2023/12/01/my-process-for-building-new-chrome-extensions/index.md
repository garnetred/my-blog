---
title: "my process for building new chrome extensions"
date: "2023-12-01"
categories:
  - "coding"
tags:
  - "browser-extensions"
coverImage: "lindsay-henwood-7_kRuX1hSXM-unsplash.jpeg"
description: "A step-by-step guide to building your first Chrome extension using Manifest Version 3 (MV3)."
---

<figure>

![a person walking up a set of stairs](images/lindsay-henwood-7_kRuX1hSXM-unsplash-1024x683.jpeg)

<figcaption>

Photo by [Lindsay Henwood](https://unsplash.com/@lindsayhenwood?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash) on [Unsplash](https://unsplash.com/photos/person-stepping-on-blue-stairs-7_kRuX1hSXM?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash)

</figcaption>

</figure>

Recently, I decided to build another small Chrome extension, this time for [edX](http://edx.org). I signed up for a course and began going through the course materials. I noticed that there was a notification pane to the right-hand side of the page that was open by default. The only notification encouraged me to use a discount code to upgrade the course. Although you could close the panel manually, once you navigated to the next page, it would be open once again. To me, it was clearly a dark UX pattern designed to encourage more people to upgrade the course instead of auditing, but I found it so distracting that it took away from the course itself.

So I decided to create [another small browser extension](https://decembergarnetsmith.com/code/#extensions) to fix this issue. This time I wanted to take some time to document the exact process that I use to build new [Chrome extensions](https://github.com/stars/garnetred/lists/browser-extensions).

### getting started

I started with the initial setup. I created a new folder on my computer with the same name as my future browser extension ([hide-edx-notifications](http://garnetred / hide-edx-notifications)). After that, I copied over the contents of my [browser extension template](https://github.com/garnetred/browser-extension-template) folder into the new `hide-edx-notifications` folder.

I created the browser extension template after noticing that I oftentimes created very similar browser extensions; typically they remove some or multiple elements from the DOM. Rather than typing out the same code over and over again, I decided to create a base template that I could use for all of my browser extensions. This easily shaves off an hour or so of my development time. After this, I'll usually set up my Github repo following their instructions for [adding a local repository to Github](https://docs.github.com/en/migrations/importing-source-code/using-the-command-line-to-import-source-code/adding-locally-hosted-code-to-github#adding-a-local-repository-to-github-using-git).

### browser extension basics

#### types of extensions

Manifest is Chrome's extension API. There are two versions of Manifest used currently: Manifest V2 and Manifest V3. Manifest V2 is an earlier version that is being deprecated; browser extension developers will need to update their older extensions to V3 in order for them to continue working. In June 2024, [Manifest V2 extensions will be removed from the Chrome Web Store](https://developer.chrome.com/blog/resuming-the-transition-to-mv3/) for pre-stable versions of Chrome.

For that reason, this article only covers Chrome extensions built using Manifest V3.

#### files

Here are the basic files you need to build a browser extension:

- manifest.json

  - This file includes basic information about your extension such as the name, Manifest version, [permissions](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/manifest.json/permissions) (what your extension is allowed to do beyond basic functionality), etc.

- background.js

  - This is a [service worker](https://developer.chrome.com/docs/extensions/mv3/service_workers/) typically named background.js. It can access certain portions of a website such as the URL. It cannot access the same information as content.js, so you would need to send a message between the two files if you want to access that information.

- content.js

  - This is the [content scripts](https://developer.chrome.com/docs/extensions/mv3/content_scripts/) file, commonly named content.js. It can access the DOM within a particular webpage, but can't access the same information as background.js. You would need to send a message between the two pages in order to do so.

- popup.html

  - This file dictates what a user will see when they click on the Chrome extension icon in the toolbar. I usually add information about the version number and a link to the Github repo.

- popup.js

  - This file handles the JavaScript for the popup window.

- global.css

  - This file can be named differently, but it handles the CSS for the browser extension.

- icons
  - This isn't an individual file but multiple files for the Chrome extension icon (what you'll see in the toolbar and on the web store should you decide to publish an extension). These icons can be 16x16, 32x32, 48x48, and/or 128x128 pixels.

### building functionality

Once I have the basic file setup complete, I start building out the extension functionality.

For privacy reasons, my extensions only work on specific URL's. Therefore in the `background.js` file, I check that the current URL matches the website's url (in this case, `learning.edx.org/course`, which is present in the URL when users go through a specific course).

Next, I will inject the CSS that I created in the file above into the page using [insertCSS()](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/API/tabs/insertCSS). I use the Tabs API to only run this code [when the current tab is updated](https://developer.chrome.com/docs/extensions/reference/tabs/#event-onUpdated).

Here's a snippet of the code in `background.js` that injects the CSS at the specific URL:

```
chrome.tabs.onUpdated.addListener((tabId, changeInfo, tab) => {
  const tabUrl = tab.url ?? tab.pendingUrl;
  if (
    changeInfo.status === "complete" &&
    tabUrl &&
    tabUrl.includes("learning.edx.org/course")
  ) {
    chrome.scripting.insertCSS({
      target: { tabId: tabId },
      files: ["css/global.css"],
    });
  }
});
```

### selecting elements

The next step is finding a way to select the right element in the DOM. I'll typically open Chrome dev tools and inspect the page. From there, I'll use the inspector to select a particular element. Once I'm looking at the HTML for the right element, I'll look for any attributes I can use to select that element in the CSS. For example, an id, class, alt, or a data-testid attribute that isn't being used for any other element on the page.

Once I have that, I'll update my CSS file to use that attribute as a selector. In this case, I'm using the data-testid for a specific section tag. I'll then add the following CSS property to apply to that specific selector: `display:none`. This will remove the element from the DOM entirely.

### testing

The final step is testing the extension. To do this, I'll [load the unpacked extension](https://developer.chrome.com/docs/extensions/mv3/getstarted/development-basics/#load-unpacked) in the browser. Basically, you navigate to the extensions page (`chrome://extensions` in the address bar), click the button that says "load unpacked" and then select the folder for your extension in the popup that appears. You should now see your browser extension appear with the name, version number, and icon that you listed in `manifest.json`.

Once I've loaded the extension, I'll navigate to the appropriate URL, which should contain the URL string indicated in `background.js` (here, "learning.edx.org/course"). I'll check that with the extension enabled, I'm seeing the desired results. In this case I checked that with the extension loaded, when going through a course, the notifications pane was closed.

You'll notice that the button to open and close the notifications pane no longer works. That's because the way it worked before was when you clicked on the notifications pane when it was open, the `display` property was set to `none`. When you clicked the pane when it was closed, this property was removed. However, since this extension adds `display:none` even when the pane should be visible, it'll always have the `display:none` property so it'll never be visible.

Long term I would maybe want to keep that toggle functionality but for now this serves my basic needs.

### voila!

And there you have it - a working browser extension that removes an element from the DOM. You can build on that functionality to remove multiple elements or use this as a base to add more complex functionality. I like building small browser extensions, so if I wanted to create another extension that did something similar on the same website, I would probably create a separate extension.

#### (but wait, december. could i use your template to learn how to make browser extensions?)

You could, theoretically; the code is on github. However, I wouldn't recommend it. If you started with the base template, you would probably end up with a working browser extension, but you wouldn't have learned how to build browser extensions. It's the same problem with following multiple tutorials and trying to build something from scratch, only to realize that you don't know how. In the future I'll write another article on the best resources for learning to build browser extensions and link it here.

---

Want more articles like this delivered straight to your inbox? Subscribe below!
