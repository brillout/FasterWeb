This proposal's goal is to reduce the average time necessary for loading common JavaScript code like jQuery, AngularJS, React, etc.

The goal is achieved by
 - solving the problems associated with using a CDN
 - introducing an `untrusted shared cache`

Solving the CDN problems increases the incentive of using CDNs which have low latency and high bandwidth.

The proposed untrusted shared cache delivers source code that, in certain situations, would normally require the loading of the source code over the network.
It can be implemented solely with web technologies such as `<iframe>` and `window.localStorage`.

Creating a new issue if you think that you found a design flaw will be greatly appreciated.



### The Problems

Drawbacks of using a CDN are:
  - Problem 1: adding a CDN is adding an entity to trust, i.e. adding trust that the CDN doesn't deliver altered and evil code
  - Problem 2: adding a CDN is adding a Single Point of Failure

Problem 1. is not to be underestimated considering the possibility of a CDN being hacked and governmental intervenience.

Because of these two drawbacks many website owners do not make use of a CDN.
This in turn leads to a third problem;
  - Problem 3: duplication of source code in the browser cache, that is, the browser cache holds several identical copies of a source code because each identical copy is identified with a different URL

This is especially inefficient considering that a website may need to use the network to load a source code even though an identical copy is already cached.


We propose solutions to all three problems.
These solutions can be implemented with JavaScript and in all current browsers.



### The Solutions

The solutions are based on the following;

  - Solution to Problem 1: a secure hashing algorithm, e.g. SHA-256
  - Solution to Problem 2: a fallback system in case the CDN is down
  - Solution to Problem 3: an untrusted shared cache

The untrusted shared cache reuses the computed secure hashes from the Solution to Problem 1.



### Solution to Problem 1

The goal here is to make the CDN an entity that doesn't need to be trusted.

To make sure that a source code provided by an untrusted CDN is valid we:
  - provide the website with the secure hash of the source code
  - use that secure hash to check the validity of the source code copy provided by the CDN

The assumption, that the website is provided with the secure hash of the required source code, can be fulfilled by having the server of the website;
  - precompute the secure hash of the source code
  - send the secure hash along the URL of the source code

For organisations trusting the CDN this may seem unnecessary and even represent a drawback since the computation of the secure hash requires computation time and therefore induces a delay.
That said, it is to be considered that the computation of the secure hash is also useful for the Solution to Problem 3.
Since we need the secure hash for the Solution to Problem 3 anyways, we can consider this Solution to be "computationally for free".



### Implementation of Solution to Problem 1

Because we have to compute the secure hash of the CDN's response, the source code has to be loaded with `XMLHttpRequest`.
This requires the CDN to have CORS enabled.
This is the case for the official CDNs of jQuery and AngularJS.
Unfortunately the official CDN for React hasn't CORS enabled.
There are no drawbacks of adding CORS headers to URLs delivering common JavaScript code.
Therefore CDNs not already adding them, will eventually do so.

At first, an implementation may seem cumbersome, but building an implementation based on the new ES6 Module Loader spec is much easier.

An ES6 Module Loader features useful characteristics that we can use:
  - An ES6 Module Loader loads source code asynchronously and makes the source code accessible to JavaScript land which is something we need to compute the secure hash
  - The process of loading source code of an ES6 Module Loader is easy to alter and extend
  - ES6 Module Loaders have been designed to be statically analyse-able. This enables the server to know beforehand what source codes are going to be required. Hence we can precompute the secure hashes.

[SystemJS](https://github.com/systemjs/systemjs) is a production ready module loader built on top of a ES6 Module Loader [pollyfill](https://github.com/ModuleLoader/es6-module-loader).

[JSPM](https://github.com/jspm/jspm-cli) is a package manager for the frontend built on top of SystemJS.
JSPM features a [Dependency Cache](https://github.com/jspm/jspm-cli/wiki/Production-Workflows#creating-a-dependency-cache) that already implements the computation of the whole dependency tree.

Additionally a JSPM extension could be written to compute and deliver the secure hashes to the website.


### Solution to Problem 2

The idea is to fallback to the website's server in case the CDN isn't responsive.

The website owner could configure the timeout after which the HTTP GET request to the CDN should be aborted and the website's server used instead.

A low timeout would effectively mean "rely less on the CDN" and a high timeout mean "rely more on the CDN".
A timeout of ~10ms would effectively mean "only get the source code if it's in the cache".

According to the spec, calling the `abort()` method of an instance of `XMLHttpRequest` makes the browser interrupt all communication.
This means that doing a request "for nothing" induces only a small bandwidth waste.

Again, ES6 Module Loader makes this Solution rather easy implementable.


### Solution to Problem 3

The idea is to have an _untrusted shared cache_.

It is a cache where instead of a resource being identified by an URL, a resource is identified by its secure hash.
In our case a resource is a source code.
So, if a website wants to load a source code from the cache, it asks the cache "give me the source code with this secure hash".
Since two identical source code have the same hash, there is no duplication problem.
This contrasts with URLs where several URLs can point to identical source code.

Furthermore, the untrusted shared cache is a cache that is shared across all websites, including websites with different origins.

In total, this solves Problem 3.

How such untrusted shared cache can be build is described in https://github.com/brillout/UntrustedSharedCache

