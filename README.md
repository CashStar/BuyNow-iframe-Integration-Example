# BuyNow Widget - iframe Embed Page

This repository hosts a small standalone page ([index.html](index.html)) whose only job is to
load the third-party BuyNow Widget (`widget.js`) and expose it to be embedded inside an
`<iframe>` on a client's site. It reads its configuration from the iframe's own URL query
string and talks back to the parent page over `window.postMessage`

## Repository structure

| Path | Purpose |
| --- | --- |
| [index.html](index.html) | The embed page itself — the file a client's `<iframe>` points to. |
| [examples/inline.html](examples/inline.html) | Reference client page: embeds the widget in a fixed-size box on the same page. |
| [examples/modal.html](examples/modal.html) | Reference client page: embeds the widget inside a modal dialog. |

## Prerequisites

- Any simple static file server (examples below use Python's built-in `http.server`, available
  with Python 3 - no project dependencies to install).
- An internet connection: `index.html` loads the actual widget bundle and its assets from a
  third-party CDN (`content.blackhawknetwork.com`), so it won't render anything offline, even
  when served locally.

## Running it locally

The two example pages deliberately load the widget from a **different origin** than they're
served from, to mirror how a real client site embeds this page cross-origin. That means you
need **two servers running at once**, both started from the repository root:

```bash
# Terminal 1 — serves index.html (the widget embed page)
python3 -m http.server 8080
```

```bash
# Terminal 2 — serves the example client pages
python3 -m http.server 5501
```

With both running, open one of the example pages in your browser:

- Inline box example: <http://127.0.0.1:5501/examples/inline.html>
- Modal example: <http://127.0.0.1:5501/examples/modal.html>

> **Important:** the embed page must be reached as `http://localhost:8080` (not
> `http://127.0.0.1:8080`) — both examples hardcode `WIDGET_ORIGIN = "http://localhost:8080"`
> and compare it exactly against incoming `postMessage` events, so the widget server needs to be
> addressed with that exact host. The example pages themselves can be opened via either
> `localhost` or `127.0.0.1`, since they read their own origin dynamically.

## Testing the examples

Both examples use a bundled demo `instanceId`/`orderingToken` already set in `index.html`'s
default config - fine for local testing, but not a real brand; a client's own iframe URL would
pass its own `instanceId`/`orderingToken` as query params instead (see the integration guide).

1. Open either example page as described above.
2. Click **Buy Now**.
   - On [inline.html](examples/inline.html), the widget loads inside the pink dashed box.
   - On [modal.html](examples/modal.html), a dimmed modal opens with the widget inside it.
3. Open the browser console to see `resize`, `onSuccess`, and `onError` events logged as they're
   forwarded from the widget.
4. Close the widget using its own close (✕) control inside the iframe. Both examples listen for
   the widget's `onClose` event to tear down the box/modal — there's no separate close button in
   the example pages themselves.
5. Click **Buy Now** again to confirm it reopens cleanly.

## Changing the local ports

If `5501` are already in use on your machine, you can serve on different port, however we suggest not changing the port used in the widget origin i.e. `8080`