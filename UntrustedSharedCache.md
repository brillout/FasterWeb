Proposal for an _untrusted shared cache_, a cache for JavaScript code that is deduplicated and cross-origin.
The cache is implementable with JavaScript in the browser.

The **punchline** is two paragraphs bellow.

The cache's characteristics are:
  - it is a browser wide cache that is shared amongst all websites, including websites with a different origin
  - a source code is not identified with a URL but is identified by its secure hash, e.g. its SHA-256 hash

This directly leads to benefits:
  - Versionning included; since two different version of a source code have two different secure hashes, there is no need to manage versions and no need to check for the freshness of cache entries
  - No duplication; two source codes that are identical also have identical hashes. This contrasts to the browser cache where the cache holds several identical copies of a source code because each identical copy is identified with a different URL
  - we can use the secure hash to verify that the source code delivered by the cache is valid 

**punchline**  
This means that once a website loads a source code, i.e. jQuery@2.1.4, then there is no need for another website to re-load jQuery@2.1.4.
The website merely needs to know the secure hash, e.g. the SHA-256 hash, of jQuery@2.1.4.
And that no matter where the source code of jQuery@2.1.4 came from and no matter the origin of the website.



### Workflow

Instead of loading the (source) code from its server, a website would:
  - load the secure hash of the code from its server
  - ask the untrusted shared cache to return the source code corresponding to the secure hash
  - if the untrusted shared cache returns code: the website computes the secure hash of the returned code and checks if it matches the original secure hash. If both secure hashes match, the website then knows that the returned code is valid and no further steps are required
  - if the untrusted shared cache doesn't have the code or if the secure hash of the returned code does not match the original secure hash then the following steps are pursued
  - the website loads the code from its server
  - the website provides the code and its secure hash to the cache
  - the cache saves the code - in a persistent way - and uses the secure hash as identifier for the code
  - further websites, including websites with different origin, may now access the code with its secure hash from the cache

To save Round trips, the website's server should provide all the secure hashes of all the source code that the website will/may need.
ES6 Module Loader makes this rather easy to implement,
see discussion at https://github.com/brillout/FasterWeb/blob/master/UntrustedCDN.md
(see paragraph at https://github.com/brillout/FasterWeb/blame/12e4ad67d284c40848fe09782a280d69564b3acd/UntrustedCDN.md#L30-L44).

### Implementation

An implementation can be achieved based on following components;
 - the `<iframe>` HTML Element
 - `window.postMessage`
 - `window.localStorage`
 - a function `sha256()` that is a JavaScript implementation of SHA-256

`window.localStorage` allows us to persist the JavaScript Object `cache` holding the `cache[sha256(source_code)] = source_code` entries.

The `<iframe>` HTML Element allows us to share the JavaScript Object `cache` across every websites with any origin.

`window.postMessage` allows us to communicate with the iframe in order to get/add code from/to the cache.
There is no limit for the size of the messages and we can therefore pass large sized code through `window.postMessage`.

The function `sha256()` takes a string as input and returns the SHA-256 hash of the string as hexadecimal number.


### Implementation - Solution to 5MB Storage Limit

The spec suggests browser vendors to dedicate 5MB per origin.
A 5MB cache is little.
To circumvent this limit, we can have several iframes with different origins.
A master iframe `<iframe src='master.example.org/cache.html'>`
would then manage several iframes with different origins
`<iframe src='slave1.example.org/cache.html'>`, `<iframe src='slave2.example.org/cache.html'>`, etc.
The number of slave iframes should be minimal as every new slave iframe induces a DNS lookup and a network round trip to retrieve `cache.html`.
Therefore the number of slave iframes should be dynamically increased as storage usage increases.


### Notes
  - The secure hash algorithm needs to be both compact and fast. See experiments of JavaScript SHA-256 implementations at https://github.com/brillout/test-secure-hash-algos
  - Space VS Time; there is a trade off between storage usage and time savings. The strategy of how much storage is used and how cache entries are pruned would be determined after doing experiments

