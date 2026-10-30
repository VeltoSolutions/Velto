[README.md](https://github.com/user-attachments/files/34191538/README.md)
# Velto — velto.co.in

Premium multi-page website for **Velto**, a technology and business solutions company.
Built as a fast, dependency-free static site with a secure serverless contact form.

- 16 pages: Home, About, Services, 9 service detail pages, Solutions, Contact, Privacy, 404
- Original Velto logo system, favicon set and social share image
- Custom motion engine: loader, smooth scroll, scroll reveals, word-by-word headline reveals, parallax, particle-network hero, 3D orb with mouse tilt, animated counters, spotlight cards, magnetic buttons, page transitions
- Fully responsive (desktop → 320px phones), keyboard accessible, respects "reduced motion"
- Real contact form → emails every inquiry to **connectrm96@gmail.com** (SMTP or Resend), with server-side validation, honeypot, timing check, origin check and rate limiting
- SEO: unique titles/descriptions, canonical URLs, Open Graph/Twitter cards, JSON-LD (Organization, WebSite, Service, BreadcrumbList), sitemap.xml, robots.txt, web manifest

**No `npm install` is required.** The project uses only Node.js built-ins (Node 18 or newer).

---

## 1. Project structure

```
velto/
├── netlify/functions/contact.mjs   Netlify Function   →  POST /api/contact
├── netlify.toml            Netlify config (build, function, headers, www → apex redirect)
├── api/contact.js          Vercel serverless function (alternative host)
├── lib/contact.mjs         Validation, spam protection, rate limiting, email composition
├── lib/smtp.mjs            Minimal secure SMTP client (TLS / STARTTLS)
├── src/data.mjs            ★ Contact details, social links, services copy — edit here
├── src/brand.mjs           Logo geometry (symbol + wordmark)
├── src/layout.mjs          <head>, SEO tags, header, mega menu, mobile menu, footer
├── src/components.mjs      Reusable sections + code-built mockups (browser, phones, HRMS, Excel, support console)
├── src/icons.mjs           Inline SVG icon set
├── src/pages/*.mjs         One file per page
├── assets/css/styles.css   Design system
├── assets/js/main.js       Motion + interaction engine
├── assets/brand/           Logo files (SVG + PNG), favicon, OG image
├── build.mjs               Static site generator  →  dist/
├── server.mjs              Node server for local preview or VPS hosting
├── vercel.json             Vercel config (only used if you deploy to Vercel instead)
├── scripts/                One-time brand asset export (already run; outputs are committed)
└── .env.example            Environment variable template
```

## 2. Run it locally

```bash
cp .env.example .env        # then fill in your email credentials (section 3)
npm run dev                 # builds to dist/ and serves http://localhost:3000
```

`npm run build` only regenerates `dist/`. `npm start` only serves it.

## 3. Email setup (contact form)

The form sends each inquiry, with every field, to `CONTACT_TO_EMAIL` (default `connectrm96@gmail.com`).
Credentials live **only** in server environment variables (Netlify → Site configuration → Environment variables) — never in the browser code.

### Option A — Gmail SMTP (simplest, free)

1. Sign in to the Gmail account that will **send** the notifications (it can be connectrm96@gmail.com itself).
2. Turn on 2-Step Verification: Google Account → Security → 2-Step Verification.
3. Create an App Password: Google Account → Security → App passwords → name it "Velto website" → copy the 16-character password.
4. Set these environment variables:

```
EMAIL_PROVIDER=smtp
SMTP_HOST=smtp.gmail.com
SMTP_PORT=465
SMTP_USER=connectrm96@gmail.com
SMTP_PASS=abcdefghijklmnop        # the App Password, without spaces
CONTACT_TO_EMAIL=connectrm96@gmail.com
```

Any other SMTP provider works the same way (Zoho Mail: `smtp.zoho.in`, port 465; Brevo: `smtp-relay.brevo.com`, port 587; Microsoft 365: `smtp.office365.com`, port 587).

### Option B — Resend API (best deliverability)

1. Create a free account at resend.com.
2. Add and verify the domain `velto.co.in` (Resend shows the DNS records to add).
3. Create an API key and set:

```
EMAIL_PROVIDER=resend
RESEND_API_KEY=re_xxxxxxxxxxxxxxxx
CONTACT_FROM_EMAIL=Velto Website <noreply@velto.co.in>
CONTACT_TO_EMAIL=connectrm96@gmail.com
```

(Before the domain is verified, you can test with `CONTACT_FROM_EMAIL=Velto Website <onboarding@resend.dev>` — Resend then only delivers to the email address you signed up with.)

### How you reply
Every notification sets **Reply-To** to the visitor, so pressing *Reply* in Gmail answers them directly.

### What happens on failure
If credentials are missing or the email provider is down, the visitor sees a clear error message with your phone, email and WhatsApp as alternatives. The form never shows "success" unless the email was actually accepted.

## 4. Deploy to Netlify

Everything is preconfigured in `netlify.toml`: build command `node build.mjs`, publish folder `dist`, the contact function at `/api/contact`, security headers and the www → apex redirect. There is nothing to install.

**Option 1 — from GitHub (recommended, auto-deploys on every push)**
1. Create a GitHub repository and upload the contents of this folder (not the folder `dist`, and never your `.env`).
2. In Netlify: **Add new site → Import an existing project → GitHub** → pick the repository.
3. Netlify reads `netlify.toml` automatically. Leave the build settings as shown (Build command `node build.mjs`, Publish directory `dist`, Functions directory `netlify/functions`).
4. Before (or right after) the first deploy, go to **Site configuration → Environment variables → Add a variable** and add the email variables from section 3 (for Gmail: `EMAIL_PROVIDER`, `SMTP_HOST`, `SMTP_PORT`, `SMTP_USER`, `SMTP_PASS`, `CONTACT_TO_EMAIL`). Mark `SMTP_PASS` / `RESEND_API_KEY` as secret.
5. **Deploys → Trigger deploy → Deploy site** so the function picks up the variables.
6. Open the site, submit a test inquiry and confirm it reaches connectrm96@gmail.com. If it doesn't, check **Logs → Functions → contact** for the exact error.

**Option 2 — Netlify CLI (no GitHub)**
```bash
npm install -g netlify-cli
netlify login
netlify init            # create/link the site
netlify env:set SMTP_PASS "your-app-password"   # repeat for each variable
netlify deploy --build --prod
```

> Drag-and-drop deploys of the `dist` folder will **not** include the contact function — use Option 1 or 2.

## 5. Connect velto.co.in (Netlify)

1. Netlify → **Domain management → Add a domain** → enter `velto.co.in`. Netlify automatically adds `www.velto.co.in` too.
2. Choose one of these:

**A. Keep DNS at your registrar** (GoDaddy, Hostinger, BigRock, etc.) and add:

| Type | Name / Host | Value |
|------|-------------|-------|
| A *(or ALIAS/ANAME if your registrar supports it)* | `@` | `75.2.60.5` *(ALIAS/ANAME: `apex-loadbalancer.netlify.com`)* |
| CNAME | `www` | `your-site-name.netlify.app` *(shown in Netlify's "Pending DNS verification" box)* |

Delete any old A / CNAME records for `@` and `www`. **Keep MX records** so existing email keeps working.

**B. Use Netlify DNS** — click *Set up Netlify DNS*, then replace the nameservers at your registrar with the four Netlify nameservers shown. Netlify then manages all records (re-add any MX/TXT records you need for email).

3. Wait for propagation (usually under an hour, up to 24 hours). Netlify issues the free HTTPS certificate automatically — check **Domain management → HTTPS**.
4. Set `velto.co.in` as the **primary domain** so `www` redirects to it.
5. Submit `https://velto.co.in/sitemap.xml` in Google Search Console.

## 5b. Alternative: Vercel

1. Put this folder in a GitHub repository (or use the Vercel CLI: `npx vercel`).
2. In Vercel: **Add New → Project → Import** the repository. The settings come from `vercel.json` automatically (Build Command `node build.mjs`, Output `dist`, no install step).
3. **Settings → Environment Variables**: add the variables from section 3 for *Production* (and *Preview* if you want). Redeploy after adding them.
4. Open the deployment, submit a test inquiry and confirm it arrives at connectrm96@gmail.com.

**Connecting the domain on Vercel:**

1. Vercel project → **Settings → Domains** → add `velto.co.in`, then add `www.velto.co.in`.
2. At your domain registrar (GoDaddy, Hostinger, BigRock, etc.) open DNS management and create the records Vercel shows. Typically:

| Type  | Name / Host | Value                  |
|-------|-------------|------------------------|
| A     | `@`         | `76.76.21.21`          |
| CNAME | `www`       | `cname.vercel-dns.com` |

   Always use the exact values Vercel displays for your project if they differ.
3. Remove any old A/CNAME records for `@` and `www` that point elsewhere (keep MX records so email keeps working).
4. Wait for DNS to propagate (minutes to a few hours). Vercel issues the HTTPS certificate automatically.
5. `www.velto.co.in` permanently redirects to `https://velto.co.in` (configured in `vercel.json`).
6. Submit `https://velto.co.in/sitemap.xml` in Google Search Console.

## 6. Alternative: your own server (VPS)

```bash
# Node 18+ on the server
git clone <your-repo> velto && cd velto
cp .env.example .env && nano .env      # add email credentials
npm run build
PORT=3000 node server.mjs              # or keep it running with: pm2 start server.mjs --name velto
```

Put Nginx (or Caddy) in front, proxying `velto.co.in` → `http://127.0.0.1:3000`, with HTTPS from Let's Encrypt. Set `X-Forwarded-For` / `X-Forwarded-Host` headers so rate limiting and origin checks see real values.

> Shared hosting without Node.js (plain cPanel) can serve `dist/` as static files, but the contact form needs Node — use Vercel or a VPS for the full site.

## 7. Editing content

- **Phone, email, WhatsApp, social links:** `src/data.mjs` → `site`. Add your LinkedIn, Instagram and Facebook URLs to `site.social`; until then the footer shows tidy "coming soon" icons (no fake links).
- **Service copy, features, benefits:** `src/data.mjs` → `services`.
- **Page text:** `src/pages/<page>.mjs`.
- After any change: `npm run build` to preview locally; Netlify rebuilds automatically when you push to GitHub.

### Statistics policy
The homepage figures (9 service lines, 17+ HRMS modules, 12 tool types, 100% custom-built) describe Velto's service catalogue — they are true by construction and labelled as such. Mockups in the HRMS, Excel and BPO pages are marked **Sample data / Illustrative**. Add real client metrics, testimonials or logos only once you have permission to publish them.

## 8. Images
All visuals (hero orb, mockups, dashboards, phones, browser previews) are built in HTML/CSS/SVG. They are resolution-independent — perfectly sharp on 4K and retina screens — weigh a few kilobytes, and can never break. To add your own photography later, put optimized `.webp`/`.avif` files in `assets/img/` and use `<img loading="lazy" width height>` or `<picture>` in a page file.

## 9. Brand files (`assets/brand/`)
| File | Use |
|------|-----|
| `velto-logo-dark.svg/.png` | Horizontal logo for dark backgrounds |
| `velto-logo-light.svg/.png` | Horizontal logo for light backgrounds (letterheads, invoices) |
| `velto-logo-dark-bg.svg` | Horizontal logo on ink background |
| `velto-logo-stacked-dark/light.svg/.png` | Square lockup (social profiles, business cards) |
| `velto-symbol*.svg/.png` | Symbol alone (app icon, avatar, watermark) |
| `favicon.svg`, `favicon.ico`, `apple-touch-icon.png`, `icon-192/512.png` | Browser & home-screen icons |
| `og-image.png` | 1200×630 social sharing image |

**Palette:** Ink `#05060A` · Navy `#0B1226` · Electric Blue `#2F6BFF` · Cyan `#22D3EE` · Violet `#8B5CF6` · White `#F5F7FB`
**Typography:** Sora (headings) · Inter (body), both from Google Fonts.
**Symbol concept:** a "V" made of two momentum strokes; the rising stroke launches a detached node forward — technology (node), movement (vector) and growth (ascent).
