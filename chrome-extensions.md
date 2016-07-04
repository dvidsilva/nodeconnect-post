# Creating popup Chrome extensions that interact with the DOM

Chrome extensions can be very useful for your projects, it can let your users interact with other sites
and improve their experience. There are some really cool ones, that you might be familiar with, they provide
the users with everything from ad-blocking, to saving content from a website on Evernote.

Creating Chrome extensions is not very complicated, unfortunately the documentation is not very clear
with some of the details, there are many options and possibilities, and reading through them can be
overwhelming; in this post I will explain the structure of a popup exension that changes the text of every link
on a page with a given string.

Download the sample extension from github, as [zip](https://github.com/dvidsilva/chrome-ext-sample/archive/1.0.0.zip)
[clone](https://github.com/dvidsilva/chrome-ext-sample).

First thing, to load an extension that you created or downloaded outside the chrome web store, go to
[chrome://extensions/](chrome://extensions/) and make sure `developer mode` is activated. This will enable
new options, choose `Load unpacked extension...` and select the folder where the extension is located.

The new extension will appear on the list of extensions, when you make changes and want to have the browser
recognize them, reload that page, or click on the `reload` option under the extension name.

### [Files](https://developer.chrome.com/extensions/manifest)

The main file in the extension is called `manifest.json`, it tells chrome what files it needs to load and what do they do,
plus some information about the extension.

The descriptive properties are pretty self-descriptive, so I'm going to explain the others.

```
    "browser_action": {
        "default_icon": "favicon.png",
        "default_popup": "popup.html",
        "default_title": "ChromiePop"
    }
```

When browser action is present, the extension will add a button to the browser, close to the omnibar.
`default_icon` is the icon that will be used to show in the bar. `default_popup` is the HTML file that will
be loaded when the button is pressed. `default_title` is the title element the icon will have, help text
that will be displayed when hovering over the button.

```
    "content_scripts": [{
        "matches": ["<all_urls>"],
        "all_frames": true,
        "js":      ["scripts/content.js"]
    }]
```

[Content scripts](https://developer.chrome.com/extensions/content_scripts) are the javascript files
that can interact with the active page. [`matches`](https://developer.chrome.com/extensions/content_scripts#match-patterns-globs)
is an array of strings, on what urls the content scripts can run, they're match patterns. `all_frames` means
that the content scripts can run in the iframes inside the pages too. `js` references the scripts, and an additional
property `css` will inject stylesheets too.

    "permissions": [
        "activeTab"
    ]

[Permissions](https://developer.chrome.com/extensions/permissions), is an array of strings with the
permissions that the extension will have access to.

### Messages

The HTML and scripts in the extension should be pretty self explanatory, there's an input, a button
and event listeners to use that information, there are two chrome specific things on the files that
I'll talk about here.

In `extension.js`:

```
chrome.tabs.query({active: true, currentWindow: true}, function(tabs) {
        chrome.tabs.sendMessage(tabs[0].id, {data: text}, function(response) {
            $('#status').html('changed data in page');
            console.log('success');
        });
    });
});
```

[`chrome.tabs.query`](https://developer.chrome.com/extensions/tabs#method-query)
allows you to find a tab in the browser, like every chrome extension API method,
it works asynchronically, it receives a function that will be called with the list of tabs retrieved.
With the `activeTab` permission that was requested in the manifest you'll only have access ot the
current active tab the user is at.

Extension.js will execute every time the user clicks on the button,
so every time that you want to interact with the tab you have to request the tab id.

[`chrome.tabs.sendMessage`](https://developer.chrome.com/extensions/tabs#method-sendMessage) will
allow you to pass some data from the popup script to the content scripts.

It takes the following params
`integer tabId, any message, object options, function responseCallback`.
The message can be any value, in this case we're passing an object with a data property that has
the string the links will be changed to.

```
chrome.runtime.onMessage.addListener( function(request, sender, sendResponse) {
    console.log("something happening from the extension");
    var data = request.data || {};
    var linksList = document.querySelectorAll('a');
    [].forEach.call(linksList, function(header) {
        header.innerHTML = request.data;
    });
    sendResponse({data: data, success: true});
});
```
[`onMessage`](https://developer.chrome.com/extensions/runtime#event-onMessage) allows you to add
an event listener to react to the messages send by the popup.

The request is the message data sent by the popup, sender has some information about the extension and
the origin of the message, and sendResponse is the callback that was passed.

### Conclusion

Creating a chrome popup extension to interact with the DOM is quite easy once you get a hold of it,
remember to declare everything correctly in the manifest.json and practice with small incremental changes
when passing messages to communicate from the popupt to the content scripts.
