# Wheel of Verses website

A small static site for GitHub Pages with your own domain. It gives the stores and AdMob
everything they ask for, and hosts the Daily Verse files the app downloads.

| Path | What it's for | Where you paste the URL |
|---|---|---|
| `/` | Landing page with store links | Play Console → Store settings → **Website** |
| `/privacy/` | Privacy policy (draft) | Play Console → App content → **Privacy policy**; App Store Connect → **Privacy Policy URL**; `LINKS.privacyPolicy` in the app |
| `/terms/` | Terms of use (draft) | `LINKS.terms` in the app |
| `/delete-account/` | Account deletion steps, in and outside the app | Play Console → App content → Data safety → **Delete account URL** |
| `/support/` | Contact + FAQ | App Store Connect → **Support URL** |
| `/app-ads.txt` | Authorises AdMob to sell your ad space (publisher `pub-7265062094984165`) | Read automatically by AdMob from your **Website** |
| `/content/index.json`, `/content/daily.json` | Daily verses for the app | `CONTENT.CONTENT_BASE_URL` in the app |

Everything is plain HTML + one CSS file. No build step, no scripts, no trackers.

---

## 1. Buy the domain

**Recommended: Cloudflare Registrar** (dash.cloudflare.com → Domain Registration). It sells at
cost with no markup on renewals, and gives you free DNS and free email forwarding, which you need
for `support@yourdomain`. Porkbun and Namecheap are fine alternatives.

- Prefer a **.com** (most trusted by parents and churches). `.app` also works well; it forces
  HTTPS, which GitHub Pages provides.
- Check that the name isn't someone else's trademark, and that the matching Play listing name is
  free.
- Turn on auto-renew. If the domain lapses, your privacy-policy link breaks and the stores may pull
  the app.

## 2. Fill in your domain and name

```bash
cd website
chmod +x set-domain.sh
./set-domain.sh yourdomain.com "Your Name"
```

This replaces `YOURDOMAIN.com` and `YOUR NAME` in every page and in `CNAME`. Your name appears
as the developer in the policies; use the same name as your Play Console developer account.

## 3. Publish on GitHub Pages (free)

1. Create a **public** repository, e.g. `wheel-of-verses-site`, and push the contents of this
   folder to its `main` branch (files at the repository root, not inside `website/`).
2. Repository → **Settings → Pages** → Source: *Deploy from a branch*, Branch: `main`, folder
   `/ (root)` → Save.
3. Still on that page, **Custom domain**: enter `yourdomain.com` (no `www`) → Save. Leave it until
   DNS is set up in step 4.
4. Recommended: GitHub → your profile → **Settings → Pages → Add a domain** to verify it. This
   stops anyone else from pointing a GitHub site at your domain.

## 4. Point the domain at GitHub (DNS)

In Cloudflare → your domain → **DNS → Records**, add:

| Type | Name | Content |
|---|---|---|
| A | `@` | `185.199.108.153` |
| A | `@` | `185.199.109.153` |
| A | `@` | `185.199.110.153` |
| A | `@` | `185.199.111.153` |
| AAAA | `@` | `2606:50c0:8000::153` |
| AAAA | `@` | `2606:50c0:8001::153` |
| AAAA | `@` | `2606:50c0:8002::153` |
| AAAA | `@` | `2606:50c0:8003::153` |
| CNAME | `www` | `YOUR-GITHUB-USERNAME.github.io` |

**Set every one of these to "DNS only" (grey cloud), not "Proxied".** Otherwise GitHub can't issue
the HTTPS certificate. Check the IP addresses against GitHub's docs ("Managing a custom domain for
your GitHub Pages site") in case they have changed.

Wait 10 minutes to a few hours, then in **Settings → Pages** tick **Enforce HTTPS**.
`www.yourdomain.com` will redirect to `yourdomain.com` automatically.

## 5. Support email (free)

Cloudflare → your domain → **Email → Email Routing** → enable → add a rule:
`support@yourdomain.com` → your Gmail. Cloudflare adds the MX records for you. Send a test email.

To *reply* from `support@` rather than your Gmail address, add it in Gmail → Settings → Accounts →
"Send mail as" (needs an SMTP server; replying from Gmail is fine to start with).

## 6. Point the app at the site

In `src/config.js` **and** your `src/config.js.prd`:

```js
export const CONTENT = {
  CONTENT_BASE_URL: 'https://yourdomain.com/content', // no trailing slash
  // ...
};

export const LINKS = {
  privacyPolicy: 'https://yourdomain.com/privacy/',
  terms: 'https://yourdomain.com/terms/',
};
```

Then `npm run validate`, `npm test`, `npm run sync:release`, and build the release bundle.

Test on the phone: Settings → Privacy Policy opens your page; the Daily Verse still loads in
airplane mode (bundled) and picks up a new day added to `content/daily.json` when online.

**CORS:** the app fetches the JSON from `https://localhost`, so the host must send
`Access-Control-Allow-Origin: *`. GitHub Pages does this on every file by default, so the
`_headers` file from `content-host/` isn't needed here. Check once with:
`curl -sI https://yourdomain.com/content/index.json | grep -i access-control`

GitHub Pages caches files for about 10 minutes, so new days appear in the app shortly after you push.

## 7. Store and AdMob settings

- **Play Console → Store settings → Store listing contact details:** Website `https://yourdomain.com`
  (the bare domain, because that's where AdMob looks for `app-ads.txt`), Email `support@yourdomain.com`.
- **Play Console → App content:** Privacy policy URL and Data safety → Delete account URL (table above).
- **AdMob → Apps → your app → App settings:** link it to the Play listing. AdMob then crawls
  `https://yourdomain.com/app-ads.txt`. Verification can take a day or more after the listing is
  live. Check the line in `app-ads.txt` matches what AdMob shows under **Apps → app-ads.txt**.
- **App Store Connect (later):** Support URL `/support/`, Privacy Policy URL `/privacy/`.

## 8. Updating

- **New daily verses:** edit `content/daily.json`, update `updatedAt` in both `content/index.json`
  and `content/daily.json`, then push. Keep the app's `content-host/` folder in sync (or delete it and
  treat this folder as the source). Run the app's `npm run validate` on the files first.
- **Policy changes:** edit the page, change the "Last updated" date, push.
- If the app starts collecting anything new, update `/privacy/` **and** the Play Data safety form at
  the same time. They must match.

## Before you publish: review these

- The privacy policy and terms are drafts written from the app's code (v0.8.0). They are not legal
  advice. Read every section, and search for `TODO` in the HTML.
- **Remove Ads:** the pages mention the Remove Ads purchase. This build has no purchase code yet.
  If you release without it, delete the paragraphs marked `TODO` in `/privacy/` and `/terms/`, and the
  Remove Ads answers in `/support/` and `/delete-account/`.
- **Children / target audience:** the app tags ads as child-directed. Section 8 of the privacy
  policy must match the age groups you choose in Play Console → Target audience. If you include
  under-13s, Google's Families policy applies and is stricter about sign-in and ads.
