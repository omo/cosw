
# Cross-Origin Service Worker (DRAFT)

This proposal is an extension of [Service Workers](http://slightlyoff.github.io/ServiceWorker/spec/service_worker/)
which allows pages to access service workers registered by pages in different origins.
This proposal aims to be unified into the Service Workers standard.

## Concepts

A *discovery* may let pages access to an already installed service worker that has given script URL, even if the worker is not installed by the page.
If the discovery requests a script of the same origin as one of the requesting page, it may return a service worker that has requried URL.
Otherwise, the *discovery* process reuqests a *grant*. Once it is *granted*, it may return a *foreign instance* to the matched service worker
so that a limited set of functionality of the service worker is available to the page.

From the service worker's perspective, a page requesting the access through the discovery is accessible as a *foreign instance*. 
Similar to foreign proxies, a foreign client lets the worker to access a limited set of information of the accessing page.

By default, services workers aren't avaialbe through the discovery process. They become available only if they *publish* themselves.

## ServiceWorkerContainer Extension

```
[Exposed=Window]
interface ServiceWorkerDiscovery : EventTarget {
  [Unforgeable] readonly attribute ServiceWorker? active;

  // event
  attribute EventHandler onupdatefound;
};

partial interface ServiceWorkerContainer {
  Promise<ServiceWorkerDiscovery> discover(ScalarValueString scriptURL);
}
```

## ServiceWorkerGlobalScope Extension

```
partial interface ServiceWorkerGlobalScope {
  Promise<boolean> publish();
  Promise<boolean> unpublish();
}
```

## Foreign instance of ServiceWorker

## Foreign instance of ServiceWorkerClient

## Design Considerations

 * How and who should give the grant to the page?
 * Should we provide inter-worker communication API?