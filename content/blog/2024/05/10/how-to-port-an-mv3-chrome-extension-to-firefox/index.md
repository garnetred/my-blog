---
title: "how to port an mv3 chrome extension to firefox"
date: "2024-05-10"
categories:
  - "browser-extensions"
tags:
  - "browser-extensions"
coverImage: "rubaitul-azad-4xmVvHRioKg-unsplash-1.jpeg"
description: "A detailed guide on porting a Manifest Version 3 (MV3) Chrome extension to Firefox in 2024."
---

<figure>

![A tile with the Firefox logo on an orange background](images/rubaitul-azad-4xmVvHRioKg-unsplash-1.jpeg)

<figcaption>

Photo by [Rubaitul Azad](https://unsplash.com/@rubaitulazad?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash) on [Unsplash](https://unsplash.com/photos/logo-4xmVvHRioKg?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash)

</figcaption>

</figure>

Last year, I created my first Chrome extension ([No Linkedin News](https://github.com/garnetred/no-linkedin-news)). I always intended to port my extensions to Firefox but found that Firefox didn't offer much support for Manifest V3 (MV3) at the time. Since [Google Chrome is moving forward with plans to deprecate Manifest V2 extensions this year](https://developer.chrome.com/blog/resuming-the-transition-to-mv3), I wanted to take the time to port all of my Chrome extensions to Firefox, starting with [Better YouTube Search](https://github.com/garnetred/better-youtube-search), [Hide Amazon Cart](https://github.com/garnetred/hide-amazon-cart), and [Reddit to Old Reddit](https://github.com/garnetred/reddit-to-old-reddit).

While making these Chrome extensions cross-browser compatible, I noticed that some of the documentation that I found on Chrome and Firefox seemed to contradict one another. I ran into several issues on both sides so thought I would write a post detailing the steps that I took in hopes that it's helpful to others. You can check out [this pull request](https://github.com/garnetred/reddit-to-old-reddit/pull/2) for Reddit to Old Reddit on Github to view these changes line-by-line. In this tutorial I'll use this extension as an example; it's a small browser extension that redirects users from reddit.com to old.reddit.com. The tech stack is vanilla JS, CSS, and HTML.

To port my Chrome extensions to Firefox, I went through the following steps:

#### Step 1: Adding a gecko id

Firefox has some different requirements for their manifest.json than Chrome. For this extension, the main differences were:

- the [gecko id](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/manifest.json/browser_specific_settings#firefox_gecko_properties) property in the `[browser_specific_settings](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/manifest.json/browser_specific_settings)` key;

- background scripts versus service workers; and

- the format for the sub-key `extension_ids` within the key `web_accessible_resources`.

Firefox expects the gecko id to be either a [GUID](https://en.wikipedia.org/wiki/Universally_unique_identifier) or a string formatted similar to an email address. (Using a real email address can result in privacy issues due to increased spam.)

Here's an example of what that code might look like:

```
 "browser_specific_settings": {
    "gecko": {
        "id": "{1f39e6db-82c0-4af2-a25e-3f16809b31f6}"
    }
  }
```

#### Step 2: Removing any `extension_ids`

Chrome and Firefox expect any `web_accessible_resources extension_ids` to be in differing formats. Chrome expects a string of lowercase letters while Firefox expects a GUID with brackets, alphanumeric characters, and four hyphens. To fix this issue, I deleted the extension id. However, Chrome requires each object in the `web_accessible_resources` array to have at least two properties, so I added the `match` key with the same URLs as `host_permissions`.

For Reddit to Old Reddit, here's what that looks like:

```
 "web_accessible_resources": [
    {
      "resources": ["css/*.css"],
      "matches": ["*://www.reddit.com/*"]
    }
  ],
```

#### Step 3: Adding the Background Script

[Firefox does not currently support service workers](https://blog.mozilla.org/addons/2024/03/13/manifest-v3-manifest-v2-march-2024-update/). Instead, they use non-persistent background scripts, also called [event pages](https://blog.mozilla.org/addons/2022/10/31/begin-your-mv3-migration-by-implementing-new-features-today/). However, Chrome's Manifest V3 architecture relies heavily on service workers. To reconcile these differences, Firefox allows you to add your original service workers as background scripts. To do this, I used the `script` property in addition to the `service_worker` property in the [background](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/manifest.json/background#browser_support) key. Here's an example:

```
  "background": {
    "service_worker": "scripts/background.js",
    "scripts": ["scripts/background.js"]
  },
```

#### Step 4: Testing the Extension on Firefox

I tested the extension using the `[web-ext](https://github.com/mozilla/web-ext)` command line tools, which offer helpful commands to build Firefox add-ons. These are a few of the commands:

- `[web-ext lint](https://extensionworkshop.com/documentation/develop/web-ext-command-reference/#web-ext-lint)` reviews the code and shows any errors or warnings.

- `[web-ext run](https://extensionworkshop.com/documentation/develop/web-ext-command-reference/#web-ext-run)` runs the extension in a sandbox Firefox window. There are additional flags with this command such as `devtools` and `logging`.

- `[web-ext sign](https://extensionworkshop.com/documentation/develop/web-ext-command-reference/#web-ext-sign)` signs the extension, which I'll explain in more detail in the next step.

After updating the manifest.json successfully, I ran the `web-ext run` command in the terminal, which opened up a new window. I then went to Reddit, where I noticed a notification asking me to enable the browser extension on reddit.com, which I did. After refreshing the page, I was able to confirm that the code was working and redirecting to old.reddit.com.

#### Step 5: Signing the Firefox Add-On

Now that my extension was working, I thought I could load the extension into Firefox and that would be it, similar to how extensions work in Chrome. However, [Firefox add-ons must be signed](https://wiki.mozilla.org/Add-ons/Extension_Signing) in order for them to be loaded in the browser by default. As part of this process, each add-on is also added to Mozilla's directory, even if they're unlisted. This differs considerably from Google Chrome, but does make it more difficult for someone to create and distribute a malicious extension because first they would need to create a developer account.

I created a Mozilla developer account [here](https://accounts.firefox.com/) and then generated an [API key and secret](https://addons.mozilla.org/en-US/developers/addon/api/key/), both of which are needed to sign the extension. I then ran the command `web-ext sign` with the following flags:
`--channel`, which can be `listed` or `unlisted` depending on whether or not the extension should be listed publicly or available for [self-distribution](https://extensionworkshop.com/documentation/publish/self-distribution/) only, `--api-key`, and `--api-secret`.

Once it was signed successfully, Firefox generated a `web-ext-artifacts` folder, in which there was a new file with an `xpi` extension.

#### Step 6: Installing the Firefox Add-On

Now that I had the `xpi` file, I was ready to install the extension. I typed `about:addons` in the URL to get to the add-ons manager page. On the top right was a gear icon, which showed an additional menu once clicked. One of the menu options allowed me to install an add-on from a file, which I used to upload the `xpi` file.

#### Step 7: Enabling Permissions

Even if URLs are listed in the `host_permissions` key in the manifest.json file, [they're not enabled by default in Firefox](https://discourse.mozilla.org/t/porting-chrome-add-on-firefox-page-permissions-not-granted-by-default/112873/8). Instead, users need to navigate to those URLs, click on the extension, and enable the permissions for those pages directly.

For example, for the Reddit to Old Reddit extension, a user would go to reddit.com, click the extension icon in the top right, and allow the extension to either run when they click the extension or on all pages on reddit.com. When a user refreshes the page, the extension will run as normal.

#### Main Takeaways:

Now that I've ported a few Chrome extensions to Firefox, I think there are a few issues to keep in mind.

- Firefox and Chrome don't fully support one another, so there are warning and error messages on both sides even though the extensions still run.

- Chrome and Firefox sometimes expect different formats for the same inputs (ex. `extension_ids`).

- Chrome does not recognize the `browser` object, whereas [Firefox recognizes both `browser` and `chrome`,](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/Chrome_incompatibilities#firefox_supports_both_the_chrome_and_browser_namespaces) so when porting to Firefox developers will likely need to use the `chrome` API namespace until Chrome offers support for the `browser` namespace.

- Firefox requires different permissions for the `browser` API namespace than Chrome does for the `chrome API` namespace (ex. requiring the `tabs` permission when Chrome does not).

All in all, once I had the steps down, porting each extension to Firefox was pretty straightforward - but figuring this out took a long time because the docs for both browsers seemed to be a bit unclear in this area. I hope that as we move closer to June 2024 Chrome will offer more support for cross-browser extensions and APIs so porting Chrome extensions to Firefox is easier for developers in the future.

---

Want more articles on browser extension development delivered straight to your inbox? Subscribe below!
