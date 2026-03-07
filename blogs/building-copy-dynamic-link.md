---
title: Building Copy Dynamic Link
date: 2026-03-06
excerpt: How a small workflow frustration turned into a Chrome extension — and what I learned building it accessibly from scratch.
tag: Tooling
readtime: 4
---

## The problem

Every time I was migrating a site I'd right-click a link, hit "Copy Link Address", then have to manually strip the domain before I could use it. Every single time. After doing it a few hundred times I figured there had to be a better way — and when I couldn't find an extension that did exactly what I wanted, I built one.

## What it does

Copy Dynamic Link adds a single item to your right-click context menu: **Copy Dynamic Link**. Click it on any link and you get the relative path copied to your clipboard — no protocol, no domain, just `/path/to/page?query=string#hash`.

It preserves query strings and hash fragments too, which most workarounds miss.

## Building it

The extension itself is straightforward — Manifest V3, a background service worker to register the context menu item, and a small popup explaining what it does. The tricky part was clipboard access.

In Manifest V3, writing to the clipboard from a background service worker isn't allowed directly. The solution is to inject a small script into the active tab using `chrome.scripting.executeScript` and call `navigator.clipboard.writeText()` from there instead, with a `document.execCommand` fallback for restricted pages.

```javascript
function copyToClipboard(text) {
  navigator.clipboard.writeText(text).catch(() => {
    const el = document.createElement('textarea');
    el.value = text;
    el.style.position = 'fixed';
    el.style.opacity = '0';
    document.body.appendChild(el);
    el.select();
    document.execCommand('copy');
    document.body.removeChild(el);
  });
}
```

## The popup

I wanted the popup to actually explain what the extension does rather than just being a blank icon. It shows a before/after example of a full URL versus the dynamic path, and has a Buy Me a Coffee link at the bottom.

The whole thing is built to be accessible — proper colour contrast, keyboard navigable, and it works with screen readers too.

## Get it

You can find it on the Chrome Web Store, or check out the source on GitHub.
