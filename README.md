This proposal's goal is to reduce the average time necessary for loading common JavaScript code like jQuery, AngularJS, React, etc.


The goal is achieved by
 - solving the problems associated with using a CDN
 - introducing an "untrusted shared cache"

Solving the CDN problems increases the incentive of using CDNs which have low latency and high bandwidth.

The proposed _untrusted shared cache_ delivers source code from cache that, in certain situations, would normally require the loading of the source code over the network.


The proposed solutions can rather easily be implemented thanks to recently created tools and technologies. 
The solutions are compatible with current browsers.
The intended implementation requires almost no work for website owners and web developers.

Creating a new issue if you think that you found a design flaw will be greatly appreciated.



### The Problems

Drawbacks of using a CDN are:
  - Problem 1: adding a CDN is adding an entity to trust, i.e. adding trust that the CDN doesn't deliver altered and evil code
  - Problem 2: adding a CDN is adding a Single Point of Failure

Problem 1. is not to be underestimated considering the possibility of a CDN being hacked and governmental intervenience.

Because of these two drawbacks many website owners do not make use of a CDN.
This in turn leads to a third problem;
  - Problem 3: duplication of source code in the browser cache, that is, the browser cache holds several identical copies of a source code because each identical copy is identified with a different URL

This is especially inefficient considering that a website may need to use the network to load a source code even though an identical copy is already in the cache.


We propose solutions to all three problems.



### The Solutions

The solutions are based on the following;


##### Solution to Problem 1: The source code of the CDN is verified using a secure hashing algorithm, e.g. SHA-256

We verify that the secure hash of the source code delivered by the CDN matches the secure hash of the expected source code.

The secure hash of the source code delivered by the CDN is computed in the browser.  
The secure hash of the required source code is precomputed and provided by the website's server.

Implementation details are described in https://github.com/brillout/FasterWeb/blob/master/UntrustedCDN.md.


##### Solution to Problem 2: a fallback system in case the CDN is down or slow

If the CDN is not able to deliver the required source code in a timely manner, we then fallback by retrieving the source code from the website's server.

Implementation details are described in https://github.com/brillout/FasterWeb/blob/master/UntrustedCDN.md.


##### Solution to Problem 3: an "untrusted shared cache"

The _untrusted shared cache_ is a cache where
  - a source code is identified by its secure hash
  - the cache is shared across all websites, including websites with different origins

That a source code is identified by its secure hash contrasts with the browser cache where a source code is identified with a URL.
So, if a website wants to load a source code from the cache, it asks the cache "give me the source code with this secure hash".
Since two identical source code have the same hash, there is no duplication problem.
This contrasts with URLs where several URLs can point to identical source code.

Since the cache is cross-origin we cannot trust the source code delivered by it.
Similary to Solution to Problem 1 we use the secure hash of the required source code to ensure that the source code delivered by the cache is valid.

How such untrusted shared cache can be build is described in https://github.com/brillout/FasterWeb/blob/master/UntrustedSharedCache.md.
