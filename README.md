# COSS Office Furniture Landing Page

A single-page landing site for the HON and Lorell office furniture flyers. Visitors arrive by QR code or short URL, mostly on phones, and the page's goal is to get them to request a quote.

Everything is in `index.html`: HTML, CSS, and JS, with no build step. The only external pieces are Google Fonts and EmailJS.

```
furniture landing page/
├── index.html    (the whole page)
├── README.md
└── images/       (create this folder and add your category photos; see below)
```

To test locally, open `index.html` in a browser.

## Editing the config block

All copy and settings are in the `CONFIG` object at the top of the `<script>` near the bottom of `index.html`. Search for `CONFIG — edit everything here`.

| Setting | What it does |
|---|---|
| `OFFER_LAYOUT_CONSULT` | `true` shows "Free quote and layout help," the layout line in the form subheading, and the "I'd like help planning my layout" checkbox. `false` removes all three. |
| `EMAILJS_*` | EmailJS keys (see below). |
| `phone`, `email`, `address`, `hours` | Contact info used in the hero call button, the contact panel, the sticky bar, the error message, and the footer. `phone.tel` is the dialable number (`+1` followed by 10 digits). |
| `mainSiteUrl` / `mainSiteLabel` | Footer "back to website" link. |
| `hero`, `brands`, `categories`, `why`, `form`, `sticky`, `footer` | All page copy. |
| `brands.list[].link` | Optional manufacturer catalog link. Leave it `''` (empty) to hide it, which is the v1 default. |

A few copy strings contain simple HTML, such as the accent `<span>` in the hero headline and the link in the brands footer line. Keep those tags in place when you edit the text around them.

## Swapping placeholder images

Each category tile reads its `image` path from `CONFIG.categories.list`. The defaults are:

```
images/desks.jpg
images/seating.jpg
images/storage.jpg
images/conference.jpg
images/reception.jpg
images/full-office.jpg
```

When a file is missing, the tile shows a striped box labeled "Placeholder image" with the expected filename. To replace one, drop a photo at that path. Photos are cropped to a **4:3** frame, and around 1200×900 px works well. JPG or WebP files under ~200 KB keep the page fast on phones. To use a different filename, change the `image` value in the config.

## EmailJS setup

The quote form sends through EmailJS with no backend. Until real keys are entered, every submission shows the error state with the phone/email fallback.

1. Sign in at [emailjs.com](https://www.emailjs.com). You can use the account the copier wizard uses.
2. **Email Services:** use the existing service or add one. Copy its **Service ID**.
3. **Email Templates:** create a new template. Set **To Email** to the address that should receive leads, set **Reply To** to `{{reply_to}}`, and use a subject like `Furniture quote request: {{name}} ({{furnishing}})`. Example body:

   ```
   New furniture quote request

   Name:          {{name}}
   Company:       {{company}}
   Phone/email:   {{contact}}
   Furnishing:    {{furnishing}}
   Layout help:   {{layout_help}}
   Notes:         {{notes}}

   Flyer source:  {{src}}
   Page:          {{page_url}}
   Submitted:     {{submitted_at}}
   ```

   Copy the **Template ID**.
4. **Account → General:** copy the **Public Key**.
5. Paste all three into `CONFIG.EMAILJS_PUBLIC_KEY`, `EMAILJS_SERVICE_ID`, and `EMAILJS_TEMPLATE_ID`. The public key is meant to be public, so having it in the page is fine.
6. In EmailJS, go to **Account → Security**, add your GitHub Pages domain and your main site's domain to the allowed origins, and send a test submission.

### Template variables sent

| Variable | Value |
|---|---|
| `name`, `company`, `contact`, `furnishing`, `notes` | Form fields |
| `reply_to` | The visitor's email if they entered one, otherwise blank |
| `layout_help` | `Yes` / `No`, or `N/A` when `OFFER_LAYOUT_CONSULT` is false |
| `src` | Value of the `?src=` URL parameter, or `(none)` |
| `page_url`, `submitted_at` | Full page URL and Central time timestamp |

## Flyer tracking

Give each flyer its own link or QR code with a `src` value:

```
https://<your-pages-url>/?src=flyer-oct
https://<your-pages-url>/?src=flyer-trade-show
```

The value arrives in the lead email as `{{src}}`. Allowed characters are letters, numbers, `-`, `_`, and `.`, up to 60 characters.

**When the page is embedded in an iframe**, the iframe reads its own URL, not the host page's. To keep tracking, either point the QR code straight at the GitHub Pages URL, or have the host page pass the parameter into the iframe `src` (snippet below).

## GitHub Pages deployment

1. Create a repo, for example `coss-furniture`, and push `index.html`, `README.md`, and the `images/` folder.
2. In the repo, go to **Settings → Pages → Build and deployment**, choose **Deploy from a branch**, then pick `main` and `/ (root)`. Save.
3. After a minute or so the page is live at `https://<user-or-org>.github.io/coss-furniture/`.
4. Add that domain to EmailJS allowed origins (step 6 above).

To update the page later, edit `index.html` and push. Pages redeploys automatically.

## Embedding on the main website

Link to the page directly, or embed it the same way as the copier wizard. Inside an iframe, the page:

- hides the sticky mobile bar, since a fixed bar would sit at the bottom of a tall iframe instead of the screen
- opens tap-to-call, email, and the "back to website" link in the top window
- posts its height to the parent as `{ type: 'furniture-page-height', height }` so the iframe can size itself without inner scrollbars

Host page snippet:

```html
<iframe id="coss-furniture" title="Office furniture" style="width:100%;border:0;display:block" scrolling="no"></iframe>
<script>
  (function () {
    var base = 'https://<user-or-org>.github.io/coss-furniture/';
    var src = new URLSearchParams(location.search).get('src');
    var frame = document.getElementById('coss-furniture');
    frame.src = base + (src ? '?src=' + encodeURIComponent(src) : '');
    window.addEventListener('message', function (e) {
      if (e.data && e.data.type === 'furniture-page-height') frame.style.height = e.data.height + 'px';
    });
  })();
</script>
```

## Before launch

These items still need decisions or real values (see Part 3 of the spec):

- Real phone, email, address, and hours in `CONFIG`
- EmailJS keys, plus the address that leads should go to (set in the EmailJS template)
- Product photos in `images/`
- Keep or drop the layout consult (`OFFER_LAYOUT_CONSULT`)
- Confirm delivery and setup is offered, or remove that point from `CONFIG.why.points`
- Review the HON/Lorell blurbs, "serving Kansas businesses", "Real people on your quote", and "We can order far more than what's shown here"
- Confirm `mainSiteUrl`
