# Baby Courtney Splish Splash Gender Reveal

Live boy-or-girl prediction dashboard for the Baby Courtney gender reveal and pool party.

Guests vote through a Google Form using either a QR code or a direct link. Responses are written to Google Sheets, aggregate totals are read through the Google Sheets API by a Cloudflare Worker, and the live results are displayed on a responsive GitHub Pages dashboard at the custom domain.

## Live Links

- **Dashboard:** https://vote.courtney.biz
- **Voting Form:** https://forms.gle/dVeoDoYPM2qgviJa9

## Features

- Live Team Boy and Team Girl vote counts
- Live percentages and progress bars
- Total predictions cast
- Current leader display
- "Voting Is Open" state before the first vote
- QR code linking directly to the voting form
- Direct **Open Voting Form** button for desktop/mobile visitors
- Dad and Mom baby photos
- Responsive desktop, TV, and mobile layouts
- Splish Splash pool-party visual theme
- Automatic refresh every 3 seconds
- Connection-loss warning if the live API becomes unavailable
- Custom domain with HTTPS

## Architecture

```text
Guest
  |
  | scans QR code or opens voting link
  v
Google Form
  |
  | writes response
  v
Google Sheets
  |
  | Data tab
  | Dashboard Data formulas
  v
Google Sheets API
  |
  | authenticated with a read-only service account
  v
Cloudflare Worker
  |
  | returns aggregate JSON
  v
GitHub Pages
  |
  v
https://vote.courtney.biz
  |
  v
TV / Desktop / Mobile Browser
```

## Technology Stack

| Component | Purpose |
|---|---|
| Google Forms | Guest voting interface |
| Google Sheets | Stores responses and calculates aggregate totals |
| Google Sheets API | Provides current aggregate values |
| Google Cloud Service Account | Read-only authentication to the spreadsheet |
| Cloudflare Worker | Public JSON API between Sheets and the website |
| GitHub Pages | Hosts the static dashboard |
| GoDaddy DNS | Hosts DNS for `courtney.biz` |
| GitHub Pages Custom Domain | Publishes the dashboard at `vote.courtney.biz` |

## Repository Files

The repository should contain at least:

```text
baby-courtney-gender-reveal/
├── CNAME
├── README.md
├── index.html
├── dad-baby.JPEG
├── mom-baby.jpg
└── vote-qr.png
```

Optional design/reference assets may also be stored in the repository, such as the Splish Splash Google Forms header.

### `index.html`

Complete dashboard application containing:

- layout and responsive CSS
- Splish Splash pool-party theme
- live vote API integration
- three-second polling
- QR code display
- direct voting link
- current leader logic
- connection-error handling

### `dad-baby.JPEG`

Dad's baby photo. Filename and capitalization matter because web asset paths are case-sensitive.

### `mom-baby.jpg`

Mom's baby photo.

### `vote-qr.png`

QR code that points to:

```text
https://forms.gle/dVeoDoYPM2qgviJa9
```

### `CNAME`

Contains:

```text
vote.courtney.biz
```

## Google Form

### Form

**Baby Courtney Gender Prediction**

Primary question:

```text
What's your prediction?
```

Choices:

```text
Boy
Girl
```

The form is intentionally simple so guests can vote quickly.

### Public Voting Link

```text
https://forms.gle/dVeoDoYPM2qgviJa9
```

The QR code and the website's direct voting button both point to this URL.

## Google Sheets

The response workbook contains two important tabs.

### `Data`

This tab is owned by Google Forms.

Expected structure:

| Timestamp | What's your prediction? |
|---|---|
| Form-generated | Boy or Girl |

Do not change the headers or restructure this tab.

### `Dashboard Data`

This tab exposes only aggregate values used by the dashboard.

| Metric | Value |
|---|---:|
| Boy | Formula |
| Girl | Formula |
| Total | Formula |

Recommended formulas:

**Boy**

```gs
=COUNTIF(Data!B:B,"Boy*")
```

**Girl**

```gs
=COUNTIF(Data!B:B,"Girl*")
```

**Total**

```gs
=SUM(B2:B3)
```

The wildcard allows the formulas to keep working if the form choices include additional text or emoji after "Boy" or "Girl."

## Google Cloud

A Google Cloud project supports the live dashboard.

### Google Sheets API

The **Google Sheets API** must remain enabled.

### Service Account

Service account:

