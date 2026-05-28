# Deploy to Netlify (drag-and-drop)

Zero config, zero account-linking, ~30 seconds.

## Steps

1. Sign in (or sign up free) at https://app.netlify.com
2. On the dashboard, find the drag-and-drop area labeled **"Want to deploy a new site without connecting to Git? Drag and drop your site output folder here"**
3. Unzip this archive on your computer. You'll get a folder containing:
   - `index.html`
   - `send.html`
   - `receive.html`
   - `_redirects`
   - `_headers`
   - `favicon.svg`
4. **Drag the entire folder** (not the zip, not the individual files) onto the drop zone
5. Wait ~10 seconds. Netlify assigns a random URL like `https://gleaming-otter-1a2b3c.netlify.app`
6. Click the URL to open your site

## Custom URL (optional)

After deploy:
- **Site settings** → **Domain management** → **Options** → **Edit site name** to pick a friendlier subdomain like `your-name-airgap.netlify.app`
- Or add a custom domain you own under the same panel

## Updating later

To push an update: zip the folder again, drag onto the same site's **Deploys** tab. Netlify replaces the live version instantly.

## Verification — what should work after deploy

| URL | Should show |
|---|---|
| `https://your-site.netlify.app/` | Landing page with two cards |
| `https://your-site.netlify.app/send` | Sender (no .html in URL) |
| `https://your-site.netlify.app/receive` | Receiver (prompts for camera) |
| `https://your-site.netlify.app/send.html` | Redirects to `/send` |
| `https://your-site.netlify.app/receive.html` | Redirects to `/receive` |

## Camera permission

Netlify auto-issues HTTPS for every deploy, so `getUserMedia` works immediately. No setup needed.

If a user denies camera permission, they need to re-enable it in their browser's site settings — there's no in-app prompt to re-request.

## Files Netlify ignores

`_redirects` and `_headers` are config — they configure the CDN, they aren't served as pages.

## File structure on Netlify after deploy

```
/                  → index.html
/send              → send.html (via _redirects)
/receive           → receive.html (via _redirects)
/favicon.svg       → favicon.svg
```

## Cost

Free tier on Netlify covers this comfortably:
- 100 GB bandwidth/month
- Unlimited sites
- Free HTTPS

The site itself loads ~200 KB of JS from CDNs (pako, qrcodejs, zbar-wasm) — your Netlify bandwidth is just the three HTML files.
