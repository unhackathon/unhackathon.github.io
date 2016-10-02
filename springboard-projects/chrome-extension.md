---
layout: springboard
title: Browser Extensions
published: true
---


# Springboard Project: Browser Extensions

Web browsers are our portal to the internet - we can use them to communicate with each other, search for material we want, shop for products, and so much more. But web browsers aren't perfect. That's where browser extensions come in.

Browser Extensions are pieces of code that allow you to augment your browser with newer features to enhance your web browsing experience. Some common ones in use today include µBlock Origin, Reddit Enhancement Suite, and OneTab.

Extensions, as a concept, began in the late 90s with the Netscape browser, the predecessor to many internet browsers out there. For users to add in much requested functionality, they created the Netscape Plugin Application Programming Interface (NPAPI), which allowed developers to create plugins to augment the web browsing experience. This allowed technologies such as Adobe Flash Player and Java Web Start to take off.

Over time, other browsers began creating their own functionality for integrating into the browser, such as ActiveX for Internet Explorer. Today, NPAPI still exists, but alongside PPAPI (Pepper Plugin Application Programming Interface), Chrome Extensions/WebExtensions, and Safari Extensions.

So what does this guide aim to do? The future of extensions, in the author's opinion, are Chrome Extensions (or, as Mozilla terms, "WebExtensions" -- we'll refer to them as just _Extensions_ here on in), so we'll cover that extension format here, which will work on both Chrome, Firefox, and Microsoft Edge. Safari uses their own extension format (though [they indicate that it is simple to port Chrome extensions to](https://developer.apple.com/library/content/documentation/UserExperience/Conceptual/SafariExtensionsConversionGuide/Chapters/Chrome.html#//apple_ref/doc/uid/TP40009993-CH2-SW1)).

We'll introduce you to the format of the Extensions, walk you through a simple text replacer extension, and finally leave you with resources for you to expand on.

# Extensions Format

Extensions contain a relatively simple format. They contain your source files, and a `manifest.json`. Your source files are plain HTML/CSS/JS, while your `manifest.json` lets the browser know how to call your source files based on user activity.

## manifest.json

Extensions contain vital pieces of metadata so that we know where to start. For example, what do you want to do? What's the name of the extension going to be?

We want to declare what the extension does so that our users know what functionality they're adding. This is declared in a `manifest.json` in your extension root directory. For example, here is one from the [MDN (CC-BY-SA 2.5)](https://developer.mozilla.org/en-US/Add-ons/WebExtensions/Your_first_WebExtension):

```json
{
  "manifest_version": 2,
  
  "name": "Borderify",
  "version": "1.0",
  "description": "Adds a solid red border to all webpages matching mozilla.org.",
  "homepage_url": "http://www.mozilla.org/",

  "icons": {
    "48": "icons/border-48.png"
  },
  
  "browser_action": {
    "default_icon": "icons/icon.png",
    "default_popup": "popup.html",
    "default_title": "Borderify",
  },
  
  "permissions": [
  	"*://*.mozilla.org/*"
  ],

  "content_scripts": [
    {
      "matches": ["*://*.mozilla.org/*"],
      "js": ["borderify.js"]
    }
  ]

}
```

A brief overview of the properties:

