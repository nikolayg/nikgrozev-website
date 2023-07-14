---
layout: post
title: Solving Progress Indication Issues in Legacy Web Applications
date: 2023-07-14 01:22:09.000000000
type: post
published: true
status: publish
excerpt: This post summarises an approach to add progress indication for a legacy web app.
  This is done globally, without having to rewrite or retest the application.
categories:
  - JavaScript
tags:
  - JavaScript
  - HTML
  - Legacy
---


# Introduction

Recently, I was working on a large legacy web application which didn't have adequate progress feedback and action confirmation.
Users would often click a button, wait for a few seconds and click it again, because they didn't see any indication
that it was clicked. Double clicks resulted in repeated records in the database.

I was tasked to mitigate this across the application, with as little changes to existing screens and logic as possible.
Hence, it had to be done globally with JavaScript by dynamically decorating HTML nodes.

For the impatient, here's a [JSFiddle working example](https://jsfiddle.net/nikgrozev/5x0vd2c9/251/) of the approach. Let's break it down.

# Dynamic Styling of Buttons on Click

Let's start by creating a function, which given a button, adds a style whenever it's clicked. 
We'll mark every button with a data attribute `data-button-registered`
so that we know it's been processed and we don't process it again. 
We'll assume that the class `button-clicked` adds a visual
effect to a clicked button:

```javascript
const clickNotificationTimeout = 500;
const registerNewButton = (button) => {
  // Skip if already processed
  if (!button || button.getAttribute("data-button-registered" === "true")) {
    return;
  }

  // Callback to enable, disable after timeout
  const onBtnClick = () => {
    button.classList.add("button-clicked");
    setTimeout(() => {
      button.classList.remove("button-clicked");
    }, clickNotificationTimeout);
  };

  // Add the listener - do not run the listener immediately as a courtesy to subsequent events
  button.addEventListener("click", () => setTimeout(onBtnClick, 0), {
    capture: true,
  });

  // mark it as processed
  button.setAttribute("data-button-registered", "true");
};
```

Note that we added `capture: true` to the event listener so that it runs before other non-capture events 
and hence there's less chance it's precented
(see the [options documentation](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener#syntax)).
Also, we're wrapping up the event handler in a `setTimeout` so that we don't block subsequent events. 

# Register All Buttons Dynamically

Now that we have a function to process all buttons, we'll need to call it for every button in the app.
However, our app's using many AJAX calls, which dynamically add/remove HTML content, including buttons. 
Furthermore, DOM elementes are added dynamically with JavaScript. 
Hence, we need to listen to the page loading event 
[DOMContentLoaded](https://developer.mozilla.org/en-US/docs/Web/API/Window/DOMContentLoaded_event) 
and any subsequent DOM change via a [MutationObserver](https://developer.mozilla.org/en-US/docs/Web/API/MutationObserver).
The observer provides the DOM difference on every change, which we would need to process:

```javascript
// Get all buttons on the page after load:
window.addEventListener("DOMContentLoaded", () => {
  const buttons = document.getElementsByTagName("button");
  for (let i = 0; i < buttons.length; i++) {
    registerNewButton(buttons[i]);
  }
});

// Traverse a node recursively to get all the buttons
const getButtonChildren = (node) => {
  if (!node || !node.tagName) {
    return [];
  }
  if (node.tagName.toLowerCase() === "button") {
    return [node];
  }
  const result = [];
  for (let child of Array.from(node.childNodes)) {
    result.push(...getButtonChildren(child));
  }
  return result;
};

// Observe the DOM for changes
const domObserver = new window.MutationObserver(function (mutations) {
  mutations.forEach(function (mutation) {
    // For every change - get the new buttons
    for (let i = 0; i < mutation.addedNodes.length; i++) {
      const buttons = getButtonChildren(mutation.addedNodes[i]);
      for (const button of buttons) {
        registerNewButton(button);
      }
    }
  });
});
domObserver.observe(document, {
  childList: true,
  subtree: true,
});
```

Now the user will see a visual effect on every button click! We can easily extend that to add visual
effects on other HTML element types too.

# Progress Indicator

Although the user can see a visual indication that a button is clicked, they can still double click it by mistake.
Hence, we need a progress indicator layer which blocks user input while an operation is in progress.
The [XMLHttpRequest](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest) and 
[fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) APIs are the main ways to make AJAX and API requests
from the browser. We can overwrite their definitions so that we execute code before and and after every request, which
opens or closes a page spinner.

Another considerations is that HTTP requests can overlap, so we'll need to take care of concurrency. We'll keep
track of how many API requests are active, and we'll keep the spinner open if there's at least one.

Lastly, we also need to show a spinner when navigating to another page for which we'll use the
[beforeunload event](https://stackoverflow.com/questions/10616456/how-to-display-a-loading-dialog-while-navigating-between-pages):


```javascript
let concurrentAPIOperations = 0;
const startSpinner = () => {
  concurrentAPIOperations++;
  if (concurrentAPIOperations === 1) {
    // In the HTML - we'll need a new HTML element which represents the spinner overlay
    // The following code assumes hidden and running-spinner are the appropriate CSS classe
    const spinner = document.getElementById("spinner-id");
    spinner.classList.remove("hidden");
    spinner.classList.add("running-spinner");
  }
};
const stopSpinner = () => {
  concurrentAPIOperations = Math.max(concurrentAPIOperations - 1, 0);
  if (concurrentAPIOperations === 0) {
    // In the HTML - we'll need a new HTML element which represents the spinner overlay
    // The following code assumes hidden and running-spinner are the appropriate CSS classe
    const spinner = document.getElementById("spinner-id");
    spinner.classList.add("hidden");
    spinner.classList.remove("running-spinner");
  }
};

// https://stackoverflow.com/questions/5202296/add-a-hook-to-all-ajax-requests-on-a-page
const _open = XMLHttpRequest.prototype.open;
XMLHttpRequest.prototype.open = function () {
  startSpinner();
  this.addEventListener("loadend", stopSpinner);
  _open.apply(this, arguments);
};

// https://stackoverflow.com/questions/63122604/how-can-i-execute-a-function-every-time-fetch-is-used-in-javascript
const _fetch = window.fetch;
window.fetch = function (...args) {
  startSpinner();
  return Promise.resolve(_fetch.apply(window, args)).finally(stopSpinner);
};

// /https://stackoverflow.com/questions/10616456/how-to-display-a-loading-dialog-while-navigating-between-pages
window.addEventListener("beforeunload", startSpinner);
```

The full example is on [JSFiddle](https://jsfiddle.net/nikgrozev/5x0vd2c9/253/).

# Conclusion and Catches

This approach works pretty well for us and we didn't have to modify the existing code. 
We only had to add a small JavaScript snippet together with the required HTML and CSS.
However, there are a few things to look out for. 

Firstly, if the app has `capture` click event handlers which stop the event propagation, then
our click events may not be triggered. Search in the codebase to see if that happens. 
If there are Ajax/API calls that happen on blur, they may prevent subsequent click events, due to
the extra logic for starting/stopping the spinner - see here for [more details](https://erikmartinjordan.com/onblur-prevents-onclick-react).
[There are workarounds](https://erikmartinjordan.com/onblur-prevents-onclick-react) like opening the 
spinner overlay with a delay or using `onMouseDown` instead of `click`.