```text
gender-reveal-dashboard
```

Purpose:

```text
Read-only access to Baby Courtney gender reveal vote totals
```

The Google Sheet is shared directly with this service account using **Viewer** access.

### Service Account Key

A JSON service-account key is used by Cloudflare for authentication.

**Never commit the JSON key or private key to GitHub.** Treat it like a password.

If the key is ever exposed:

1. Delete/revoke the exposed key in Google Cloud.
2. Create a replacement key.
3. Update the corresponding Cloudflare secret.

## Cloudflare Worker

Worker:

```text
baby-courtney-vote-api
```

The Worker authenticates to Google using the service account, reads the `Dashboard Data` range, and returns only aggregate values.

Typical response:

```json
{
  "boy": 4,
  "girl": 3,
  "total": 7,
  "updatedAt": "2026-08-22T..."
}
```

The website polls this endpoint every 3 seconds.

### Worker Variables and Secrets

| Variable | Type | Purpose |
|---|---|---|
| `GOOGLE_CLIENT_EMAIL` | Secret | Service account email |
| `GOOGLE_PRIVATE_KEY` | Secret | Service account private key |
| `GOOGLE_SPREADSHEET_ID` | Secret | ID of the response spreadsheet |
| `GOOGLE_SHEET_RANGE` | Text | `Dashboard Data!A1:B4` |

No secret values belong in this repository.

## GitHub Pages

Repository:

```text
baby-courtney-gender-reveal
```

Publishing configuration:

| Setting | Value |
|---|---|
| Source | Deploy from a branch |
| Branch | `main` |
| Folder | `/ (root)` |
| Custom domain | `vote.courtney.biz` |
| HTTPS | Enforced |

Changes committed to `main` are published through GitHub Pages.

## Custom Domain

Custom domain:

```text
vote.courtney.biz
```

GoDaddy DNS record:

| Type | Name | Value |
|---|---|---|
| CNAME | `vote` | `newtguppy.github.io` |

The root domain `courtney.biz` is not changed by this configuration.

### DNS Verification

From macOS:

```bash
dig vote.courtney.biz CNAME
```

Expected result:

```text
vote.courtney.biz.  IN  CNAME  newtguppy.github.io.
```

## Dashboard Behavior

### Before Any Votes

The footer displays:

```text
Voting Status
Voting Is Open
```

### After Voting Begins

The status becomes one of:

```text
Current Leader
Team Boy +N
```

```text
Current Leader
Team Girl +N
```

or, when tied:

```text
It's a Tie!
```

### Live Refresh

The dashboard polls the Cloudflare Worker every:

```text
3000 ms
```

No manual browser refresh should be required for new votes.

### Connection Failure

Normal technical update timestamps are hidden.

If the API becomes unreachable, the dashboard displays:

```text
Connection lost. Reconnecting...
```

The dashboard continues retrying automatically.

## Responsive Behavior

### TV / Desktop

The full dashboard includes:

- both baby photos
- Team Boy / Team Girl panels
- vote counts
- percentages
- progress bars
- QR code
- direct voting button
- total predictions
- voting status/current leader

### Mobile

The layout stacks vertically.

The QR code may be hidden on narrow screens because a user already viewing the site on a phone cannot conveniently scan a QR code displayed on that same phone.

The direct **Open Voting Form** control remains available.

## Theme

The design follows the **Splish Splash** pool-party gender reveal invitation.

Core visual elements include:

- watercolor blue and pink
- pool-water accents
- water splashes
- pool floats
- swimwear/clothesline details
- navy typography
- soft cream/gold accents
- coordinated Team Boy and Team Girl panels

A matching Google Forms header is used so the form and dashboard feel like one event experience.

## Updating Baby Photos

To replace a photo while keeping the same filename:

1. Upload the replacement image to GitHub.
2. Keep the expected filename:
   - `dad-baby.JPEG`
   - `mom-baby.jpg`
3. Increment the cache-busting version in `index.html`.

Example:

```html
<img src="dad-baby.JPEG?v=5" alt="Dad as a baby">
```

Changing the `?v=` number forces browsers to retrieve the new image instead of displaying an older cached copy.

## Updating the Voting Form URL

If the Google Form URL ever changes:

1. Update every voting-form link in `index.html`.
2. Generate a new QR code.
3. Replace `vote-qr.png`.
4. Increment the QR cache-busting version in `index.html`.
5. Commit the changes to `main`.
6. Test both the QR code and direct voting button.

