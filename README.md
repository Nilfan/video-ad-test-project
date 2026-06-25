# Ad Video Widget

A banner with a video background, offer text, and a CTA button. Embedded with a single `<iframe>` tag — no scripts required on the partner's side.

## How to embed

```html
<iframe
  src="https://your-cdn.com/banner.html?clickLink=https%3A%2F%2Fyour-site.com%2F&utm_source=mysite&partner_id=42"
  width="720"
  height="450"
  style="border:none"
  allow="autoplay"
></iframe>
```

Host `banner.html` and the `assets/` folder on your CDN or server, then put the full URL into the `src` attribute.

## Parameters

| Parameter      | Required | Description                                                          |
| -------------- | -------- | -------------------------------------------------------------------- |
| `clickLink`    | Yes      | The destination URL. Must be URL-encoded via `encodeURIComponent`.   |
| `utm_source`   | No       | Forwarded to `clickLink`.                                            |
| `utm_campaign` | No       | Forwarded to `clickLink`.                                            |
| `partner_id`   | No       | Your partner ID for analytics. Also forwarded to `clickLink`.        |
| anything else  | No       | Any other query params are forwarded to the destination URL as well. |

## Analytics events

The widget sends two events to `ANALYTICS_URL` (constant at the top of the script in `banner.html`):

- `impression` — fired on load
- `click` — fired when the user clicks the banner

Each event payload contains `type`, `timestamp`, `parentPage` (the URL of the page hosting the widget), and `widgetUrl`.

## Design decisions

**Why `sendBeacon` / `fetch keepalive` for click events**

When a user clicks the banner, the browser starts navigating away and can cancel a regular `fetch` before it completes. `sendBeacon` queues the request at the browser level and guarantees delivery even when the page is being unloaded. If `sendBeacon` is not available, we fall back to `fetch` with `keepalive: true`, which has the same intent but is limited to ~64 KB total body size.

**How we get the parent page URL**

The widget runs inside an `iframe`, so reading `window.parent.location` is blocked by the Same-Origin Policy. The browser automatically sets the `Referer` header when loading an iframe, making the parent page URL available as `document.referrer` — no extra code needed from the partner.

**Why the gradient overlay is on the link element**

The entire banner is a single `<a>` tag sitting on top of the video. The bottom gradient is just a `background` on that link. This avoids an extra `div` layer and keeps the z-index stack simple.

## What was intentionally left out

**Nested iframes.** If the partner page itself is inside an iframe, `document.referrer` will return the intermediate frame's URL, not the actual user-facing page. The proper fix is a `postMessage` protocol across frame levels, but that requires code on the partner's side.

**Old browser support.** The code uses `var` and plain functions, but `sendBeacon` and `fetch` are still not available in IE 11 and some early Android WebViews. Adding polyfills was out of scope.

**Retry on network failure.** If the network is down when an event fires, it is silently lost. A proper solution would be a `localStorage` queue flushed by a Service Worker via Background Sync — significantly more complexity.

**Video cropping at extreme aspect ratios.** `object-fit: cover` works well for 16:9-ish slots. In a very narrow or very wide container the main subject of the video will be cropped. Fixing this properly would require multiple video sources selected by a media query inside the iframe.

## Install

You need to install http server:

```bash
npm i -D http-server
```

Then reopen terminal window.

## Running locally

```bash
http-server
# then open http://localhost:8080/src/index.html
```

Opening via `file://` won't work — browsers block autoplay for local files.

# video-ad-test-project
