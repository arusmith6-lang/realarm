# ARM Transport website

One self-contained file, `index.html`. No build step, no dependencies to install.
The 3D libraries (Three.js, GSAP) and fonts load from public CDNs at runtime; everything else is in the file.

## GitHub

This folder is the whole repository: commit it as it is (the `.glb` models are 2–6 MB each, well under GitHub's 100 MB file
limit, so no Git LFS). Either upload the folder through the GitHub website ("Add file → Upload files" on a new empty
repository — drag the folder's *contents* in, including the `models/` and `assets/` folders), or from a terminal:

    git init
    git add .
    git commit -m "ARM Transport website"
    git branch -M main
    git remote add origin https://github.com/<your-account>/arm-transport-website.git
    git push -u origin main

### Hosting straight from GitHub (GitHub Pages)

1. Repository → **Settings → Pages**. Under *Build and deployment* choose **Deploy from a branch**, branch `main`, folder `/ (root)`, Save.
2. After a minute the site is live at `https://<your-account>.github.io/arm-transport-website/`. Open it with `#debug` at the
   end: the grey status line bottom-right should list all four models as "ready".
3. Custom domain: add it in the same Pages screen and create the DNS records GitHub shows there (a CNAME for `www`, A records for
   the bare domain); tick **Enforce HTTPS** once the certificate is issued. GitHub writes a `CNAME` file into the repo for it.

`.nojekyll` is included so GitHub serves the files exactly as they are. The `_headers`, `vercel.json` and `.htaccess` files are
ignored by GitHub Pages; they only matter on Netlify/Cloudflare, Vercel and Apache hosts respectively. Pages does not set a
long cache lifetime for the models, which is fine: the browser still caches them for repeat visits.

### Updating the 3D models later

The models are plain files: replace `models/*.glb` (and `models/lite/*.glb` for phones) keeping the same file names, commit and
push, and the live site picks them up. The vehicle lengths the page scales to are in `CONFIG.modelLengths` in `index.html` —
change them only if a model's real length changes.

## Deploy elsewhere

Upload the whole folder: `index.html`, `assets/`, the `models/` folder (four `.glb` files plus the `lite/` set, 23 MB) and the config file for your
host. The page loads the models from `models/…` on the same domain, so nothing depends on any other server.

1. Create a GitHub repository and add the folder (the model files are 2–6 MB each; do not use Git LFS).
2. Import it on the host:
   - **Netlify** — "Add new site → Import an existing project". Build command: *none*. Publish directory: `/`. The `_headers` file sets caching and CORS for `/models/*`; Netlify serves `.glb` as `model/gltf-binary`.
   - **Vercel** — "Add New → Project". Framework: *Other*, no build command, output directory `.`. `vercel.json` sets the `Content-Type: model/gltf-binary` and cache headers.
   - **Cloudflare Pages** — "Create a project → Connect to Git". Framework: *None*, no build command, output directory `/`. `_headers` applies here too.
   - **cPanel / Apache / LiteSpeed** — upload the folder to `public_html`; `.htaccess` adds `AddType model/gltf-binary .glb` and caching. Make sure the files are world-readable (permissions 644, folders 755).
   - **nginx** — add `model/gltf-binary glb;` to `mime.types` (or `types { model/gltf-binary glb; }` in the server block).
3. Point the domain at the host; each one issues HTTPS automatically.
4. Check: open `https://your-domain/models/man_tge_van.glb` in a browser. It should download (not show 403/404). Then open the site with `#debug` at the end of the URL: the grey status line bottom-right lists each model as "ready".

Note: the preview copy of the site hosted on claude.ai cannot serve the model files (that host returns HTTP 403 for them), so the real trucks only appear once the folder is on your own hosting.

If the models must live on a different domain or a CDN, set `modelBase: 'https://cdn.example.com/'` in `CONFIG` and make sure that server sends `Access-Control-Allow-Origin` for the site's domain (the included config files already do for the same host).

## Before launch — settings inside `index.html`

All settings are in the `CONFIG` object at the top of the `<script>` block near the end of the file.

### 1. Connect the quote form (required before launch)

Every "Get a quote" button opens the quote page (also reachable at `/#quote`). Until a delivery service is connected,
the page says so above the form and submitting opens a pre-filled email to `CONFIG.leadEmail` in the visitor's own
email app. That works, but it depends on the visitor having an email app set up, so connect a service before launch.

**Recommended: Formspree** (works on any host, nothing secret goes in the file)

1. Go to https://formspree.io and create a free account (50 submissions/month on the free plan; paid plans from about $10/month if more are needed).
2. Click **New form**. Name it "ARM Transport quote" and set the **recipient email** to `info@armtransport.co.uk`
   (or whichever address is confirmed). Formspree emails that address a confirmation link the first time; it must be clicked.
3. Copy the form's endpoint, which looks like `https://formspree.io/f/abcdwxyz`.
4. In `index.html`, find `leadEndpoint: '',` in `CONFIG` and paste it in: `leadEndpoint: 'https://formspree.io/f/abcdwxyz',`
   Check `leadEmail` holds the same address; it is shown on the page and used as the fallback.
5. In the Formspree form settings, turn on **Restrict to domain** and enter the site's domain, so the endpoint cannot be used from other sites.
6. Deploy, open the live site, click **Get a quote**, fill it in with real-looking test data and send. You should see the
   "Request sent" panel with a reference, and within a minute an email titled "Quote request ARM-…" should arrive at the
   recipient address (check the spam folder the first time). The same submission is also listed under **Submissions** in Formspree.
7. Reply to the test email: it goes to the customer's address because the form sets the reply-to.

Formspree form IDs are public by design (they only accept submissions), so leaving one in the HTML is normal. Do not put
any service's secret API key in the file; this site never needs one.

**Alternative: Netlify Forms** (only if hosted on Netlify): set `leadEndpoint: 'netlify'`. Submissions appear under
*Forms* in the Netlify dashboard; add the company email under *Form notifications*. Free tier: 100 submissions/month.

**Other services** (Basin, Getform, a Make or Zapier webhook, your own API) accept the same JSON POST: set `leadEndpoint`
to their URL. Fields sent: `reference, name, company, email, phone, collection_postcode, collection_address, collection_date,
delivery_postcode, delivery_address, delivery_date, goods, quantity, weight, vehicle, requirements, notes, subject, summary,
_subject, _replyto, page, sent`. `summary` is a plain-text version of the whole request.

**Spam and duplicates**: the form has a honeypot field (`_gotcha`, which Formspree and Netlify also recognise), rejects
submissions made within three seconds of opening, disables the button while sending, and will not send the same request
twice within ten minutes from the same browser tab. Formspree adds its own spam filtering; reCAPTCHA can be turned on in
its settings if junk still gets through.

### 2. Company details

Phone, address and map link are already set in `CONFIG.company` and in the page text (`0161 706 0601`, Yard 1, Excel Center, Preston Street, Manchester, M18 8DB). If anything changes, search the file for the old value.

### 3. Copy marked `<!-- CONFIRM: … -->`

Search `index.html` for `CONFIRM`. Each comment sits next to wording about services, vehicles or standards that has not been verified with the company. Edit or delete the wording as needed, then remove the comment. The footer has a commented-out line for the legal entity name, company number, VAT number and operator licence number: fill it in and uncomment it.

### 4. Analytics (optional)

Leave `analytics.gtagId` empty and the site sets no cookies and shows no banner.
To use Google Analytics 4, set `analytics: { gtagId: 'G-XXXXXXXXXX' }`. Visitors then see a consent bar; GA loads only after they accept, their choice is remembered in their browser, and "Cookie settings" appears in the footer so they can change it. Add a short cookies paragraph to the privacy notice at the same time (it currently has none, because the site sets no cookies).
A cookieless tool such as Plausible or Fathom needs no consent bar; add its script tag in `<head>` instead.

### 5. The MAN fleet models (included)

`models/` holds the four vehicles of the fleet, already wired into `CONFIG.models`. Each one scrolls in at its own stop:

| File | Slot | Shown at | Size | Moving parts |
|---|---|---|---|---|
| `models/man_tgx_tractor.glb` | tractor | hero (Overview), Fleet standards, line-up | 5.3 MB | wheels |
| `models/man_curtainsider_trailer.glb` | trailer | Full loads (coupled to the tractor file above) | 0.2 MB | wheels, curtains (`CurtainOpen`), rear doors (`DoorsOpen`), 26 pallets |
| `models/man_tgm_rigid_curtainsider.glb` | curtainsider | Pallets (18 t rigid with tail-lift) | 5.4 MB | wheels, curtains, rear doors, tail-lift loading two pallets (`PalletsLoad`) |
| `models/man_tge_van.glb` | van | Same day | 3.6 MB | wheels, rear doors (`DoorsOpen`), cargo interior |

The cabs come from MAN reference photos (Meshy image-to-3D, multi-view); chassis, axles, wheels, fifth wheel, trailer,
bodies, curtains, doors, tail-lift, pallets and the logo decals were modelled in Blender at 1:1 scale, so
`CONFIG.modelLengths` simply holds each file's real length. Geometry is Draco-compressed (the decoder loads from the
jsDelivr CDN like Three.js itself) and textures are JPEG: 4K colour, 2K normal, 1K roughness/metal.
`models/lite/` holds the same four with 2K/1K textures (0.2–3 MB each); phones and small touch screens load these
automatically (`CONFIG.modelsLite`). All 8 files together: 23 MB; a desktop visitor downloads the 15 MB full set once,
then the browser caches it for a year (see the host config files). The artic is the tractor file plus the trailer file:
the page couples them itself (`TRAILER_OFFSET` / `ARTIC_T` in the script), so the tractor unit is downloaded only once.

