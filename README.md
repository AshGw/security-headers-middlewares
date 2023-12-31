### Server 

First you might want to yeet the `Server` & `X-Powered-By` headers (the latter being unofficial), preferably through the server configs, or check `no_srv.go`.

> *Security through obscurity ain’t all that chief, but it's just a good complementary security measure.*

  

To test that you've removed this entirely you can use curl:

```bash

➜  ~ curl -IsL https://https://www.google.com/ | grep -i server

server: gws

server: gws

```

Google doesn't hide this info, but guess who does ?

The feds

```bash

➜  ~ curl -IsL https://cia.gov/ | grep -i server

➜  ~

```

  

### Content Security Policy

  

  Don't use `unsafe-inline` directive.

  Use a nonce

```http

Content-Security-Policy: script-src 'nonce-r4nD0mV41u3';

```

  

If you're using `NextJS` Here's how you set the scripts

  

```Typescript

import Script from 'next/script'

export default function Page() {

  const nonce = "" // get it depending on the nature of the app

  return (

    <Script

      src="https://www.googletagmanager.com/gtag/js"

      strategy="afterInteractive"

      nonce={nonce}

    />

  )

}

```

`NextJS v13+` has a server implementation for homogeneous apps view  *[ref](https://nextjs.org/docs/app/building-your-application/configuring/content-security-policy)*

  
  
  

#### Directives reminder

- `base-uri`  Restrict  `base-uri` to block the injection of <base> tags. This prevents attackers from changing the locations of scripts loaded from relative URLs.

- `img-src` you might add trusted sources

```http

img-src 'self' blob: data: https://i.imgur.com https://fontawesome.com/;

```

- `font-src` you might add some trusted sources, or just download them already why bother, anyways here's an example

  

```http

font-src https://fonts.gstatic.com

```

Here's a good *[ref](https://infosec.mozilla.org/guidelines/web_security.html#implementation-notes)*

#### Test First

use  `Content-Security-Policy-Report-Only` . If it's all good, push the **CSP**  changes to prod.

  

### X-Content-Type-Options

View the original **Microsoft** *[ref](https://learn.microsoft.com/en-us/archive/blogs/ie/ie8-security-part-vi-beta-2-update)*  over why they had it promoted.

  

```Go

w.Header().Set("X-Content-Type-Options", "nosniff")

```

  

### X-Frame-Options

This whole header has been obsoleted in favor of `frame-ancestors` directive of the **CSP**

But some older browsers don't support **CSP** so you might as well use this

```Go

w.Header().Set("X-Frame-Options", "DENY")

```

**Note**

> `Content-Security-Policy: frame-ancestors 'none';` ==>  `X-Frame-Options: 'DENY;`

>  `Content-Security-Policy: frame-ancestors 'self';` ==>  `X-Frame-Options: 'SAMEORIGIN;`

  

**Also**

> `ALLOW-FROM` Directive is <u>DEPRECATED</u>  for this header.

  

### X-XSS-Protection

Do not use this so set it to `0`.

```go

w.Header().Set("X-XSS-Protection", "0")

```

Only for ancient browsers that don't support **CSP**.

But if you're somehow trying to support legacy browsers set it to

```http

X-XSS-Protection: 1; mode=block

```

  

### Strict-Transport-Security

Max age of two years is recommended.

Extreme care is needed when setting the `includeSubDomains` flag, as it could disable sites on subdomains that don’t yet have HTTPS enabled.

Secure cookies G.

If you set `preload` attribute after submission to [HSTS](ttps://hstspreload.org) preload list

then switch back to HTTP, you've fumbled the bag big time, [ref](https://hstspreload.org/#removal).

  

```Go

w.Header().Set("Strict-Transport-Security", "max-age=63072000; includeSubDomains, preload")

```

  
  

### Referrer-Policy

View [ref](https://www.w3.org/TR/referrer-policy/#referrer-policy-no-referrer)

  

```Go

w.Header().Set("Referrer-Policy", "no-referrer, strict-origin-when-cross-origin")

```

  

### Cross-Origin-Opener-Policy

Popups and windows

```Go

w.Header().Set("Cross-Origin-Opener-Policy", "same-origin")

```

### Cross-Origin-Resource-Policy

document fetching

```go

w.Header().Set("Cross-Origin-Resource-Policy", "same-origin")

```

  

### Cross-Origin-Embedder-Policy

need to check in with the homie above first

  

```go

w.Header().Set("Cross-Origin-Embedder-Policy", "require-corp")

```

  

### Cache-Control

For a good balance

```go

w.Header().Set("Cache-Control", "no-cache")

```

  

### Feature-Policy

This has been deprecated in favor for `Permissions-policy`

```go

// w.Header().Set("Feature-Policy", "geolocation 'none'; microphone 'none'; camera 'none'")

```

### Permissions-Policy

You can pick and choose which ones to block or activate

View *[ref](https://www.w3.org/TR/permissions-policy/)*

  

```go

w.Header().Set("Permissions-Policy", "accelerometer=(), ambient-light-sensor=(), autoplay=(), battery=(), camera=(), clipboard-read=(), clipboard-write=(), cross-origin-isolated=(), display-capture=(), document-domain=(), encrypted-media=(), execution-while-not-rendered=(), execution-while-out-of-viewport=(), fullscreen=(), gamepad=(), geolocation=(self 'https://trusted.com'), gyroscope=(), magnetometer=(), microphone=(), midi=(), navigation-override=(), payment=(), picture-in-picture=(), publickey-credentials-get=(), screen-wake-lock=(), speaker=(), speaker-selection=(), sync-xhr=(), usb=(), web-share=(), xr-spatial-tracking=()")

```

  

### Other Notes

  

Out in the wild you might find occurrences where

[HPKP](https://tools.ietf.org/html/rfc7469)  is still being used, well it got deprecated in [2018](https://www.chromestatus.com/feature/5903385005916160). It was to be replaced with the CT framework w/ [Expect-CT](https://datatracker.ietf.org/doc/rfc9163/) header, but as of today this header is **obsolete**.

View *[ref](https://chromestatus.com/feature/6244547273687040)*