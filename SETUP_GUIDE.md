# Wedding Website — Setup Guide

## Overview

Your wedding website (`index.html`) is a single-page site inspired by Zola wedding websites.
It includes: **Hero** → **Our Story** → **Details** → **Schedule** → **Travel** → **RSVP Form** → **Q&A** → **Footer**.

The RSVP form submits to **Google Forms** (free), and responses go directly to a
**Google Sheet** you can share with Kaia.

---

## 1. Free Hosting Options

| Service | Cost | Custom Domain | HTTPS | How |
|---|---|---|---|---|
| **GitHub Pages** ⭐ RECOMMENDED | Free | Yes (CNAME) | Yes | Push to repo → Settings → Pages |
| **Netlify** | Free | Yes | Yes | Drag & drop folder or connect Git |
| **Cloudflare Pages** | Free | Yes | Yes | Connect Git repo |
| **Vercel** | Free | Yes | Yes | Import repo |
| **Google Firebase Hosting** | Free (spark plan) | Yes | Yes | `firebase deploy` CLI |
| **Render** (static) | Free | Yes | Yes | Connect Git |
| **Surge.sh** | Free | surge.sh subdomain | Yes | `npx surge` CLI |

### Recommended: **GitHub Pages** (you already have a GitHub repo!)

Since your files are already at `personal_docs/wedding stuffs/`, you can either:

**Option A — Use the existing repo (simplest)**
1. Push the `wedding form html/` folder to GitHub
2. Go to repo **Settings → Pages**
3. Set source to your branch + folder
4. Your site will be at `https://yourusername.github.io/personal_docs/wedding form html/`

**Option B — Dedicated repo (cleaner URL)**
1. Create a new repo named e.g. `luca-and-kaia-wedding`
2. Copy the `wedding form html/` contents into it
3. Enable Pages → site will be at `https://yourusername.github.io/luca-and-kaia-wedding/`

**Option C — Netlify drag & drop (easiest, no Git needed)**
1. Go to https://app.netlify.com/drop
2. Drag the `wedding form html` folder onto the page
3. Done! You get a URL like `https://random-name.netlify.app`
4. You can set a custom subdomain: `luca-and-kaia.netlify.app`

> ⚠️ **IMPORTANT**: If your repo is PUBLIC, the RSVP data goes to Google Sheets (not stored on GitHub). The HTML source code will be visible, but that's fine — it contains no secrets. The Google Form URL is not sensitive.

---

## 2. Setting Up Google Forms for RSVP

This is the key part. Google Forms acts as your **free backend** — no server needed!

### Step 1: Create the Google Form

