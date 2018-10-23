---
layout: post
title: Programmatic Form Submission with Fields and Files
date: 2018-04-18 05:22:09.000000000
type: post
published: true
status: publish
excerpt: 
    In this article, I'll show how to wrap the XMLHttpRequest primitives
    in a simple promise-based function for arbitrary form submission ... 
categories:
- JavaScript
- Node
- TypeScript
tags:
- JavaScript
- Node
- TypeScript
- WebForm
- File Upload
---

<div id='introduction'/>
# Introduction

Recently, I've been working on a file upload from a web app, which needed to report 
the upload progress to the end user. I typically use the browsers' native `fetch` function to
perform API calls to the server. It supports promises and does not require external
dependencies. For the few browsers, which do not implement `fetch`, there are 
convenient polyfills.

However, I couldn't make `fetch` report the progress of a server call. 
I could `await` for a call's promise to finish, but couldn't get intermediate progress updates.
Thus, I had to resolve to a native browser primitive called `XMLHttpRequest`.
Unfortunately, it's quite old school and did not fit well with the rest of project's code
which uses promises. In this article, I'll show how to wrap the `XMLHttpRequest` primitives
in a simple promise-based function for arbitrary form submission.



<div id='solution'/>
# The Solution

The following function implements programmatic form submission with fields and files.
I've used TypeScript to show the types of the parameters. If you are using plain JavaScript,
you'll need to remove the type definitions. The function takes a few params:

- `url` - the server endpoint.
- `fields` - a dictionary/map whose keys are the form fields. The values are either strings, files (e.g. from a file selection event), or blobs.
- `headers` - a dictionary/map whose keys and values represent the headers. This is typically used to authenticate to the server.
- `onprogress` - a callback function called by the browser every now and then to report the upload progress.  

The result of the function call is a promise which resolves with the server response if the form
submission was successful. On failure the promise rejects/fails. Check out the comments in the code below
for more details on the implementation:

```typescript
export function submitForm(
  url: string,
  fields: Map<string, string | File | Blob>,
  headers: Map<string, string>,
  onprogress: (pe: ProgressEvent) => any) {
  // FormData represents the payload of the form
  const form = new FormData();

  // Attach all files and field values
  fields.forEach((val, field) => form.append(field, val));

  // The XHR request/call
  const xhrRequest = new XMLHttpRequest();

  // The "true" parametes makes the request asynchronous
  xhrRequest.open("POST", url, true);

  // Set up the the request headers.
  // Skip "content-type" - it messes up xhr. See - http://www.olioapps.com/blog/formdata-fetch-gotchas/
  headers.forEach((val, h) => h.toLowerCase() !== "content-type" && xhrRequest.setRequestHeader(h, val));

  // Set up the callback for the upload progress
  xhrRequest.upload.onprogress = onprogress;

  // Wrap the result callback in a promise
  const responsePromise = new Promise((resolve, reject) => {
    xhrRequest.onreadystatechange = () => {
      // State 4 means "Done" - https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/readyState
      if (xhrRequest.readyState === 4) {
        if (xhrRequest.status >= 200 && xhrRequest.status <= 299) {
          resolve(xhrRequest.responseText);
        } else {
          reject(xhrRequest.statusText);
        }
      }
    };
  });

  // Kick off the actual form submission
  xhrRequest.send(form);

  // Return the promise so the caller can await
  return responsePromise;
}
``` 

<div id='usage'/>
# Sample Usage

Let's demonstrate how to call the form upload function:

```typescript
// Assuming we have an "onChange" listener on a file input in the HTML
// we can extract the selected File from the event object
const file = e.target.files[0] as File;

// Prepare the headers for authentication
const headers = new Map([['Authorization', 'Bearer secret-token']]);
const fields = new Map([
  ['user', 'john'],
  ['upload', file]
]);

// A callback for the upload progress
const onprogress = (pe: ProgressEvent) => {
  const percent = Math.floor(pe.loaded / pe.total * 100);
  console.log(`${percent} % finished`);
}

// Call the function ..
const submissionPromise = submitForm('http://example.com', fields, headers, onprogress);

// Wait for server call to succeed or fail
submissionPromise
  .then(resp => console.log(resp))
  .catch((e: any) => console.log(e));
```
