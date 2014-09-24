
# Cross-Origin Service Workers (DRAFT)

This proposal is an extension of [Service Workers](http://slightlyoff.github.io/ServiceWorker/spec/service_worker/)
which allows pages to access service workers registered by pages in different origins.
This "Cross-Origin Service Workers" (COSW) proposal aims to be unified into the Service Workers standard.

Note: This is a retake of @dglazkov's [Tubes](https://github.com/dglazkov/tubes) proposal.

## Motivation

Service Workers standard doesn't allow third-parties to control its data.
For example, it is not possible to build "Facebook SDK with offline support" with allows offline-like with sync support, authentication without redirect, etc.

This is because there is no way to provide for these third parties to provide service workers.
The only way to do is let first party worker to handle all of data access.
Finer level, control is desirable in some cases. This is what "Cross-Origin Service Workers" aims to address.

## Concepts

A *connection* may let pages access to an already installed service worker that has given script URL, even if the worker is not installed by the page.
If the connection requests a script of the same origin as one of the requesting page, it may return a service worker that has requried URL.
Otherwise, the *connection* needs an *acceptance* of the service worker script.

From the service worker's perspective, the connection request is notified through an `connect` event.
Once the service worker *accepted* the connection, a *foreign instance* of a `ServiceWorkerClient` becomes accessible.
A foreign instance of a client lets the worker to access a limited set of information of the connecting page.

## Navigator Extension

`Navigator.connect()` requests a connection to a service worker on given `url`.
nn
```
partial interface Navigator {
  Promise<MessagePort> connect(ScalarValueString url, any request);
};
```

 * ISSUE: Specify the algorithm that matches `url` to registered servicveworkers. The URL matching criterial should be consistent with 
   how Service Workers spec decide [controlling](https://slightlyoff.github.io/ServiceWorker/spec/service_worker/#dfn-document-control) worker.
 * ISSUE: List the type of possible errors.
 * ISSUE: We might want to have `ServiceWorker` or its subset so that the page can detect workers being gone.
 * QUESTION: Should the return value `MessagePort` or something more indirect like `ServiceWorkerRegistration` ?
 * QUESTION: Do we need the `request` parameter or should be just let peers to use given `MessagePort`?
   It adds extra complexity. On the other hand, it will expose the fact that the user has visited the site if we allow service to be connected without any authentication.
 * QUESTION: Do we need any ways to enumerate connected service workers?


Example: Requesting a connection

```
navigator.connect('//example.com/', { 'apiKey': '...', 'cred': '....' }).then(
  function(port) {
    port.postMessage("Hello!");
  },
  function(why) {
    console.log("Failed to connect example.com because: ", why);
  }
);
```

## ServiceWorkerGlobalScope Extension

A service worker may accept connection request through `onconnect` event.
The `onconnect` event is a lifecycle event of the 'ServiceWorker' which is called when the page invokes `connect()`.
It handles `ConnectionEvent`, a subtype of [ExtendableEvent](https://slightlyoff.github.io/ServiceWorker/spec/service_worker/#extendable-event-interface). 

```
interface ForeignServiceWorkerClient {
  readonly attribute ScalarValueString url;
  readonly attribute MessagePort port;
};

interface ConnectionEvent extends ExtendableEvent {
  readonly attribute ForeignServiceWorkerClient client;
  readonly attribute any request;
};

partial interface ServiceWorkerGlobalScope {
  // event
  attribute EventHandler onconnect;
};
```

Example: Accepting a connection

```
onconnect = function(e) {
  e.waitUntil(new Promise(resolve, reject) {
    var req = new XMLHttpRequest();
    req.open('GET', myBuildQuery({ pageURL: e.client.url, apiKey: e.request.apiKey, credentials: e.request.cred }));
    req.onload = function() {
      if (req.status != 200) {
        return reject(Error("Some Error"));
      }

      resolve(); // The resolved value doesn't matter.
      e.client.port.postMessage({ type: 'connected', payload: req.response });
    };

    req.onerror = function() {
      reject(Error("Network Error"));
    };

    req.send();
  });
};
```

 * ISSUE: The lifecycle of Service Worker should be clearer.
   * For example, what happens if the worker suspended/updated? Is re-`connect()` required then?
 * QUESTION; Do we need 'ForeignServiceWorkerClient' or is it sufficient to use `MesssagePort` directly?
 * QUESTION: Do we need any ways to enumerate `ForeignServiceWorkerClient` ?
 * QUESTION: Should `ForeignServiceWorkerClient` be strict subset of `ServiceWorkerClient` ?
 * QUESTION: Should the message arrive to `ForeignServiceWorkerClient.port`, or global `onmessage` handler?
 * QUESTION: Do we need any ways to "disconnect"? Should `ForeignServiceWorkerClient.port.close` work?
   If so, how does the client (connecting page) know that? `MessagePort` apparently doesn't provide such an API.
