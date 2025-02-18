
## **Web Cache**

Temporarily stores web resources like pages or images to improve speed and reduce server load.  
**Risks:** Cache poisoning, data leakage, unauthorized access to sensitive data.

## **Cache Keys**

Unique identifiers (e.g., URL, headers) used to locate cached content.  Typically, this includes the URL path and query parameters, but it can also include a variety of other elements like headers and content type.
**Risks:** Key collisions, exposure of sensitive data, serving incorrect user data.

## **Cache Rules**

Define how and when content is cached or bypassed (e.g., `Cache-Control`, `Vary`).  Cache rules are often set up to store static resources, which generally don't change frequently and are reused across multiple pages. Dynamic content is not cached as it's more likely to contain sensitive information, ensuring users get the latest data directly from the server.
**Risks:** Caching sensitive data, serving stale content, misconfigurations causing leaks.


## **Constructing A Basic Web Cache Deception Attack**

1. Identify a target endpoint that returns a dynamic response containing sensitive information. Review responses in Burp, as some sensitive information may not be visible on the rendered page. Focus on endpoints that support the `GET`, `HEAD`, or `OPTIONS` methods as requests that alter the origin server's state are generally not cached.

2. Identify a discrepancy in how the cache and origin server parse the URL path. This could be a discrepancy in how they:
	- Map URLs to resources.
	- Process delimiter characters.
	- Normalize paths.

3. Craft a malicious URL that uses the discrepancy to trick the cache into storing a dynamic response. When the victim accesses the URL, their response is stored in the cache. Using Burp, you can then send a request to the same URL to fetch the cached response containing the victim's data. Avoid doing this directly in the browser as some applications redirect users without a session or invalidate local data, which could hide a vulnerability.

## **Using a cache buster**

While testing for discrepancies and crafting a web cache deception exploit, make sure that each request you send has a different cache key. Otherwise, you may be served cached responses, which will impact your test results.

## **Detecting cached responses**

- The `X-Cache` header provides information about whether a response was served from the cache. Typical values include:
    - `X-Cache: hit` - The response was served from the cache.
    - `X-Cache: miss` - The cache did not contain a response for the request's key, so it was fetched from the origin server. In most cases, the response is then cached. To confirm this, send the request again to see whether the value updates to hit.
    - `X-Cache: dynamic` - The origin server dynamically generated the content. Generally this means the response is not suitable for caching.
    - `X-Cache: refresh` - The cached content was outdated and needed to be refreshed or revalidated.

- The `Cache-Control` header may include a directive that indicates caching, like `public` with a `max-age` higher than `0`. Note that this only suggests that the resource is cacheable. It isn't always indicative of caching, as the cache may sometimes override this header.


## **Exploiting Static Extension Cache Rules**

Cache rules often target static resources by matching common file extensions like `.css` or `.js`. This is the default behavior in most CDNs.

If there are discrepancies in how the cache and origin server map the URL path to resources or use delimiters, an attacker may be able to craft a request for a dynamic resource with a static extension that is ignored by the origin server but viewed by the cache.

## **Path Mapping Discrepancies**

`http://example.com/user/123/profile/wcd.css`

- An origin server using REST-style URL mapping may interpret this as a request for the `/user/123/profile` endpoint and returns the profile information for user `123`, ignoring `wcd.css` as a non-significant parameter.

- A cache that uses traditional URL mapping may view this as a request for a file named `wcd.css` located in the `/profile` directory under `/user/123`. It interprets the URL path as `/user/123/profile/wcd.css`. If the cache is configured to store responses for requests where the path ends in `.css`, it would cache and serve the profile information as if it were a CSS file.

To test how the origin server maps the URL path to resources, add an arbitrary path segment to the URL of your target endpoint. If the response still contains the same sensitive data as the base response, it indicates that the origin server abstracts the URL path and ignores the added segment. For example, this is the case if modifying `/api/orders/123` to `/api/orders/123/foo` still returns order information.

To test how the cache maps the URL path to resources, you'll need to modify the path to attempt to match a cache rule by adding a static extension. For example, update `/api/orders/123/foo` to `/api/orders/123/foo.js`. If the response is cached, this indicates:

- That the cache interprets the full URL path with the static extension.
- That there is a cache rule to store responses for requests ending in `.js`.

You can then craft a URL that returns a dynamic response that is stored in the cache.

## **Delimiter Discrepancies**

Delimiters specify boundaries between different elements in URLs. The use of characters and strings as delimiters is generally standardized. For example, `?` is generally used to separate the URL path from the query string. However, as the URI RFC is quite permissive, variations still occur between different frameworks or technologies.

You may be able to use a delimiter discrepancy to add a static extension to the path that is viewed by the cache, but not the origin server. To do this, you'll need to identify a character that is used as a delimiter by the origin server but not the cache.

Firstly, find characters that are used as delimiters by the origin server. Start this process by adding an arbitrary string to the URL of your target endpoint. For example, modify `/settings/users/list` to `/settings/users/listaaa`. You'll use this response as a reference when you start testing delimiter characters.

If the response is identical to the original response, this indicates that the request is being redirected. You'll need to choose a different endpoint to test.

