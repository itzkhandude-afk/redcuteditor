# REDCUT

A browser video editor plus the full site around it. Static files, no build step, no dependencies.

## Run it

Double click `index.html`, or serve the folder:

```
python3 -m http.server 8080
```

Then open http://localhost:8080. Chrome or Edge give the widest format and export support.

## Files

| File | What it is |
|---|---|
| `index.html` | Landing page: hero, features, how it works, comparison, FAQ |
| `auth.html` | Login, signup, password reset, demo account |
| `dashboard.html` | Project library: folders, tags, pins, trash, version history, storage, export log |
| `editor.html` | The editor: 4 tracks, trim, split, ripple, fades, speed, titles, generated SFX, 4K export |
| `templates.html` | 12 presets that open a project sized for the platform |
| `pricing.html` | Four tiers, monthly/yearly toggle, comparison table, billing FAQ |
| `help.html` | Docs, shortcuts, troubleshooting, roadmap, changelog, contact form |
| `settings.html` | Account, workspace, editor, export defaults, keyboard, storage, notifications, plan |
| `admin.html` | Ops view over local data with feature flags |
| `legal.html` | Privacy, terms, cookies, GDPR, security notes (template text) |
| `assets/app.css` | Design system |
| `assets/store.js` | Auth, projects, media and export storage, nav and footer, toasts, modals |

## What actually works

Everything that can run without a server does run:

- Import video, audio and images by drag and drop or file picker
- Four track timeline with drag, trim, split, ripple delete, duplicate, snapping, markers
- Per clip volume, speed, fade in and fade out; per track mute and solo
- Title cards and colour cards rendered live on the canvas
- Sound effects generated with the Web Audio API and written to WAV
- Playback with J K L shuttle and around 60 Premiere and CapCut shortcuts
- Export to 1080p, 4K, vertical and square with selectable frame rate and bitrate
- Accounts, projects, folders, tags, pins, trash, version history
- Media cached in IndexedDB so projects reopen with clips attached
- Export history, storage usage, workspace preferences
- Full offline operation

## What is deliberately stubbed

These need servers. The interface is built, the call is not:

| Feature | What to attach |
|---|---|
| OAuth sign in | Google and GitHub OAuth, then replace `Auth.login` / `Auth.signup` in `assets/store.js` |
| Password reset | Mail service and a token endpoint |
| Checkout and invoices | Stripe on the `[data-buy]` buttons in `pricing.html` |
| AI panel | Model hosting behind an API; the panel lists the intended jobs |
| Stock library | Pixabay or Pexels API keys behind a proxy so keys stay server side |
| Cloud sync and teams | A projects API plus a realtime layer for shared editing |
| Support tickets | A `/api/support` endpoint; the form currently prints its payload |
| Admin panel | Move it server side behind role checks. Never ship a client side admin route |

## Wiring a backend

`assets/store.js` is the only file that touches storage. Replace the bodies of `Auth`, `Projects`, `Media` and `Exports` with HTTP calls and the rest of the app keeps working, since every page talks through `RC`.

Suggested API shape:

```
POST /api/auth/signup            POST /api/auth/login
GET  /api/projects               POST /api/projects
GET  /api/projects/:id           PUT  /api/projects/:id
POST /api/media                  GET  /api/media/:id
GET  /api/exports                POST /api/exports
POST /api/support
```

## Known limits

- Export records the timeline in real time, so a three minute cut takes three minutes. WebCodecs would render faster than real time and is the top roadmap item.
- Browsers decode only what they ship codecs for. HEVC, ProRes and RAW will not import; convert to H.264 MP4 first.
- `.redcut` files hold edit decisions, not media, exactly like a Premiere project file.
- Passwords are hashed in the browser, which is fine for a local prototype and not fine for production. See the security section in `legal.html`.
- The editor asks for a desktop below 860px wide.

## Before going live

1. Move authentication server side with Argon2 or bcrypt and short lived tokens.
2. Add rate limiting, CSRF protection and a strict content security policy.
3. Validate and scan every upload if you add server side media.
4. Replace the legal templates with reviewed text.
5. Add real analytics if you want them, and say so in the cookie notice.
