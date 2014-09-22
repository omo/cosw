
# Cross-Origin Service Workers (DRAFT)

This proposal is an extension of [Service Workers](http://slightlyoff.github.io/ServiceWorker/spec/service_worker/)
which allows pages to access service workers registered by pages in different origins.
This proposal aims to be unified into the Service Workers standard.

## Motivation

Service Workers standard doesn't allow third-parties to control its data.
For example, it is not possible to build "Facebook SDK with offline support" with allows offline-like with sync support, authentication without redirect, etc.

This is because there is no way to provide for these third parties to provide service workers.
The only way to do is let first party worker to handle all of data access.
Finer level, control is desirable in some cases. This is what "Cross-Origin Service Workers" aims to address.

## Concepts

A *discovery* may let pages access to an already installed service worker that has given script URL, even if the worker is not installed by the page.
If the discovery requests a script of the same origin as one of the requesting page, it may return a service worker that has requried URL.
Otherwise, the *discovery* process reuqests a *grant*. Once it is *granted*, it may return a *foreign instance* to the matched service worker
so that a limited set of functionality of the service worker is available to the page.

From the service worker's perspective, a page requesting the access through the discovery is accessible as a *foreign instance*. 
Similar to foreign proxies, a foreign client lets the worker to access a limited set of information of the accessing page.

By default, services workers aren't avaialbe through the discovery process. They become available only if they *publish* themselves.

## ServiceWorkerContainer Extension

With Cross-Origin Service Workers, `ServieWorkerContainer` has`discover()` function to request
to an already registered service worker, including one in a different domain.

```
// This is a ServiceWorkerRegistration equivalent with limited capability.
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

With Cross-Origin Service Workers, each service worker can publish it to pages on other domains though 'publish()',
and also can dismiss its discoverability through 'unpublish()' call.

```
partial interface ServiceWorkerGlobalScope {
  Promise<boolean> publish();
  Promise<boolean> unpublish();
}
```

## Foreign instance of ServiceWorker

Foreign instance of ServiceWorker is a [ServiceWorker](https://slightlyoff.github.io/ServiceWorker/spec/service_worker/#service-worker-interface) whose 
domain is not same as the page. The foreign instance is retrieved by the 'discover()' function.

 * TBD: Some of the API should be inert.

## Foreign instance of ServiceWorkerClient for security reasons

Foreign instance of ServiceWorkerClient is a [ServiceWorkerClient](https://slightlyoff.github.io/ServiceWorker/spec/service_worker/#service-worker-client)
whose domain is not same as the service worker.

 * TBD: Some of the API should be inert for security reasons.

## Design Considerations

 * How and who should give the grant to the page?
 * Should we provide inter-worker communication API?