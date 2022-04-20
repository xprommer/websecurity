# Content Security Policy

A CSP (Content Security Policy) is used to detect and mitigate certain types of website related attacks
like [Cross-site_scripting](https://developer.mozilla.org/en-US/docs/Glossary/Cross-site_scripting), [clickjacking](https://developer.mozilla.org/en-US/docs/Glossary/Clickjacking) and data injections

The implementation is based on an HTTP header called `Content-Security-Policy`.

The HTTP Content-Security-Policy response header allows web site administrators to control resources the user agent is allowed to load for a given page. With a few exceptions, policies mostly involve specifying server origins and script endpoints. This helps guard against cross-site scripting attacks (Cross-site_scripting).

## Syntax

```
Content-Security-Policy: <policy-directive>; <policy-directive>
```

where `<policy-directive>` consists of: `<directive> <value>` with no internal punctuation.

Examples:

Disable unsafe inline/eval, only allow loading of resources (images, fonts, scripts, etc.) over https:

```
Content-Security-Policy: default-src https:
```

Or

```html
<meta http-equiv="Content-Security-Policy" content="default-src https:">
```

Pre-existing site that uses too much inline code to fix but wants to ensure resources are loaded only over HTTPS and to disable plugins:

```
Content-Security-Policy: default-src https: 'unsafe-eval' 'unsafe-inline'; object-src 'none'
```

Do not implement the above policy yet; instead just report violations that would have occurred:

```
Content-Security-Policy-Report-Only: default-src https:; report-uri /csp-violation-report-endpoint/
```

## Directives

### Fetch directives

Fetch directives control the locations from which certain resource types may be loaded.
CSP fetch directives are used in a `Content-Security-Policy` header and control locations from which certain resource types may be loaded. For instance, `script-src` allows developers to allow trusted sources of script to execute on a page, while `font-src` controls the sources of web fonts.

All fetch directives fall back to `default-src`. That means, if a fetch directive is absent in the CSP header, the user agent will look for the `default-src` directive.

#### `child-src`

Defines the valid sources for web workers and nested browsing contexts loaded using elements
such as `<frame>` and `<iframe>`.

!!!!!! Warning: Instead of `child-src`, if you want to regulate nested browsing contexts and workers, you should use the `frame-src` and `worker-src` directives, respectively. !!!!!!!!

#### `connect-src`

Restricts the URLs which can be loaded using script interfaces

The APIs that are restricted are:
- `<a>` ping,
- fetch(),
- XMLHttpRequest,
- WebSocket,
- EventSource, and
- Navigator.sendBeacon().

`ping` - A space-separated list of URLs. When the link is followed, the browser will send `POST` requests with the body `PING` to the URLs. Typically for tracking.

!!!!!! Note: `connect-src` 'self' does not resolve to websocket schemes in all browsers,
more info in this [issue](https://github.com/w3c/webappsec-csp/issues/7).

Given this CSP header:

```
Content-Security-Policy: connect-src https://example.com/
```

The following connections are blocked and won't load:

```html
<a ping="https://not-example.com">

<script>
  var xhr = new XMLHttpRequest();
  xhr.open('GET', 'https://not-example.com/');
  xhr.send();

  var ws = new WebSocket("https://not-example.com/");

  var es = new EventSource("https://not-example.com/");

  navigator.sendBeacon("https://not-example.com/", { ... });
</script>
```

#### `default-src`

Serves as a fallback for the other fetch directives.

#### `font-src`

Specifies valid sources for fonts loaded using `@font-face`.

#### `frame-src`

Specifies valid sources for nested browsing contexts loading using elements such as `<frame>` and `<iframe>`.

#### `img-src`

Specifies valid sources of images and favicons.

Given this CSP header:

```
Content-Security-Policy: img-src https://example.com/
```

The following `<img>` is blocked and won't load:

```html
<img src="https://not-example.com/foo.jpg" alt="example picture">
```

#### `manifest-src`

Specifies valid sources of application manifest files.

#### `media-src`

Specifies valid sources for loading media using the `<audio>` , `<video>` and `<track>` elements.

#### `object-src`

Specifies valid sources for the `<object>`, `<embed>`, and `<applet>` elements.

!!!!!!!! Note: Elements controlled by `object-src` are perhaps coincidentally considered legacy HTML elements and are not receiving new standardized features (such as the security attributes `sandbox` or `allow` for `<iframe>`). Therefore it is recommended to restrict this fetch-directive (e.g., explicitly set `object-src 'none'` if possible).

#### `prefetch-src`

Specifies valid sources to be prefetched or prerendered.

#### `script-src`

Specifies valid sources for JavaScript.
This includes not only URLs loaded directly into `<script>` elements, but also things like inline script event handlers (`onclick`) and `XSLT` stylesheets which can trigger script execution.

```
Content-Security-Policy: script-src <source>;
Content-Security-Policy: script-src <source> <source>;
```

Given this CSP header:

```
Content-Security-Policy: script-src https://example.com/
```

the following script is blocked and won't be loaded or executed:

```html
<script src="https://not-example.com/js/library.js"></script>
```

Note that inline event handlers are blocked as well:

```html
<button id="btn" onclick="doSomething()">
```

You should replace them with `addEventListener` calls:

```javascript
document.getElementById("btn").addEventListener('click', doSomething);
```

###### Unsafe inline script

!!!!!!!! Note: Disallowing inline styles and inline scripts is one of the biggest security wins CSP provides. However, if you absolutely have to use it, there are a few mechanisms that will allow them.

To allow inline scripts and inline event handlers, `'unsafe-inline'`, a `nonce-source` or a `hash-source` that matches the inline block can be specified.

```
Content-Security-Policy: script-src 'unsafe-inline';
```

The above Content Security Policy will allow inline `<script>` elements

```html
<script>
  var inline = 1;
</script>
```

You can use a `nonce-source` to only allow specific inline script blocks:

```
Content-Security-Policy: script-src 'nonce-2726c7f26c'
```

you will have to set the same `nonce` on the `<script>` element:

```html
<script nonce="2726c7f26c">
  var inline = 1;
</script>
```

Alternatively, you can create hashes from your inline scripts. CSP supports `sha256`, `sha384` and `sha512`.

```
Content-Security-Policy: script-src 'sha256-B2yPHKaXnvFWtRChIbabYmUBFZdVfKKXHbWtWidDVF8='
```

When generating the hash, don't include the `<script>` tags and note that capitalization and whitespace matter, including leading or trailing whitespace.

```html
<script>var inline = 1;</script>
```

###### Unsafe eval expressions

The `'unsafe-eval'` source expression controls several script execution methods that create code from strings. If `'unsafe-eval'` isn't specified with the `script-src` directive, the following methods are blocked and won't have any effect:

- `eval()`
- `Function()`
- When passing a string literal like to methods like:
  ```javascript
  window.setTimeout("alert(\"Hello World!\");", 500);
  ```
- `setTimeout()`
- `setInterval()`
- `window.setImmediate`
- `window.execScript()` Non-Standard (IE < 11) only

###### strict-dynamic

The `'strict-dynamic'` source expression specifies that the trust explicitly given to a script present in the markup, by accompanying it with a nonce or a hash, shall be propagated to all the scripts loaded by that root script. At the same time, any allowlist or source expressions such as `'self'` or `'unsafe-inline'` will be ignored.
For example, a policy such as `script-src 'strict-dynamic' 'nonce-R4nd0m' https://allowlisted.example.com/` would allow loading of a root script with `<script nonce="R4nd0m" src="https://example.com/loader.js">` and propagate that trust to any script loaded by loader.js, but disallow loading scripts from https://allowlisted.example.com/ unless accompanied by a nonce or loaded from a trusted script.

```
Content-Security-Policy: script-src 'strict-dynamic' 'nonce-someNonce'
```

Or:

```
Content-Security-Policy: script-src 'strict-dynamic' 'sha256-base64EncodedHash'
```

It is possible to deploy `strict-dynamic` in a backwards compatible way,
without requiring user-agent sniffing. The policy:

```
Content-Security-Policy: script-src 'unsafe-inline' https: 'nonce-abcdefg' 'strict-dynamic'
```

will act like `'unsafe-inline' https:` in browsers that support CSP1, `https: 'nonce-abcdefg'` in browsers that support CSP2, and `'nonce-abcdefg' 'strict-dynamic'` in browsers that support CSP3.

#### `style-src`

Specifies valid sources for stylesheets.

Examples
Given this CSP header:

```
Content-Security-Policy: style-src https://example.com/
```

the following stylesheets are blocked and won't load:

```html
<link href="https://not-example.com/styles/main.css" rel="stylesheet" type="text/css" />

<style>
#inline-style { background: red; }
</style>

<style>
  @import url("https://not-example.com/styles/print.css") print;
</style>
```

as well as styles loaded using the `Link` header:

```
Link: <https://not-example.com/styles/stylesheet.css>;rel=stylesheet
```

Inline style attributes are also blocked:

```html
<div style="display:none">Foo</div>
```

As well as styles that are applied in JavaScript by setting the style attribute directly,
or by setting `cssText`:

```javascript
document.querySelector('div').setAttribute('style', 'display:none;');
document.querySelector('div').style.cssText = 'display:none;';
```

!!!!!!! However, styles properties that are set directly on the element's style property
will not be blocked, allowing users to safely manipulate styles via JavaScript:

```javascript
document.querySelector('div').style.display = 'none';
```

These types of manipulations can be prevented by disallowing Javascript via the `script-src` directive.

###### Unsafe inline styles

!!!!!!!! Note: Disallowing inline styles and inline scripts is one of the biggest security wins CSP provides. However, if you absolutely have to use it, there are a few mechanisms that will allow them.

To allow inline styles, `'unsafe-inline'`, a `nonce-source` or a `hash-source` that matches
the inline block can be specified.

```
Content-Security-Policy: style-src 'unsafe-inline';
```

The above Content Security Policy will allow inline styles like the `<style>` element,
and the style attribute on any element:

```html
<style>
  #inline-style { background: red; }
</style>

<div style="display:none">Foo</div>
```

You can use a `nonce-source` to only allow specific inline style blocks:

```
Content-Security-Policy: style-src 'nonce-2726c7f26c'
```

You will have to set the same nonce on the `<style>` element:

```html
<style nonce="2726c7f26c">
  #inline-style { background: red; }
</style>
```

Alternatively, you can create hashes from your inline styles. CSP supports `sha256`, `sha384` and `sha512`.
The binary form of the hash has to be encoded with base64. You can obtain the hash of a string
on the command line via the `openssl` program:

```bash
echo -n "#inline-style { background: red; }" | openssl dgst -sha256 -binary | openssl enc -base64
```

You can use a `hash-source` to only allow specific inline style blocks:

```
Content-Security-Policy: style-src 'sha256-ozBpjL6dxO8fsS4u6fwG1dFDACYvpNxYeBA6tzR+FY8='
```

###### Unsafe style expressions

The `'unsafe-eval'` source expression controls several style methods that create style declarations from strings. If `'unsafe-eval'` isn't specified with the style-src directive, the following methods are blocked and won't have any effect:

- `CSSStyleSheet.insertRule()`
- `CSSGroupingRule.insertRule()`
- `CSSStyleDeclaration.cssText`

### Document directives

Document directives govern the properties of a document or worker environment to which a policy applies.

#### `base-uri`

This restricts the URLs which can be used in a document's `<base>` element.
If this value is absent, then any URI is allowed.

Examples

Since your domain isn't `example.com`, a `<base>` element with its `href` set to `https://example.com` will result in a CSP violation.

```html
<meta http-equiv="Content-Security-Policy" content="base-uri 'self'">
<base href="https://example.com/">

// Error: Refused to set the document's base URI to 'https://example.com/'
// because it violates the following Content Security Policy
// directive: "base-uri 'self'"
```

#### `sandbox`

Enables a sandbox for the requested resource similar to the `<iframe>` `sandbox` attribute.
It applies restrictions to a page's actions including preventing popups, preventing the execution of plugins and scripts, and enforcing a same-origin policy.

Syntax

```
Content-Security-Policy: sandbox;
Content-Security-Policy: sandbox <value>;
```

where `<value>` can optionally be one of the following values:

- `allow-downloads` Allows for downloads after the user clicks a button or link.
- `allow-forms` Allows the page to submit forms. If this keyword is not used, this operation is not allowed.
- `allow-modals` Allows the page to open modal windows.
- `allow-orientation-lock` Allows the page to disable the ability to lock the screen orientation.
- `allow-pointer-lock` Allows the page to use the Pointer Lock API
  [https://developer.mozilla.org/en-US/docs/Web/API/Pointer_Lock_API]
- `allow-popups` Allows popups (like from `window.open`, `target="_blank"`, `showModalDialog`).
  If this keyword is not used, that functionality will silently fail.
- `allow-popups-to-escape-sandbox` Allows a sandboxed document to open new windows without forcing the sandboxing flags upon them. This will allow, for example, a third-party advertisement to be safely sandboxed without forcing the same restrictions upon the page the ad links to.
- `allow-presentation` Allows embedders to have control over whether an iframe can start a presentation session.
- `allow-same-origin` Allows the content to be treated as being from its normal origin. If this keyword is not used, the embedded content is treated as being from a unique origin.
- `allow-scripts` Allows the page to run scripts (but not create pop-up windows). If this keyword is not used, this operation is not allowed.
- `allow-top-navigation` Allows the page to navigate (load) content to the top-level browsing context. If this keyword is not used, this operation is not allowed.

### Navigation directives

Navigation directives govern to which locations a user can navigate or submit a form, for example.

#### `form-action`

Restricts the URLs which can be used as the target of a form submissions from a given context.

!!!!!!!! Warning: Whether `form-action` should block redirects after a form submission is `debated` and browser implementations of this aspect are inconsistent (e.g. Firefox 57 doesn't block the redirects whereas Chrome 63 does)[https://github.com/w3c/webappsec-csp/issues/8]

Examples

Using a `<form>` element with an action set to inline JavaScript will result in a CSP violation.

```html
<meta http-equiv="Content-Security-Policy" content="form-action 'none'">

<form action="javascript:alert('Foo')" id="form1" method="post">
  <input type="text" name="fieldName" value="fieldValue">
  <input type="submit" id="submit" value="submit">
</form>

// Error: Refused to send form data because it violates the following
// Content Security Policy directive: "form-action 'none'".
```

#### `frame-ancestors`

Specifies valid parents that may embed a page using `<frame>`, `<iframe>`, `<object>`, `<embed>`, or `<applet>`

Setting this directive to `'none'` is similar to `X-Frame-Options`: deny (which is also supported in older browsers).

```
Content-Security-Policy: frame-ancestors <source>;
Content-Security-Policy: frame-ancestors <source> <source>;
```

!!!!!!!!! Note: The `frame-ancestors` directive's syntax is similar to a source list of other directives (e.g. `default-src`), but doesn't allow `'unsafe-eval'` or `'unsafe-inline'` for example. It will also not fall back to a `default-src` setting.

###### Only the sources listed below are allowed:

- `<host-source>` Internet hosts by name or IP address, as well as an optional URL scheme
  and/or port number, separated by spaces.

  Examples:

  - `http://*.example.com` Matches all attempts to load from any subdomain of `example.com` using the `http:` URL scheme.
  - `mail.example.com:443` Matches all attempts to access port 443 on `mail.example.com`
  - `https://store.example.com` Matches all attempts to access store.example.com using `https:`

!!!!!!!! Warning: If no URL scheme is specified for a `host-source` and the iframe is loaded from an `https` URL, the URL for the page loading the iframe must also be `https`, per the `Does URL match expression in origin with redirect count?` section of the CSP spec[https://w3c.github.io/webappsec-csp/#match-url-to-source-expression]

- `<scheme-source>` A scheme such as `http:` or `https:`. The colon is required and scheme should not be quoted. You can also specify data schemes (not recommended).

  - `data:` Allows `data: URIs` to be used as a content source. _This is insecure; an attacker can also inject arbitrary data: URIs. Use this sparingly and definitely not for scripts._
  - `mediastream:` Allows `mediastream: URIs` to be used as a content source.
  - `blob:` Allows `blob: URIs` to be used as a content source.
  - `filesystem:` Allows `filesystem: URIs` to be used as a content source.

- `'self'` Refers to the origin from which the protected document is being served, including the same URL scheme and port number. You must include the single quotes. Some browsers specifically exclude `blob` and `filesystem` from source directives. Sites needing to allow these content types can specify them using the Data attribute.

- `'none'` Refers to the empty set; that is, no URLs match. The single quotes are required.

### Other directives

#### `require-sri-for`

Requires the use of SRI (Subresource Integrity) for scripts or styles on the page.
!!!!!!!! Deprecated: This feature is no longer recommended. Though some browsers might still support it, it may have already been removed from the relevant web standards, may be in the process of being dropped, or may only be kept for compatibility purposes. Avoid using it, and update existing code if possible; see the compatibility table at the bottom of this page to guide your decision. Be aware that this feature may cease to work at any time.

## Multiple content security policies

The CSP mechanism allows multiple policies being specified for a resource, including via the `Content-Security-Policy` header, the `Content-Security-Policy-Report-Only` header and a `<meta>` element.

You can use the `Content-Security-Policy` header more than once, as in the example below. Pay special attention to the `connect-src` directive here. Even though the second policy would allow the connection, the first policy contains `connect-src 'none'`. Adding additional policies can only further restrict the capabilities of the protected resource, which means that there will be no connection allowed and, as the strictest policy, `connect-src 'none'` is enforced.

```
Content-Security-Policy: default-src 'self' http://example.com;
                          connect-src 'none';
Content-Security-Policy: connect-src http://example.com/;
                          script-src http://example.com/
```
