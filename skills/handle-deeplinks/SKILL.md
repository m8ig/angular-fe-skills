---
name: handle deeplinks (web + mobile)
description: "Use when a project has native iOS/Android apps and needs URL-based deep linking. Covers two scenarios: (A) web app + mobile — Universal Links/App Links with web fallback and smart banner; (B) landing page only + mobile — AASA file intercepts links from emails/SMS, fallback page with store download buttons if app not installed."
---

# Handle deeplinks — web + mobile

Two scenarios are covered below. Choose based on what the project has:

- **Scenario A** — project has both a web app and native mobile apps
- **Scenario B** — project has only a landing page + native mobile apps (no web app)

---

## Scenario A — Web app + mobile apps

The goal: a single URL works everywhere — opens the native app if installed, falls back to the web app if not.

```
User taps a URL
  ├── iOS/Android app installed?  → Open native app on the correct screen
  └── Not installed?              → Stay on the web version
```

Web app also shows a **smart banner** on mobile devices — a native-style prompt to download the app.

---

## iOS — Universal Links

**File:** `apps/web/src/.well-known/apple-app-site-association`

```json
{
    "applinks": {
        "apps": [],
        "details": [
            {
                "appID": "TEAM_ID.com.yourapp.bundle",
                "paths": [
                    "NOT /news/*",
                    "*",
                    "/"
                ]
            }
        ]
    }
}
```

Rules:
- `appID` = `{Apple Team ID}.{Bundle ID}`
- `paths` lists which URL paths open the native app — use `NOT /path/*` to exclude paths that should always stay on the web (e.g. news, blog, auth flows)
- File must be served at `https://yourdomain.com/.well-known/apple-app-site-association` with `Content-Type: application/json`, no redirect, no auth
- If the project has multiple build environments (prod, stage, test), add each as a separate `appID` entry — each environment has its own Bundle ID and Team ID combination

**Multiple environments example** (only relevant if project has separate app builds per environment):
```json
{
    "applinks": {
        "apps": [],
        "details": [
            { "appID": "2DJ69J9Z22.com.casashares.casashares", "paths": ["NOT /news/*", "*", "/"] },
            { "appID": "47YU4DF97R.io.scal.casa-shares.stage", "paths": ["NOT /news/*", "*", "/"] },
            { "appID": "47YU4DF97R.io.scal.casa-shares.test",  "paths": ["NOT /news/*", "*", "/"] }
        ]
    }
}
```

**In the iOS app** — enable Associated Domains capability:
```
Associated Domains: applinks:yourdomain.com
```

---

## Android — App Links

**File:** `apps/web/public/.well-known/assetlinks.json`

```json
[
    {
        "relation": ["delegate_permission/common.handle_all_urls"],
        "target": {
            "namespace": "android_app",
            "package_name": "com.yourapp",
            "sha256_cert_fingerprints": [
                "AA:BB:CC:..."
            ]
        }
    }
]
```

Rules:
- Served at `https://yourdomain.com/.well-known/assetlinks.json`
- Add separate entries for debug and release signing keys (if applicable)
- In `AndroidManifest.xml` add `intent-filter` with `autoVerify="true"` for the domain

---

## Serving `.well-known` files correctly

The files must be accessible without redirects and with the correct content type.

**Nginx:**
```nginx
location /.well-known/ {
    default_type application/json;
    add_header Content-Type application/json;
}
```

**Angular / Next.js:** Place files in `public/.well-known/` — they are served as static assets automatically.

**NX monorepo:** Files in `apps/web/src/.well-known/` need to be included in the build output. In `project.json`:
```json
{
    "assets": [
        { "glob": "**/*", "input": "apps/web/src/.well-known", "output": ".well-known" }
    ]
}
```

---

## Smart banner — prompt to install the app

Shows a native-style banner on mobile browsers suggesting to open or download the app.

**iOS Safari smart banner** — add to `<head>`:
```html
<meta name="apple-itunes-app" content="app-id=YOUR_APP_STORE_ID">
```
Safari renders a native install/open banner automatically. No JS needed.

**Android / cross-platform** — use a custom banner or a library.

