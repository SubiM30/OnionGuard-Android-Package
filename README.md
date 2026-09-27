# OnionGuard — Installable App Package

This package turns the single-file `OnionGuard_Web_App_v7.html` into:

1. **A PWA (Progressive Web App)** — installable from the browser, free to host and share.
2. **An Android APK / AAB** — a real installable app you can publish on Google Play.

Both routes use the same `www/` folder. Users enter their **own Gemini API key** in Settings; the key is stored only on their device (localStorage) and sent only to Google's Generative Language API.

---

## What's inside

```
onionguard/
├── www/                        # the app (PWA-ready)
│   ├── index.html              # your app, with manifest + service worker added
│   ├── manifest.webmanifest    # install metadata (name, icons, colors)
│   ├── sw.js                   # offline caching (never touches Gemini API calls)
│   └── icons/                  # generated app icons (any + maskable + iOS)
├── capacitor.config.json       # Android wrapper config
├── package.json                # Capacitor dependencies
└── .github/workflows/
    └── build-android.yml       # GitHub Actions: builds APK + AAB automatically
```

---

## Route 1 — PWA (100% free, fastest)

Host the `www/` folder on any static host. GitHub Pages is free:

1. Create a GitHub repository (e.g. `onionguard`), public.
2. Copy the contents of `www/` to the repo root (index.html, manifest, sw.js, icons/).
3. GitHub repo → Settings → Pages → Source: "Deploy from a branch" → `main` / root.
4. Your app is live at `https://<username>.github.io/onionguard/`.

Anyone who opens that URL in Chrome on Android can tap **"Add to Home screen / Install app"** — it then launches fullscreen with its own icon and works offline (after first load). No store, no fees.

Notes:
- The service worker uses relative paths, so it works under a subpath (like GitHub Pages) or a custom domain.
- Hosting must be HTTPS (GitHub Pages, Netlify, Vercel, Cloudflare Pages all are).

## Route 2 — Android APK / Play Store (Capacitor)

No Android Studio needed — GitHub Actions builds it for you:

1. Push this whole folder (including `.github/`) to a GitHub repository, `main` branch.
2. GitHub repo → Actions tab → workflow "Build OnionGuard Android" runs automatically.
3. When it finishes (about 5–10 minutes), download the artifacts:
   - **OnionGuard-debug-APK** → `app-debug.apk`: installable directly on any Android phone ("Install unknown apps" must be allowed). Fine for testing and sharing directly.
   - **OnionGuard-release-AAB** → `app-release.aab`: the bundle format for Play Store upload.

If you do have Android Studio locally: `npm install`, `npx cap add android`, `npx cap sync android`, open the `android/` folder in Android Studio and build.

### Signing for Play Store

The AAB from CI is unsigned. For a store upload you must sign it with your own keystore:

1. Generate a keystore once:
   `keytool -genkey -v -keystore onionguard.keystore -alias onionguard -keyalg RSA -keysize 2048 -validity 10000`
2. In Play Console → your app → Release → Production → "App signing by Google Play", let Google manage the key and upload the AAB, or configure signing in `android/app/build.gradle` with your keystore (add the secrets to GitHub repo → Settings → Secrets and use them in the workflow).

---

## Publishing on Google Play — costs and requirements

- **Developer account: one-time USD 25 fee** (full distribution). There is also a no-fee "limited distribution" account, but it caps you at ~20 devices — not suitable for a public app.
- **Personal accounts must run a closed test first**: newer personal accounts need a closed test with at least 20 testers who opt in for 14 days before production access. Organization accounts skip this but need a D-U-N-S number.
- **Privacy policy URL is required.** OnionGuard is easy to write honestly: it collects nothing, no analytics, no ads; the API key and history are stored only in the browser/app on the device; images are sent directly from the user's device to Google's Gemini API using the user's own key.
- **Data safety form**: declare "no data collected" (data goes directly from the user's device to Google under the user's own key — you never see it).
- Content rating questionnaire: educational/utility, everyone.

## End-user cost

- The app itself is free.
- Gemini API has a **free tier** with daily rate limits per model; users can create a free API key at aistudio.google.com/apikey. Heavy users can enable billing on their own Google account, but for typical onion-grading usage the free tier is enough. Your users bear their own usage; you pay nothing and charge nothing.

## Before you publish — checklist

- [ ] Test the PWA install flow on a real Android phone.
- [ ] Test the debug APK: install, paste a real API key in Settings, run an assessment with camera photos.
- [ ] Test offline: kill network after first load — the app shell should still open.
- [ ] Replace the placeholder `appId` if you own a domain (reverse-DNS, e.g. `com.yourdomain.onionguard`) — it cannot be changed after publishing.
- [ ] Bump `versionCode` / `versionName` in `android/app/build.gradle` for each Play Store update.