Next, add a possible delimiter character between the original path and the arbitrary string, for example `/settings/users/list;aaa`:

- If the response is identical to the base response, this indicates that the `;` character is used as a delimiter and the origin server interprets the path as `/settings/users/list`.
- If it matches the response to the path with the arbitrary string, this indicates that the `;` character isn't used as a delimiter and the origin server interprets the path as `/settings/users/list;aaa`.

Once you've identified delimiters that are used by the origin server, test whether they're also used by the cache. To do this, add a static extension to the end of the path. If the response is cached, this indicates:

- That the cache doesn't use the delimiter and interprets the full URL path with the static extension.
- That there is a cache rule to store responses for requests ending in `.js`.

Make sure to test all ASCII characters and a range of common extensions, including `.css`, `.ico`, and `.exe`. [Web cache deception delimiter list](https://portswigger.net/web-security/web-cache-deception/wcd-lab-delimiter-list)

Differences in which delimiter characters are decoded by the cache and origin server can result in discrepancies in how they interpret the URL path, even if they both use the same characters as delimiters. Consider the example `/profile%23wcd.css`, which uses the URL-encoded `#` character:

- The origin server decodes `%23` to `#`. It uses `#` as a delimiter, so it interprets the path as `/profile` and returns profile information.
- The cache also uses the `#` character as a delimiter, but doesn't decode `%23`. It interprets the path as `/profile%23wcd.css`. If there is a cache rule for the `.css` extension it will store the response.

Thus, You may be able to exploit a decoding discrepancy by using an encoded delimiter to add a static extension to the path that is viewed by the cache, but not the origin server.

## **Exploiting static directory cache rules**

### Normalization Discrepancies

Normalization involves converting various representations of URL paths into a standardized format.

Discrepancies in how the cache and origin server normalize the URL can enable an attacker to construct a path traversal payload that is interpreted differently by each parser. Consider the example `/static/..%2fprofile`:

- An origin server that decodes slash characters and resolves dot-segments would normalize the path to `/profile` and return profile information.
- A cache that doesn't resolve dot-segments or decode slashes would interpret the path as `/static/..%2fprofile`. If the cache stores responses for requests with the `/static` prefix, it would cache and serve the profile information.

#### Detecting normalization by the origin server

To test how the origin server normalizes the URL path, send a request to a non-cacheable resource with a path traversal sequence and an arbitrary directory at the start of the path. To choose a non-cacheable resource, look for a non-idempotent method like `POST`. For example, modify `/profile` to `/aaa/..%2fprofile`:

- If the response matches the base response and returns the profile information, this indicates that the path has been interpreted as `/profile`. The origin server decodes the slash and resolves the dot-segment.
- If the response doesn't match the base response, for example returning a `404` error message, this indicates that the path has been interpreted as `/aaa/..%2fprofile`. The origin server either doesn't decode the slash or resolve the dot-segment.

#### Detecting normalization by the cache server

Choose a request with a response that contains evidence of being cached. For example, `/aaa/..%2fassets/js/stockCheck.js`:

- If the response is no longer cached, this indicates that the cache isn't normalizing the path before mapping it to the endpoint. It shows that there is a cache rule based on the `/assets` prefix.
- If the response is still cached, this may indicate that the cache has normalized the path to `/assets/js/stockCheck.js`.

You can also add a path traversal sequence after the directory prefix. For example, modify `/assets/js/stockCheck.js` to `/assets/..%2fjs/stockCheck.js`:

- If the response is no longer cached, this indicates that the cache decodes the slash and resolves the dot-segment during normalization, interpreting the path as `/js/stockCheck.js`. It shows that there is a cache rule based on the `/assets` prefix.
- If the response is still cached, this may indicate that the cache hasn't decoded the slash or resolved the dot-segment, interpreting the path as `/assets/..%2fjs/stockCheck.js`.

#### Exploiting normalization by the origin server

If the origin server resolves encoded dot-segments, but the cache doesn't, you can attempt to exploit the discrepancy by constructing a payload according to the following structure:

```
/<static-directory-prefix>/..%2f<dynamic-path>`
```

#### Exploiting normalization by the cache server

If the cache server resolves encoded dot-segments but the origin server doesn't, you can attempt to exploit the discrepancy by constructing a payload according to the following structure:

```
/<dynamic-path>%2f%2e%2e%2f<static-directory-prefix>
```

In this situation, path traversal alone isn't sufficient for an exploit. For example, consider how the cache and origin server interpret the payload `/profile%2f%2e%2e%2fstatic`:

- The cache interprets the path as: `/static`
- The origin server interprets the path as: `/profile%2f%2e%2e%2fstatic`

The origin server is likely to return an error message instead of profile information.

To exploit this discrepancy, you'll need to also identify a delimiter that is used by the origin server but not the cache. Test possible delimiters by adding them to the payload after the dynamic path:

- If the origin server uses a delimiter, it will truncate the URL path and return the dynamic information.
- If the cache doesn't use the delimiter, it will resolve the path and cache the response.
## **Reference**

https://portswigger.net/web-security/web-cache-deception