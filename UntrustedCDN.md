This is a proposal to solve the following two problems of using a CDN for the delivery of common JavaScript source code like jQuery, AngularJS, or React
  - Problem 1: adding a CDN is adding an entity to trust, i.e. adding trust that the CDN doesn't deliver altered and evil code
  - Problem 2: adding a CDN is adding a Single Point of Failure


The solutions are now presented.

### Solution to Problem 1

The goal here is to make the CDN an entity that does not need to be trusted.

To make sure that a source code provided by an untrusted CDN is valid we:
  - provide the website with the secure hash of the source code
  - use that secure hash to check the validity of the source code copy provided by the CDN

The assumption, that the website is provided with the secure hash of the required source code, can be fulfilled by having the server of the website;
  - precompute the secure hash of the source code
  - send the secure hash along the URL of the source code


##### Implementation

Because we have to compute the secure hash of the CDN's response, the source code has to be loaded with `XMLHttpRequest`.
This requires the CDN to have CORS enabled.
This is the case for the official CDNs of jQuery and AngularJS.
Unfortunately the official CDN for React hasn't CORS enabled.
There are no drawbacks of adding CORS headers to URLs delivering common JavaScript code.
Therefore CDNs not already adding them, will eventually do so.

At first, an implementation may seem cumbersome, but building an implementation based on the new ES6 Module Loader spec is much easier.

An ES6 Module Loader features useful characteristics that we can use:
  - An ES6 Module Loader loads source code asynchronously and makes the source code accessible to JavaScript land which is something we need in order to compute the secure hash
  - The process of loading source code of an ES6 Module Loader is easy to alter and extend
  - ES6 Modules have been designed to be statically analyse-able. This enables the server to know beforehand what source codes are going to be required. The server can therefore precompute the secure hashes.

[SystemJS](https://github.com/systemjs/systemjs) is a production ready module loader built on top of the [ES6 Module Loader pollyfill](https://github.com/ModuleLoader/es6-module-loader).
The Client-Side implementation could therefore be built on top of SystemJS.

[JSPM](https://github.com/jspm/jspm-cli) is a package manager for the frontend built on top of SystemJS.
JSPM features a [Dependency Cache](https://github.com/jspm/jspm-cli/wiki/Production-Workflows#creating-a-dependency-cache) that already implements the computation of the whole dependency tree.
The [JSPM NPM Package](https://www.npmjs.com/package/jspm) exposes the computed dependency tree.
We can therefore reuse the dependency tree for our purpose of precomputing all required secure hashes and delivering them to the website.
The Server-Side implementation could therefore be build on top of JSPM.


The secure hash algorithm needs to be both compact and fast. See experiments of JavaScript SHA-256 implementations at https://github.com/brillout/test-secure-hash-algos.

### Solution to Problem 2

The idea is to fallback to the website's server in case the CDN isn't responsive.

The website owner configures the timeout after which an HTTP GET request to the CDN should be aborted and the website's server used instead.

A low timeout would effectively mean "rely less on the CDN" and a high timeout mean "rely more on the CDN".
A timeout of ~10ms would effectively mean "only get the source code if it's in the cache".

According to the spec, calling the `abort()` method of an instance of `XMLHttpRequest` makes the browser interrupt all communication.
This means that doing a request "for nothing" induces only a small bandwidth waste.

Again, ES6 Module Loader makes this Solution rather easy implementable.