* __manifest_version__: Defines what format the `manifest.json` is. Should be __2__.
* __name__: The name of your extension.
* __version__: The version of your extension.
* __description__: A description of your extension. What does it do?
* __homepage_url__: Does your extension have a homepage?
* __icons__: A map of pixel size to image file of the icon for your extension.
* __browser_action__: This places a button of your extension on the taskbar of the browser. When the user clicks it, it activates a popup associated with your extension. This is most commonly useful for configuring user settings, allowing easy enable/disable of your extension, and feedback.
* __permissions__: Chrome requires that you declare what you do with the browser. From using the microphone or webcam to modifying page content, you must declare that in permissions.
* __content_scripts__: Content scripts allow you to execute scripts based on sites that the user visits. For example, the above extension would attempt to place a red border on the page whenever the user visits content from mozilla.org (given the implementation defined in `borderify.js`.

The directory structure can be fluid and based on however you like; however, the `manifest.json` is required and must be in the root directory for your extension.

You can read more about `manifest.json` on the [MDN](https://developer.mozilla.org/en-US/Add-ons/WebExtensions/manifest.json), the [Chrome Developer Portal](https://developer.chrome.com/extensions/manifest), and the [Microsoft Developer Portal](https://developer.microsoft.com/en-us/microsoft-edge/platform/documentation/extensions/api-support/supported-manifest-keys/).

## .xpi, .crx, etc.

When you have an extension that is ready for distribution, you can package it in a `.xpi` for Firefox and a `.crx` for Chrome. For now, we will work in debugging environments, which means we want to skip the packaging process.

* If you use Firefox, you can enable debugging mode [here](https://developer.mozilla.org/en-US/Add-ons/WebExtensions/Temporary_Installation_in_Firefox).
* If you use Chrome, follow the directions [here](https://developer.chrome.com/webstore/get_started_simple#step4) as needed.
* If you use Microsoft Edge, follow the directions [here](https://developer.microsoft.com/en-us/microsoft-edge/platform/documentation/extensions/guides/adding-and-removing-extensions/) as needed.

When it comes time to package for real, you must zip up your extension files and submit them to [addons.mozilla.org](https://addons.mozilla.org/) and [chrome.google.com](https://developer.chrome.com/webstore/get_started_simple#step5). They will create the `.xpi` and `.crx` automatically. Please note that fees may apply, and they are not covered in this tutorial.

Microsoft Edge will allow extensions to be installed through the Windows Store; this feature is not available at the time of writing (1 October 2016).

# Text Replacement

Let's walk through the steps needed to create a text replacement extension.

## Create the initial directory structure

Create a directory wherever you'd like. Let's name it `textreplace-extension`. You can rename it whatever, but we'll refer to this as the _root directory_ from here on in.

## Create the manifest.json

In the root directory, create a manifest.json:

```json
{
  "manifest_version": 2,
  
  "name": "Text Replacement",
  "version": "1.0",
  "description": "Replaces text on the page with other substitute text.",
  "homepage_url": "http://www.unhackathon.org/",

  "icons": {
    "48": "icon.png"
  },
  
  "browser_action": {
    "default_icon": "icon.png",
    "default_title": "Text Replacement",
  },
  
  "permissions": [
  	"*://*/*"
  ],

  "content_scripts": [
    {
      "matches": ["*://*/*"],
      "js": ["replace.js"]
    }
  ]

}
```

From here, we can see that we're creating an extension:

* with the icon `icon.png`
* that creates a button on the Browser Toolbar with the icon `icon.png` and the title `Text Replacement`, and does nothing when clicked.
* with permissions on all websites
* that executes `replace.js` on every site navigated to.

## Add an icon

We don't have `icon.png`, so let's download a sample one (courtesy of MDN). Save [this icon file](https://github.com/mdn/webextensions-examples/raw/master/borderify/icons/border-48.png) as `icon.png` in the root directory.

## Create our content script

We want `replace.js` to run on every website and substitute text, so let's create reject.js. Here, I'm replacing all instances of `force` with `horse` (inspired by [xkcd](http://xkcd.com/1418/)) -- feel free to substitute that with some other variations (like the ones listed on [another xkcd comic](http://xkcd.com/1288/)).

```js
chrome.extension.sendMessage({}, function(response) {
    var readyStateCheckInterval = setInterval(function() {
    if (document.readyState === "complete") {
        clearInterval(readyStateCheckInterval);

        // Walk through all the nodes to get all text nodes.
        // Thanks to: http://stackoverflow.com/a/5904945
        // and also: http://stackoverflow.com/a/9452386
        var textNodes = [];
        (function walkNodeForText(node) {
            if (node) {
                node = node.firstChild;
                while (node != null) {
                    switch (node.nodeType) {
                        // Recurse into elements, documents, and document fragments
                        case 1:
                        case 9:
                        case 11:
                            walkNodeForText(node);
                            break;
                        // Add text nodes to the list of elements we want to modify
                        case 3:
                            textNodes.push(node);
                            break;
                    }

                    node = node.nextSibling;
                }
            }
        })(document.body);

        textNodes.forEach(function(currentVal, index, array) {
            // replace all case-insensitive occurences of 'cloud' with 'butt'.
            currentVal.nodeValue = currentVal.nodeValue.replace(/force/gi, "horse");
        });

    }
    }, 10);
});
```

## Load your extension

Let's test this extension. As above:

* If you're using Firefox, load or reload the extension [as demonstrated on MDN](https://developer.mozilla.org/en-US/Add-ons/WebExtensions/Temporary_Installation_in_Firefox).
* If you're using Chrome, load or reload the extension [as documented](https://developer.chrome.com/webstore/get_started_simple#step4).
* If you're using Microsoft Edge, load or reload the extension [as documented](https://developer.microsoft.com/en-us/microsoft-edge/platform/documentation/extensions/guides/adding-and-removing-extensions/).

If all is successful, the following line should say nothing but horse:

force force horse force horse force force force force force force

# Going Further

This is only a brief overview of Extensions and how powerful they are. You now have a working extension that does text substitution; there are ways to expand it to do options menus, modify the CSS of a page, block text, and more.

We would like to point you towards the following resources for more information:

* [MDN](https://developer.mozilla.org/en-US/Add-ons/WebExtensions)
* [Chrome](https://developer.chrome.com/extensions)
* [Microsoft](https://developer.microsoft.com/en-us/microsoft-edge/platform/documentation/extensions/)
* [Safari](https://developer.apple.com/safari/extensions/)

