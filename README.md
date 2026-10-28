# Notebook — animated paper workspace + encrypted online transfer vault

Notebook now has two distinct experiences:

- `index.html` — the public-facing landing/home page with animated hero, feature sections, responsive design, and clear entry points.
- `app.html` — the separate workspace dashboard for notes, pages, tasks, collections, favorites, search, templates, trash, settings, backup/restore, and transfer tools.

The app is plain HTML/CSS/JavaScript. There is no build step for the front end.

## Run locally

Open `index.html` in a modern browser, then choose **Open workspace**. Notes and tasks save in that browser's `localStorage`. For secure encryption features, use HTTPS or localhost; `file://` may not expose Web Crypto consistently.

## Local workspace features

- Create and edit pages with a rich-text editor
- Share a single page online using its own encrypted alphanumeric code; import it on another laptop without replacing existing pages
- Page icons, descriptions, favorites, duplicate, Markdown export, and trash/restore
- Notes, journal, and project categories
- Starter templates, tasks, collections, and search (`Ctrl/⌘ + K`)
- Light/dark theme and responsive mobile navigation
- JSON backup export and restore
- Manual encrypted transfer string for a fully offline, no-upload transfer
- Animated landing page and motion-aware transitions (`prefers-reduced-motion` supported)

## Online transfer codes (Cloudflare Pages + KV)

The online vault stores encrypted snapshots under an alphanumeric code such as `NF7K2P9X4M8QAB`. The browser encrypts the workspace with AES-GCM using a passkey-derived key before uploading it. The server only receives the encrypted string. Transfers have **no automatic expiry** and remain stored until someone deletes them or the Cloudflare KV namespace is removed. This is a snapshot transfer, not automatic continuous sync.

### Deploy setup (important — prevents “Failed to fetch”)

**Important:** The browser app and API must be served from the same deployed Cloudflare Pages origin. Do not open `app.html` using `file://`, and do not upload only the HTML files to a static host. The included `functions/api/*.js` files must be deployed as Cloudflare Pages Functions. The frontend calls `/api/transfers` and `/api/transfer-status` on the same origin.

**Do not just open `app.html` from your computer, and do not use a static-only host for online transfers.** The `/api/transfers` endpoint must run as a Cloudflare Pages Function. If you use a drag-and-drop static upload, the HTML may load but the API function will not run, which causes “Failed to fetch.”

1. Deploy the entire `studyspace-notion` folder as a **Cloudflare Pages project with Functions enabled**. Recommended: connect a Git repository containing this folder using Cloudflare Pages Git integration, with this folder as the project root and no build command needed for the plain HTML front end. Alternatively, use Wrangler CLI to deploy a Pages project directory including `functions/`. Keep `index.html`, `app.html`, and the `functions/` folder at the project root.
2. In Cloudflare, open **Workers & Pages → your Pages project → Settings → Bindings**.
3. Add a **KV namespace binding** with variable name exactly `NOTEFUL_TRANSFERS`. Create a KV namespace first if needed.
4. Save the binding and redeploy the Pages project so the Functions runtime receives it.
5. Open the deployed site over HTTPS. Go to `app.html` → **Settings** → **Online Transfer Vault** and click **Test online connection** first. It should say “Connected!” before you upload a page or workspace.
6. On the source laptop, enter a strong passkey (10+ characters, letters and numbers are fine) and click **Upload encrypted workspace**. Save the generated code and passkey separately. To share only one page, open that page, click **Share page**, and upload it with a passkey.
7. On the destination laptop, open the same deployed site and enter the code and passkey in Settings. Use **Restore workspace** to replace the local workspace, or **Import shared page** to add only that page while keeping existing pages.
8. When you no longer need the online copy, enter its code and click **Delete online code**. The API permanently deletes that KV entry.

### Security and limits

- Never share the passkey in the same place as the transfer code. Anyone who gets the code can delete that snapshot; the passkey is needed to decrypt it.
- The code is a bearer locator, not an account login. Keep it private. The app does not implement user accounts, per-user ownership, or a recovery service.
- The Pages Function rejects payloads over 2 MB. For larger workspaces, use **Export backup** or reduce media/large embedded content.
- The online snapshot is a point-in-time copy. After making more edits, create a new transfer code to move the updated snapshot. Individual shared pages also represent a point-in-time copy; upload again to share later edits.
- Local browser storage can be cleared by the browser/device. Use backups for important data.
- Cloudflare KV is a distributed key-value store; updates can take a short time to become visible globally. Do not use this prototype as the only copy of critical information.
- If you deploy on a host other than Cloudflare Pages, the `functions/api/transfers.js` API route will not run automatically; you need an equivalent backend endpoint.

## Project structure

```text
studyspace-notion/
├── index.html                 # Animated marketing/home page
├── app.html                   # Separate workspace dashboard
├── functions/
│   └── api/
│       └── transfers.js       # Cloudflare Pages API backed by KV
└── README.md
```

## If you see “Failed to fetch”

- Make sure you are using the deployed `https://…pages.dev` address, not a `file:///…/app.html` local file.
- Confirm the project was deployed with Cloudflare Pages Functions (the `functions/` folder is at the project root). Static-only hosting such as GitHub Pages cannot run this API.
- Confirm a KV namespace is bound as `NOTEFUL_TRANSFERS`, then redeploy.
- In Settings, click **Test online connection**. The status message distinguishes a missing API route from a missing KV binding.
- After a new deployment, use a newly generated code; older codes only exist if their earlier upload actually succeeded.


## New page editor

Open `new-page.html` to create a page using the warm paper, charcoal, and burnt-orange theme. Saving adds the page to the same `noteful_local_v1` browser storage used by `app.html` when both pages are served from the same origin (for example, the deployed HTTPS site or localhost). The new page then opens the dashboard so you can continue editing and share the page.
