# The Westin Residences, Gurugram — Landing Page
## Developer Integration Guide (CRM + Google Analytics + Google Ads + Meta Pixel)

This is a **single-file static site**. Everything lives in `index.html` (HTML + CSS + JS inline) plus the `images/` folder. There is no build step, no framework and no dependencies — open it in any browser or drop it on any web host / CDN.

There are exactly **four integration points**. All are already stubbed in the code with `TODO` markers; you only need to paste your IDs/URL in the marked spots. Nothing else in the page needs to change.

> Tip: open `index.html` and search (Ctrl/Cmd-F) for the word **`TODO`** and for **`TRACKING`** — every spot you need to touch is flagged.

---

## 1. Analytics & Ad tags (GA4 + Google Ads + Meta Pixel)

**Where:** `<head>` of `index.html` — look for the comment block near the top:

```html
<!-- TRACKING — replace with WHITELAND's Meta Pixel + Google Ads + GA4 IDs. -->
<!-- <script>/* fbq('init',...); gtag('config','G-XXXX'); ... */</script> -->
```

**What to do:** delete that commented placeholder and paste your real snippets in its place. Use your own IDs:

```html
<!-- Google tag (gtag.js) — GA4 + Google Ads -->
<script async src="https://www.googletagmanager.com/gtag/js?id=G-XXXXXXX"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());
  gtag('config', 'G-XXXXXXX');      // GA4 Measurement ID
  gtag('config', 'AW-XXXXXXXXX');   // Google Ads Conversion ID
</script>

<!-- Meta (Facebook) Pixel -->
<script>
  !function(f,b,e,v,n,t,s){if(f.fbq)return;n=f.fbq=function(){n.callMethod?
  n.callMethod.apply(n,arguments):n.queue.push(arguments)};if(!f._fbq)f._fbq=n;
  n.push=n;n.loaded=!0;n.version='2.0';n.queue=[];t=b.createElement(e);t.async=!0;
  t.src=v;s=b.getElementsByTagName(e)[0];s.parentNode.insertBefore(t,s)}(window,
  document,'script','https://connect.facebook.net/en_US/fbevents.js');
  fbq('init', 'YOUR_PIXEL_ID');
  fbq('track', 'PageView');
</script>
```

> You can also do all of the above through **Google Tag Manager** instead — just paste your GTM container snippet here. The page already fires the right dataLayer/gtag/fbq events (see §3), so GTM triggers will pick them up.

---

## 2. CRM / lead destination (where the form submissions go)

**Where:** in the `<script>` near the bottom of `index.html`:

```js
// form wiring
var LEAD_ENDPOINT = "";   // TODO: Whiteland CRM / Apps Script
```

**What to do:** set `LEAD_ENDPOINT` to the URL of your CRM's inbound webhook / lead-capture endpoint. As soon as it is non-empty, every valid form submission is POSTed there automatically.

```js
var LEAD_ENDPOINT = "https://your-crm.example.com/api/leads";
```

**What gets sent** (HTTP `POST`, `Content-Type: application/x-www-form-urlencoded`):

| Field           | Example                          | Notes                                  |
|-----------------|----------------------------------|----------------------------------------|
| `firstName`     | `Rahul Sharma`                   | Full name entered by the user          |
| `phone`         | `98xxxxxxxx`                     | Phone number entered                   |
| `residenceType` | `3 BHK` / `4 BHK`               | Configuration selected                 |
| `page_url`      | `https://.../?utm_source=...`   | Full landing URL **incl. UTM params**  |
| `referrer`      | `https://google.com/`           | Where the visitor came from            |

Notes for whoever wires the endpoint:
- The request is sent with `mode:'no-cors'` so the browser won't block a cross-domain POST. This means the page **cannot read the response** — it always shows the user a success message. If you need read-back/validation, host the endpoint on the **same domain** as the page (or enable CORS) and we can switch it to a normal `fetch`.
- **Salesforce / Zoho / HubSpot etc.:** point `LEAD_ENDPOINT` at your Web-to-Lead URL, or at a small middleware (e.g. a Google Apps Script Web App, a Zapier/Make webhook, or a Lambda) that forwards into the CRM. Map the five fields above to your CRM's fields.
- **UTM tracking:** `page_url` already carries the full query string, so your CRM will capture `utm_source`, `utm_campaign`, `gclid`, etc. as long as they're in the ad's landing URL.
- Two forms use this same wiring: the main "Register your interest" form (`#mainForm`) and the 10-second popup form (`#popupForm`). You only set the endpoint once.

---

## 3. Conversion events that already fire (no code needed)

The page **already** fires these events on the correct user actions. Once your tags from §1 are live, these light up automatically — just create the matching conversion actions in Google Ads / GA4 / Meta:

| User action                | Google (gtag) event      | Meta (fbq) event |
|----------------------------|--------------------------|------------------|
| Submits a lead form        | `generate_lead`          | `Lead`           |
| Taps **WhatsApp**          | `whatsapp_click`         | `Contact`        |
| Taps **Call**              | `phone_click`            | `Contact`        |

**To count form submits as a Google Ads conversion:** in Google Ads create a conversion action, then either (a) import the GA4 `generate_lead` event, or (b) add your `gtag('event','conversion',{send_to:'AW-XXXX/label'})` call inside the `fireLead()` function (it's right above the wiring, clearly named).

---

## 4. Contact numbers (verify before launch)

The page currently uses this number for the Call and WhatsApp buttons. **Confirm it's the correct sales line and replace if needed** — search `index.html` for these:

- Phone: `tel:+918448190926`
- WhatsApp: `wa.me/918448190926`

---

## Testing checklist (before going live)

1. Open the page, submit the form with a test name + phone → confirm the lead lands in your CRM.
2. Open browser DevTools → Network tab → submit again → confirm the `POST` to your `LEAD_ENDPOINT` fires with the 5 fields.
3. Use **Google Tag Assistant** / **GA4 DebugView** → confirm `page_view` and `generate_lead` fire.
4. Use the **Meta Pixel Helper** Chrome extension → confirm `PageView` and `Lead` fire.
5. Tap WhatsApp and Call on a phone → confirm `Contact` / `whatsapp_click` / `phone_click` fire.

---

## Deploying / hosting

It's plain static files — host anywhere:
- **Any web server / cPanel:** upload `index.html` + the `images/` folder to the web root.
- **Netlify / Vercel / Cloudflare Pages / S3+CloudFront / GitHub Pages:** drag-and-drop or connect the folder; no build command, publish directory is the root.
- Keep the `images/` folder next to `index.html` (paths are relative).

Questions on any of the above — contact Fixit (hello@fix-it.ai).