Simple custom implementation:
```ts
// Show banner only on mobile, only if not already dismissed
function showAppBanner(): void {
    const isMobile = /Android|iPhone|iPad/i.test(navigator.userAgent);
    const dismissed = localStorage.getItem('app-banner-dismissed');
    if (!isMobile || dismissed) return;

    // Render your banner UI
    // On "Open app" → redirect to app store URL or universal link
    // On dismiss → localStorage.setItem('app-banner-dismissed', '1')
}
```

App Store / Play Store links:
```ts
const APP_STORE_URL = 'https://apps.apple.com/app/idYOUR_ID';
const PLAY_STORE_URL = 'https://play.google.com/store/apps/details?id=com.yourapp';
```

---

## Path exclusions — what stays on web

Some paths should never deep-link to the native app:
- `/news/*`, `/blog/*` — content pages not in the app
- `/auth/*` — OAuth callbacks, email confirmation links
- `/admin/*` — CMS / back-office

Add `NOT /path/*` before `*` in both `apple-app-site-association` and handle the same in Android intent filters.

---

---

## Scenario B — Landing page only + mobile apps (no web app)

Used when the project has no web application but still needs to send URLs via email, push notifications, or SMS (e.g. account activation, password reset, referral links). The landing serves as a relay: it hosts the Universal Link / App Link files, and the OS intercepts the URL before the page even loads — sending the user directly to the native app.

```
User taps a link (email, SMS, etc.)
  ├── App installed (iOS)?   → OS intercepts via Universal Links → opens app on correct screen
  ├── App installed (Android)? → OS intercepts via App Links → opens app on correct screen
  └── App not installed?     → Landing loads a purpose-built page (e.g. "Activate account")
                                 with buttons: "Download on App Store" / "Get it on Google Play"
```

---

### iOS setup (same file, same rules)

The landing must serve `apple-app-site-association` at `https://landing.yourapp.com/.well-known/apple-app-site-association`.

Since there is no web app, all paths can be claimed — no need for `NOT` exclusions:

```json
{
    "applinks": {
        "apps": [],
        "details": [
            {
                "appID": "TEAM_ID.com.yourapp.bundle",
                "paths": ["*", "/"]
            }
        ]
    }
}
```

When the app is installed, iOS intercepts the URL before Safari opens and routes the user to the correct screen inside the app. The landing page never loads.

**In the iOS app** — handle the incoming URL in `AppDelegate` / `SceneDelegate`:
```swift
func scene(_ scene: UIScene, continue userActivity: NSUserActivity) {
    guard let url = userActivity.webpageURL else { return }
    // Parse url.path and navigate to the correct screen
}
```

---

### Android setup

Same `assetlinks.json` on the landing domain. In `AndroidManifest.xml`:

```xml
<activity android:name=".MainActivity">
    <intent-filter android:autoVerify="true">
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data android:scheme="https" android:host="landing.yourapp.com" />
    </intent-filter>
</activity>
```

When the app is installed, Android intercepts the URL and opens the correct activity without loading the landing page.

---

### Fallback page (app not installed)

When the app is not installed, the OS does not intercept — the landing page opens normally. Build a minimal purpose-specific page for each deep-link type:

```
/activate?token=xxx  → "Activate your account" page
/reset?token=xxx     → "Reset password" page
/invite?code=xxx     → "You were invited" page
```

Each fallback page should:
1. Explain what the user needs to do
2. Show **"Download on App Store"** and **"Get it on Google Play"** buttons
3. Optionally try a custom URL scheme first as a last resort: `yourapp://activate?token=xxx` — if the app is installed but Universal Links failed (e.g. user copied the link manually), this may still open it

```ts
// Last-resort custom scheme attempt before showing store buttons
function tryOpenApp(deepPath: string): void {
    const customUrl = `yourapp://${deepPath}`;
    window.location.href = customUrl;
    // If app not installed, nothing happens — store buttons remain visible
}
```

---

## What NOT to do

- Don't serve `apple-app-site-association` with a redirect — iOS will not follow it
- Don't add the `.json` extension — the filename must be exactly `apple-app-site-association`
- Don't list multiple `appID` environments unless the project actually has separate builds per environment
- Don't use custom URL schemes (`myapp://`) as the primary mechanism — Universal Links / App Links are more reliable and don't show a confirmation dialog; use custom schemes only as a fallback
- Don't show the smart banner on desktop — check `navigator.userAgent` first
- Don't point email/SMS links directly to App Store — the user loses their token/context; always go through a landing URL that the OS can intercept