The animations are driven by the page scroll: the curtains open on the Full loads stop, the rigid lowers its tail-lift,
loads two pallets and closes up on the Pallets stop, and the van opens its rear doors on the Same day stop. Wheels turn as
the vehicles drive in and out. Every part is a named node in the file (`FRONT_LEFT_WHEEL`, `REAR_DOOR_LEFT`,
`TAIL_LIFT_PLATFORM`, `PALLET_01`…, `LOGO_DOOR_LEFT`…), so the Blender files in the production folder can be edited and
re-exported without touching the page.

To replace a model: export a GLB with the nose along +z and the wheels on y = 0, set the real length in
`CONFIG.modelLengths` and, if needed, `forward` / `offset` in `CONFIG.modelOptions.<slot>`. Open the browser console: each
model logs one line, e.g. `[model trailer] 6 wheel node(s); doors: doorL, doorR; clips in file: CurtainOpen, PalletsLoad,
DoorsOpen; scroll-driven: curtain, pallets, doors`.

### 6. Official logo

`assets/arm-logo.png` is the ARM Transport artwork (transparent PNG, 2580 px wide) and `CONFIG.logoUrl` points at it. The
site paints it onto the logo decals of the four MAN models: one on each cab door, next to the door handle, and nowhere
else (no livery on the trailer, bodies or doors). To swap the file, replace `assets/arm-logo.png` with another PNG or SVG
of the same proportions (about 1.85:1) and keep the path, or change `logoUrl`.

## What happens when something fails

- No WebGL, or the CDN libraries cannot load (each has two mirrors): the page shows the same content as an ordinary scrolling site, with the form, privacy notice and footer unchanged, and a one-line note in the hero.
- Form endpoint down: the visitor is told and given the phone number (or the email fallback if `leadEmail` is set).

## Checked

- Syntax and logic: `node --check`, stub harness in 3D / no-libraries / no-WebGL modes, form in Formspree, Netlify, email and unconnected modes, consent flow.
- Layout: desktop (1440 px) and mobile (390 px) screenshots of every section and of the fallback page.
- Links: every `#anchor` resolves; external links are the Google Maps reference, ico.org.uk, the CDNs and Google Fonts.
- Model loading path: exercised with a simulated GLB carrying animation clips (stub harness); the clip detection and scroll-scrub code runs without errors. A real GLB has not been loaded here.
- Not possible in the build environment: a live WebGL render. Please view the 3D stops on a real device and report any camera framing that looks off.