1. Go to [Google Forms](https://docs.google.com/forms/)
2. Click **+ Blank form**
3. Title it: `Luca & Kaia Wedding RSVP`
4. Add these questions **in this exact order**:

| # | Question | Type | Required? |
|---|---|---|---|
| 1 | Full Name | Short answer | ✅ Yes |
| 2 | Email | Short answer | ✅ Yes |
| 3 | Will you attend? | Short answer | ✅ Yes |
| 4 | Number of guests | Short answer | No |
| 5 | Names of additional guests | Short answer | No |
| 6 | Dietary restrictions | Paragraph | No |
| 7 | Need shuttle bus? | Short answer | No |
| 8 | Message for the couple | Paragraph | No |

> **Why "Short answer" for attending/shuttle?** Because we're sending values like "yes"/"no" from our custom HTML form. Google Forms just receives the text. You'll see nice data in the spreadsheet.

### Step 2: Get the Form Action URL and Entry IDs

1. Open your Google Form
2. Click the **three dots (⋮)** → **Get pre-filled link**
3. Fill in ANY text in each field (e.g., "test1", "test2", etc.)
4. Click **Get link** → **Copy link**
5. The URL will look like:

```
https://docs.google.com/forms/d/e/1FAIpQLSe.../viewform?entry.123456=test1&entry.789012=test2&entry.345678=test3...
```

6. From this URL, extract:
   - **Form action URL**: `https://docs.google.com/forms/d/e/1FAIpQLSe.../formResponse`
     (replace `viewform?...` with just `formResponse`)
   - **Entry IDs**: the `entry.XXXXXXX` numbers for each field

### Step 3: Update index.html

Open `index.html` and find this section near the bottom:

```javascript
var GOOGLE_FORM_ACTION_URL = 'YOUR_GOOGLE_FORM_URL_HERE';

var FIELD_MAP = {
  name:        'entry.XXXXXXXXXX',
  email:       'entry.XXXXXXXXXX',
  attending:   'entry.XXXXXXXXXX',
  guests:      'entry.XXXXXXXXXX',
  guest_names: 'entry.XXXXXXXXXX',
  dietary:     'entry.XXXXXXXXXX',
  shuttle:     'entry.XXXXXXXXXX',
  message:     'entry.XXXXXXXXXX'
};
```

Replace with your actual values, e.g.:

```javascript
var GOOGLE_FORM_ACTION_URL = 'https://docs.google.com/forms/d/e/1FAIpQLSe_YOURFORMID/formResponse';

var FIELD_MAP = {
  name:        'entry.123456789',
  email:       'entry.234567890',
  attending:   'entry.345678901',
  guests:      'entry.456789012',
  guest_names: 'entry.567890123',
  dietary:     'entry.678901234',
  shuttle:     'entry.789012345',
  message:     'entry.890123456'
};
```

### Step 4: Link Form to Google Sheets

1. In your Google Form, click **Responses** tab
2. Click the **Google Sheets icon** (📊) → "Create a new spreadsheet"
3. This creates a live spreadsheet that auto-updates with every RSVP!
4. Share the spreadsheet with Kaia

### Step 5: (Optional) Email notifications

1. In the Google Form Responses tab, click **three dots (⋮)**
2. Enable **"Get email notifications for new responses"**
3. You'll get an email every time someone RSVPs!

---

## 3. How the Form Technically Works

```
┌──────────────┐     POST (no-cors)     ┌──────────────────┐
│  Your HTML   │ ──────────────────────► │  Google Forms    │
│  (index.html)│                         │  /formResponse   │
└──────────────┘                         └────────┬─────────┘
                                                  │ auto-saves
                                                  ▼
                                         ┌──────────────────┐
                                         │  Google Sheets   │
                                         │  (your RSVP list)│
                                         └──────────────────┘
```

**How it works:**
- The HTML form collects user input in your beautiful custom design
- On submit, JavaScript sends a `POST` request to the Google Form endpoint
- `mode: 'no-cors'` is required because Google Forms doesn't allow CORS
- Google Forms receives the data and saves it to the linked Google Sheet
- The user sees a success message in your styled form (not Google's ugly confirmation page)

**Why `no-cors`?**
Google Forms doesn't send CORS headers, so the browser blocks reading the response.
But the data IS sent successfully! The `fetch` call returns an "opaque" response, which
triggers `.then()` (success). We can't read if Google actually accepted it, but in practice
it always works.

**Limitations:**
- You can't validate server-side if Google accepted the submission (but it always does)
- If someone disables JavaScript, the form won't work (very rare)
- Rate limiting: Google Forms has generous limits (~unlimited for personal use)

---

## 4. Alternative: Google Apps Script (advanced)

If you want **real confirmation** that the data was saved, or want to send a custom
thank-you email to guests, you can use Google Apps Script as a middleware:

1. Create a Google Sheet for responses
2. Go to **Extensions → Apps Script**
3. Create a `doPost(e)` function that:
   - Receives the form data
   - Writes it to the sheet
   - Returns a JSON response
   - (Optionally) sends a confirmation email

This is more complex but gives you full control. The Google Forms approach above is
sufficient for most weddings.

---

## 5. Adding Your Hero Image

The hero section references `hero.jpg`. To add a background image:

1. Place your photo in the `wedding form html/` folder as `hero.jpg`
2. Best dimensions: **1920×1080** or wider (landscape)
3. The CSS applies a dark gradient overlay, so even bright photos look good
4. If you don't have a photo, the gradient alone looks elegant

Good options for `hero.jpg`:
- A photo of Italy / Le Marche countryside
- The Abbazia di Sant'Elena
- A photo of you two together
- An abstract floral/botanical image

---

## 6. Customization Guide

### Change colors
Edit the `:root` variables at the top of the CSS:
```css
--gold: #c9a96e;        /* Primary accent */
--gold-dark: #b8963e;   /* Darker gold */
--red: #b22222;         /* Chinese accent red */
```

### Change content
All text is in the HTML body — search for the section you want to change.
Key sections:
- **Our Story**: Find `id="story"` — edit the `story-text` paragraph
- **Details**: Find `id="details"` — change times, venue info
- **Schedule**: Find `id="schedule"` — add/remove schedule items
- **Travel**: Find `id="travel"` — update airport/hotel info
- **FAQ**: Find `id="faq"` — add/remove Q&A items

### Add a Chinese version
You could add a language toggle that shows/hides Chinese text blocks,
or create a separate `index_cn.html` page.

---

## 7. Testing Checklist

- [ ] Open `index.html` locally — verify all sections render
- [ ] Test on mobile (Chrome DevTools → toggle device toolbar)
- [ ] Create Google Form and get entry IDs
- [ ] Update `GOOGLE_FORM_ACTION_URL` and `FIELD_MAP` in index.html
- [ ] Submit a test RSVP — check Google Sheet receives data
- [ ] Deploy to GitHub Pages / Netlify
- [ ] Test the deployed URL on phone
- [ ] Share URL with a friend for a test RSVP
- [ ] Send to guests!

---

## 8. Quick Deploy to Netlify (5 minutes)

```
1. Go to https://app.netlify.com/drop
2. Drag the "wedding form html" folder onto the page
3. Wait 30 seconds
4. Click "Site settings" → "Change site name"
5. Set it to: luca-and-kaia (→ luca-and-kaia.netlify.app)
6. Done! Share the URL with guests
```

## Quick Deploy to GitHub Pages (if you already push this repo)

```
1. Go to your GitHub repo → Settings → Pages
2. Source: Deploy from a branch
3. Branch: main (or master) → / (root) or /docs
4. If using /docs, rename "wedding form html" to "docs"
5. Save → wait 1 minute → your site is live!
```

---

## Summary

| Component | Tool | Cost |
|---|---|---|
| Website HTML | `index.html` (this file) | Free |
| Form backend | Google Forms | Free |
| Response storage | Google Sheets (auto-linked) | Free |
| Hosting | GitHub Pages or Netlify | Free |
| Domain (optional) | Namecheap / Google Domains | ~€10/year |

**Total cost: €0** (or ~€10/year if you want a custom domain like `lucaandkaia.com`)