## Resetting Test Votes

Before the party:

1. Open the Google Sheet.
2. Open the `Data` tab.
3. Delete only the test-response rows beginning at row 2.
4. Keep the header row.
5. Do not delete the `Data` sheet.
6. Do not delete the `Dashboard Data` formulas.

After deleting the test rows, the dashboard should automatically return to:

```text
Team Boy: 0
Team Girl: 0
Predictions Cast: 0
Voting Is Open
```

## Party-Day Checklist

- Confirm `https://vote.courtney.biz` loads over HTTPS.
- Confirm the Google Form accepts responses.
- Confirm the Cloudflare Worker returns current JSON.
- Confirm one Boy test vote appears automatically.
- Confirm one Girl test vote appears automatically.
- Delete all test votes.
- Verify the dashboard returns to 0 / 0.
- Confirm the QR code scans from the TV.
- Confirm the direct voting button works.
- Test the dashboard from a phone.
- Connect the laptop to the TV using HDMI.
- Use browser full-screen mode.
- Set browser zoom to 100%.
- Disable laptop sleep.
- Disable screen locking.
- Disable display dimming.
- Disable notifications.
- Keep the laptop connected to power.
- Confirm Wi-Fi coverage at the TV and guest area.
- Keep a phone hotspot available as a backup.

## Troubleshooting

### Vote appears in Google Sheets but not on the dashboard

Check in this order:

1. Confirm the response exists in `Data`.
2. Confirm `Dashboard Data` updated.
3. Open the Cloudflare Worker URL and confirm the JSON contains the new counts.
4. If the Worker is current but the dashboard is not, inspect the browser console/network request.
5. Force refresh the page.

macOS Safari:

```text
Option + Command + R
```

### Dashboard shows 0 / 0 but Worker JSON is correct

Confirm the `API_URL` in `index.html` is the correct Cloudflare Worker URL.

### Baby photo does not update

Increment its `?v=` cache-busting parameter in `index.html`.

### Custom domain does not resolve

Run:

```bash
dig vote.courtney.biz CNAME
```

Confirm it points to:

```text
newtguppy.github.io.
```

### GitHub reports an invalid DNS configuration

Confirm GoDaddy contains:

```text
CNAME
Name: vote
Value: newtguppy.github.io
```

### API says Google Sheets access failed

Verify:

- Google Sheets API is enabled.
- The service account still exists.
- The spreadsheet is still shared with the service account as Viewer.
- `GOOGLE_SPREADSHEET_ID` is correct.
- `GOOGLE_SHEET_RANGE` is `Dashboard Data!A1:B4`.
- The private key stored in Cloudflare is valid.

## Security

The repository must never contain:

- Google service-account JSON files
- Google private keys
- Cloudflare secrets
- OAuth tokens
- passwords

The website itself is intentionally public because guests need to access it.

The Cloudflare Worker exposes only aggregate voting data and does not need to expose individual Google Form responses.

## Repository Visibility

The source repository may be public or private depending on the GitHub account plan.

Important considerations:

- GitHub Free supports Pages from public repositories.
- GitHub Pro and eligible paid organization plans support Pages from private repositories.
- A Pages website may still be publicly accessible even when its source repository is private.
- Making a repository private on an account that does not support Pages from private repositories will unpublish the Pages site.
- Never assume that making the repository private also makes the website private.

Before changing repository visibility, confirm the GitHub account plan supports GitHub Pages from private repositories.

## Deprecated / Removed Architecture

Google Apps Script is **not used** by the current solution.

An early prototype used Apps Script, but it was replaced after Google Advanced Protection blocked the required authorization flow.

Current production architecture:

```text
Google Form
  ->
Google Sheet
  ->
Google Sheets API
  ->
Cloudflare Worker
  ->
GitHub Pages
  ->
vote.courtney.biz
```

The old Apps Script project can remain deleted.

## Maintenance Summary

For normal operation, the components that must remain active are:

1. Google Form
2. Google response Sheet
3. `Dashboard Data` formulas
4. Google Sheets API
5. Google service account
6. Cloudflare Worker and secrets
7. GitHub repository and Pages deployment
8. GoDaddy CNAME record
9. `vote.courtney.biz`

---

Built for the Baby Courtney Splish Splash Gender Reveal.
