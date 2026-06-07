# TrustDrop — Developer Handoff Brief

A comprehensive technical reference for developers taking over or contributing to TrustDrop.

---

## What Is TrustDrop?

TrustDrop is a **zero-knowledge, one-time secret sharing web application**. Users paste a secret message or upload a file (up to 50 MB). The browser encrypts the content client-side using AES-256-GCM before any network request. The server stores only ciphertext. A one-time link is generated. When the recipient opens the link, the content decrypts in their browser and the server destroys the ciphertext permanently.

**No accounts. No plaintext ever reaches the server. No trust required.**

Live at: [https://trustdrop.com](https://trustdrop.com)
Repository: [https://github.com/trust-drop/TrustDrop](https://github.com/trust-drop/TrustDrop)
License: AGPL-3.0

---

## Tech Stack

### Frontend
| Technology | Version | Purpose |
|---|---|---|
| React | 18 | UI framework |
| Vite | (bundler) | Dev server + production builds |
| Tailwind CSS | 3 | Utility-first styling |
| Shadcn UI + Radix | * | Accessible component primitives |
| wouter | 3 | Lightweight client-side routing |
| TanStack Query | v5 | Server state, data fetching, cache |
| react-hook-form | 7 | Form state management |
| Zod | 3 | Shared schema validation |
| lucide-react | * | Icons |
| react-icons/si | * | Brand/social icons (GitHub, etc.) |
| marked | 17 | Markdown rendering for blog posts |
| dompurify | 3 | Sanitize rendered markdown HTML |
| framer-motion | 11 | Animations |
| WebCrypto API | (browser) | AES-256-GCM encryption/decryption |

### Backend
| Technology | Version | Purpose |
|---|---|---|
| Express.js | 5 | HTTP server and API routing |
| express-session | 1 | Admin session management |
| express-rate-limit | 8 | Tiered per-IP rate limiting |
| bcryptjs | 3 | Admin password hashing (12 rounds) |
| otplib | 13 | TOTP generation and verification |
| qrcode | 1 | Generate TOTP QR code data URL |
| pg | 8 | PostgreSQL client |
| drizzle-orm | 0.39 | Type-safe ORM + query builder |
| drizzle-zod | 0.7 | Generate Zod schemas from Drizzle tables |
| zod-validation-error | 3 | Human-readable Zod error messages |
| Google Cloud Storage | 7 | Object storage for OG image uploads |

### Database
- **PostgreSQL** (Neon-backed via Replit)
- Connected via `DATABASE_URL` environment variable
- Migrations managed with Drizzle Kit (`npx drizzle-kit push`)

---

## Repository Structure

```
trustdrop/
├── client/                   # React frontend (Vite)
│   ├── public/
│   │   ├── drop-logo.png     # Square drop icon
│   │   ├── trust-logo-rect.png  # Rectangular navbar logo
│   │   ├── favicon.svg
│   │   ├── og-image.png      # Default Open Graph image
│   │   ├── widget.js         # Embeddable widget (Shadow DOM, self-contained)
│   │   └── widget-demo.html  # Widget test page
│   └── src/
│       ├── App.tsx           # Router — all page routes registered here
│       ├── main.tsx          # React entry point
│       ├── index.css         # Tailwind base styles + CSS custom properties
│       ├── components/
│       │   ├── layout.tsx    # NavBar, Footer, DropIcon, RectLogo
│       │   ├── page-meta.tsx # Per-page SEO: title, description, OG tags
│       │   └── ui/           # Shadcn UI components (auto-generated, don't hand-edit)
│       ├── lib/
│       │   ├── crypto.ts     # WebCrypto: encrypt/decrypt message + file
│       │   ├── queryClient.ts # TanStack Query setup + apiRequest helper
│       │   └── utils.ts      # cn() className helper
│       └── pages/
│           ├── home.tsx          # Create drop (text + file tabs, burn settings)
│           ├── retrieve.tsx      # Retrieve/decrypt/burned/destroyed states
│           ├── admin.tsx         # Admin login + blog CRUD dashboard
│           ├── blog.tsx          # Public blog listing
│           ├── blog-post.tsx     # Public blog post (markdown rendered)
│           ├── how-it-works.tsx  # How It Works explainer
│           ├── trust-center.tsx  # Trust Center / security principles
│           ├── compare.tsx       # TrustDrop vs competitors comparison
│           ├── professionals.tsx # Use cases for security professionals
│           ├── about.tsx         # About page
│           ├── privacy.tsx       # Privacy Policy
│           ├── terms.tsx         # Terms & Conditions
│           ├── widget-demo.tsx   # Widget demo page
│           └── not-found.tsx     # 404 page
├── server/
│   ├── index.ts              # Express app setup, session, trust proxy, vite middleware
│   ├── routes.ts             # All API endpoints + rate limiters + security headers
│   ├── storage.ts            # IStorage interface + DatabaseStorage implementation
│   ├── vite.ts               # Vite dev middleware (do not modify)
│   └── replit_integrations/
│       └── object_storage.ts # OG image upload routes (Google Cloud Storage)
├── shared/
│   └── schema.ts             # Drizzle tables, Zod schemas, shared types
├── docs/
│   ├── openapi.yaml          # OpenAPI 3.0.3 API specification
│   ├── how-it-works.svg      # Visual diagram of the encrypt/share/destroy flow
│   └── developer-brief.md    # This file
├── drizzle.config.ts         # Drizzle Kit config (points to DATABASE_URL)
├── vite.config.ts            # Vite + path aliases (@/, @shared/, @assets/)
├── tailwind.config.ts        # Tailwind config + dark mode class strategy
├── tsconfig.json             # TypeScript config
├── package.json              # Dependencies + npm scripts
└── LICENSE                   # AGPL-3.0
```

---

## Environment Variables

| Variable | Required | Purpose |
|---|---|---|
| `DATABASE_URL` | Yes | PostgreSQL connection string |
| `SESSION_SECRET` | Yes (prod) | express-session secret — throws in production if missing |
| `ADMIN_SETUP_TOKEN` | Yes (first run) | One-time token to claim admin on a fresh deployment |
| `DEFAULT_OBJECT_STORAGE_BUCKET_ID` | Yes (uploads) | Google Cloud Storage bucket ID |
| `PUBLIC_OBJECT_SEARCH_PATHS` | Yes (uploads) | GCS search paths for serving uploaded objects |
| `PRIVATE_OBJECT_DIR` | Yes (uploads) | GCS directory for private uploaded files |

On Replit these are set as Secrets. For local dev, use a `.env` file (excluded from git).

---

## Running Locally

```bash
# Install dependencies
npm install

# Push schema to database (first time / after schema changes)
npx drizzle-kit push

# Start dev server (Express + Vite on the same port)
npm run dev
```

The dev server starts Express on a port and proxies API requests. Vite serves the frontend with HMR. Both run as a single process — no need to run frontend and backend separately.

---

## Database Schema

Defined in `shared/schema.ts` using Drizzle ORM.

### `drops` table
| Column | Type | Description |
|---|---|---|
| `id` | varchar(36) PK | 128-bit base64url ID (22 chars) |
| `access_token` | varchar(44) | Separate 128-bit token required for all operations |
| `ciphertext_b64` | text | AES-256-GCM encrypted payload (base64) |
| `meta` | jsonb | IV, salt, content type, filename, MIME — no plaintext |
| `expires_at` | timestamp | Time-based burn rule |
| `max_views` | integer | View-based burn rule (1-100) |
| `view_count` | integer | Successful decryptions so far |
| `burn_on_fail` | integer | 1 = destroy on wrong password attempt |

### `admin_config` table
Stores the bcrypt password hash and TOTP secret for the single admin account.

### `blog_posts` table
Full blog post: slug, title, excerpt, body (markdown), ogImageUrl, published flag, tags array, timestamps.

### `stats` table
Key-value store. Currently tracks `total_drops` (cumulative, survives post-deletion).

---

## Encryption Model

All encryption runs in the browser using the native **WebCrypto API** — no external crypto libraries.

### Mode A — No Password
1. Generate a random 256-bit key (`crypto.getRandomValues`)
2. Encrypt payload with AES-256-GCM using a random 12-byte IV
3. Key is base64url-encoded and placed in the URL fragment: `#k={key}`
4. Fragment is **never sent to the server** (browsers strip it from HTTP requests)
5. Server stores only: ciphertext + IV

### Mode B — Password Protected
1. User provides a password
2. Generate a random 16-byte PBKDF2 salt
3. Derive a 256-bit key via PBKDF2-SHA256 with **310,000 iterations**
4. Encrypt with AES-256-GCM
5. Server stores: ciphertext + IV + salt
6. Recipient needs the password — no key in URL fragment

**The server has zero ability to decrypt anything.** Even if the database is fully compromised, ciphertexts are useless without the key material that only the client holds.

---

## Two-Step Consume Flow

Drop retrieval is a deliberate two-step process to prevent wasting views on failed attempts:

```
1. POST /api/drops/:id/consume?t={token}
   → Server returns ciphertext (NO view counted)

2. Client attempts decryption locally

3a. Success → POST /api/drops/:id/confirm?t={token}
    → Server increments view count
    → If max_views reached: drop is permanently destroyed

3b. Failure + burn_on_fail=true → POST /api/drops/:id/burn?t={token}
    → Drop permanently destroyed immediately
```

---

## Security Design Decisions

### Dual-Token System
Every drop has two independent 128-bit random values:
- **Drop ID** — identifies the drop in the URL path
- **Access token** — required for all operations, passed as `?t=` query param

Both are cryptographically random. Knowing the ID alone is not enough to access the drop.

### Uniform 404 Responses
Non-existent drops, expired drops, consumed drops, and invalid tokens all return an identical `{"error":"consumed_or_expired"}` 404. This prevents **existence leakage** — an attacker cannot tell if a drop ever existed.

### Referrer-Policy: no-referrer
Access tokens travel in the query string. Without `Referrer-Policy: no-referrer`, navigating away from a retrieve page could leak the token to external sites via the `Referer` HTTP header.

### Burn on Fail
If `burn_on_fail` is enabled (default), a single wrong password attempt destroys the drop. This makes brute-forcing impossible — the secret self-destructs on the first mistake.

### Rate Limiting
All limits are per-IP using `X-Forwarded-For` (Replit reverse proxy). In-memory store (single-server). Returns 429 with `Retry-After` header.

| Endpoint | Limit |
|---|---|
| `POST /api/drops` | 50 req / 15 min |
| Drop retrieval (meta/consume/confirm/burn) | 150 req / 15 min |
| `POST /api/admin/login` | 25 req / 15 min |
| All other `/api/*` | 500 req / 15 min |

---

## Admin System

The admin panel lives at `/admin` (not linked in public nav).

### First-Time Setup
1. Set `ADMIN_SETUP_TOKEN` environment variable
2. `POST /api/admin/setup` with `{ password, setupToken }`
3. Returns TOTP QR code — scan with Google Authenticator or Authy
4. Save the backup TOTP key somewhere secure

### Login Flow
1. `POST /api/admin/login` with `{ password, totpCode }`
2. Returns session cookie (24-hour expiry, httpOnly, sameSite strict)
3. All `/api/admin/*` protected routes check the session

### Admin Capabilities
- Blog post CRUD (create, edit, publish/draft toggle, delete)
- OG image upload for blog posts (stored in GCS, served via `/objects/uploads/:id`)

---

## Blog System

- Posts stored as Markdown in PostgreSQL
- Admin dashboard for full CRUD
- Public listing at `/blog`, individual posts at `/blog/:slug`
- Server-side OG tag injection for crawlers (Slack, Twitter, Discord, etc.) at `GET /blog/:slug`
- RSS feed at `/blog/feed.xml`
- Images uploaded via GCS integration, served from `/objects/uploads/:id`

---

## Embeddable Widget

`client/public/widget.js` — a fully self-contained embeddable widget using **Shadow DOM** so it doesn't conflict with host page styles.

Features:
- Runs on any website with a single `<script>` tag
- Implements AES-256-GCM encryption inline (no CDN dependencies)
- Calls `POST /api/drops` with CORS (the only CORS-enabled endpoint)
- Returns a TrustDrop link to the host page

Demo: `/widget-demo` or `client/public/widget-demo.html`

---

## Frontend Routing

All routes registered in `client/src/App.tsx` using `wouter`:

| Route | Page | Notes |
|---|---|---|
| `/` | Home | Create drop |
| `/d/:id` | Retrieve | Decrypt + burn flow |
| `/how-it-works` | How It Works | |
| `/trust` | Trust Center | Security principles |
| `/compare` | Compare | TrustDrop vs alternatives |
| `/professionals` | Professionals | Security pro use cases |
| `/blog` | Blog | Public post listing |
| `/blog/:slug` | BlogPost | Individual post (markdown) |
| `/about` | About | |
| `/privacy` | Privacy Policy | |
| `/terms` | Terms & Conditions | |
| `/admin` | Admin | Not linked in public nav |
| `/widget-demo` | Widget Demo | |

---

## Path Aliases

Configured in `vite.config.ts` and `tsconfig.json`:

| Alias | Resolves to |
|---|---|
| `@/` | `client/src/` |
| `@shared/` | `shared/` |
| `@assets/` | `attached_assets/` |

---

## SEO

Every page uses `<PageMeta>` (`client/src/components/page-meta.tsx`) to set:
- `<title>` — unique per page
- `<meta name="description">` — keyword-optimized
- `<meta property="og:*">` — Open Graph for social sharing
- `<meta name="twitter:*">` — Twitter card

Blog post pages get **server-side OG injection** for crawlers (which don't execute JS). Crawlers hitting `/blog/:slug` receive a minimal HTML page with post-specific OG tags; real browsers receive the SPA.

---

## Cleanup Job

A background interval in `server/index.ts` runs every **60 seconds** calling `storage.cleanupExpired()`. This hard-deletes all drops where `expires_at < NOW()` from the database.

---

## What's Intentionally Not Built Yet

From the roadmap in `replit.md`:

1. **Multi-Tenant White-Label** — Self-service onboarding funnel where prospects configure branding, activate via Stripe, and get their own branded TrustDrop instance. Planned as a 6-step implementation: Tenant Configurator → Tenant Foundation → Stripe Payment Activation → Dynamic Theming → Superadmin Dashboard → Billing Management.

2. **Browser Extension** — Chrome + Firefox extension. Core `crypto.ts` reuses as-is via WebCrypto; extension is just another client. Build in an `extension/` folder, download as zip for local testing.

3. **Self-Hosting / Docker** — Docker + docker-compose setup. Migration to Replit App Storage for larger uploads (500 MB–1 GB).

---

## Key Conventions

- **Dark mode always on** — `class="dark"` set on `<html>` in `client/index.html`. Never toggle it.
- **Color scheme** — Blue-teal primary `hsl(195 55% 42%)` to match logo branding
- **Fonts** — Inter (loaded via CSS)
- **Icons** — `lucide-react` for UI icons, `react-icons/si` for brand logos. No emojis.
- **Logos** — `DropIcon` (square) for content pages, `RectLogo` (rectangular, `mix-blend-lighten`) for navbar. Both served as static files from `client/public/`, not imported.
- **Comments** — Always update code comments when modifying code to reflect the current state.
- **Shadcn components** — Live in `client/src/components/ui/`. These are auto-generated by the Shadcn CLI — do not hand-edit them. Re-run the CLI to update.
- **Storage layer** — All database access goes through `IStorage` / `DatabaseStorage` in `server/storage.ts`. Routes should be thin wrappers that call storage methods.
- **Transactions + SELECT FOR UPDATE** — All security-sensitive operations (consume, confirm, burn) use PostgreSQL transactions with `SELECT FOR UPDATE` to prevent race conditions.

---

## API Reference

Full OpenAPI 3.0.3 specification: `docs/openapi.yaml`

Paste into [https://editor.swagger.io](https://editor.swagger.io) for an interactive UI.

---

## Custom Domain

Live at `trustdrop.com`. DNS managed separately from Replit. The app is deployed on Replit with `trust proxy` enabled for proper HTTPS cookie behavior behind the reverse proxy.
