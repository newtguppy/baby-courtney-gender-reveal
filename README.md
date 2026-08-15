# Baby Courtney Splish Splash Gender Reveal

I use this project to run a live boy-or-girl prediction dashboard for the Baby Courtney gender reveal and pool party. Guests vote through a Google Form, and the dashboard displays the results in near real time.

## Production links

| Resource | URL |
|---|---|
| Dashboard | [https://vote.courtneyfam.com](https://vote.courtneyfam.com/) |
| Legacy dashboard URL | [https://vote.courtney.biz](https://vote.courtney.biz/) |
| Voting form | [https://forms.gle/dVeoDoYPM2qgviJa9](https://forms.gle/dVeoDoYPM2qgviJa9) |

`https://vote.courtney.biz` is a permanent legacy address. GoDaddy redirects it to `https://vote.courtneyfam.com` with an HTTP `301` response.

## System overview

I use the following request path:

```text
Guest
  -> Google Form
  -> Google Sheets
  -> Google Sheets API
  -> Cloudflare Worker
  -> GitHub Pages
  -> https://vote.courtneyfam.com
```

The dashboard:

- Displays live Team Boy and Team Girl totals
- Calculates vote percentages and the current leader
- Refreshes automatically every three seconds
- Provides a QR code and direct link to the voting form
- Supports desktop, television, and mobile displays
- Shows a connection warning when the live API is unavailable
- Uses HTTPS on the production domain

## Components

| Component | Purpose |
|---|---|
| Google Forms | Collects guest predictions |
| Google Sheets | Stores responses and calculates aggregate totals |
| Google Sheets API | Provides aggregate values to the Cloudflare Worker |
| Google Cloud service account | Authenticates read-only spreadsheet requests |
| Cloudflare Worker | Returns public aggregate vote data as JSON |
| GitHub Pages | Hosts the static dashboard |
| GoDaddy DNS | Manages the production CNAME and legacy redirect |

## Repository

Repository name:

```text
baby-courtney-gender-reveal
```

Required files:

```text
baby-courtney-gender-reveal/
â”œâ”€â”€ CNAME
â”œâ”€â”€ README.md
â”œâ”€â”€ index.html
â”œâ”€â”€ dad-baby.JPEG
â”œâ”€â”€ mom-baby.jpg
â””â”€â”€ vote-qr.png
```

### `CNAME`

The file must contain one line:

```text
vote.courtneyfam.com
```

GitHub Pages supports one custom domain in this file. I manage the old `vote.courtney.biz` address through GoDaddy forwarding, not through a second CNAME entry.

### `index.html`

This file contains the complete dashboard application, including:

- Responsive layout and visual styling
- Live vote API integration
- Three-second polling
- Vote count and percentage calculations
- Current leader logic
- Voting form controls
- Connection-error handling

### Image files

| File | Purpose |
|---|---|
| `dad-baby.JPEG` | Dad's baby photo |
| `mom-baby.jpg` | Mom's baby photo |
| `vote-qr.png` | QR code for the Google Form |

File names and capitalization must remain exact because GitHub Pages asset paths are case-sensitive.

## Google Form

Form name:

```text
Baby Courtney Gender Prediction
```

Question:

```text
What's your prediction?
```

Choices:

```text
Boy
Girl
```

Public voting URL:

```text
https://forms.gle/dVeoDoYPM2qgviJa9
```

Both the dashboard button and `vote-qr.png` use this URL.

## Google Sheets

The response workbook contains two required sheets.

### `Data`

Google Forms owns this sheet. I do not rename its columns or change its structure.

| Column | Value |
|---|---|
| Timestamp | Form-generated submission time |
| What's your prediction? | `Boy` or `Girl` |

### `Dashboard Data`

This sheet exposes only aggregate values to the dashboard.

| Metric | Formula |
|---|---|
| Boy | `=COUNTIF(Data!B:B,"Boy*")` |
| Girl | `=COUNTIF(Data!B:B,"Girl*")` |
| Total | `=SUM(B2:B3)` |

The wildcard allows the count formulas to continue working if I add text or emoji after `Boy` or `Girl` in the form choices.

## Google Cloud

### Google Sheets API

The Google Sheets API must remain enabled in the Google Cloud project.

### Service account

Service account name:

```text
gender-reveal-dashboard
```

I share the response spreadsheet directly with this service account using Viewer access. The account requires read-only access to the aggregate vote totals.

### Service account key

The Cloudflare Worker uses a JSON service-account key for authentication. I treat the key as a password and never commit it to GitHub.

If the key is exposed:

1. Revoke or delete it in Google Cloud.
2. Create a replacement key.
3. Update the corresponding Cloudflare secret.

## Cloudflare Worker

Worker name:

```text
baby-courtney-vote-api
```

The Worker authenticates to Google, reads `Dashboard Data!A1:B4`, and returns aggregate values only.

Example response:

```json
{
  "boy": 4,
  "girl": 3,
  "total": 7,
  "updatedAt": "2026-08-22T..."
}
```

### Worker configuration

| Variable | Type | Purpose |
|---|---|---|
| `GOOGLE_CLIENT_EMAIL` | Secret | Service account email |
| `GOOGLE_PRIVATE_KEY` | Secret | Service account private key |
| `GOOGLE_SPREADSHEET_ID` | Secret | Response spreadsheet ID |
| `GOOGLE_SHEET_RANGE` | Text | `Dashboard Data!A1:B4` |

I do not store secret values in this repository.

## GitHub Pages

I publish the dashboard with these settings:

| Setting | Value |
|---|---|
| Source | Deploy from a branch |
| Branch | `main` |
| Folder | `/ (root)` |
| Custom domain | `vote.courtneyfam.com` |
| HTTPS | Enforced |

Commits to `main` publish automatically.

## Domain configuration

### Production domain

In the DNS zone for `courtneyfam.com`, I use this record:

| Type | Name | Value |
|---|---|---|
| CNAME | `vote` | `newtguppy.github.io` |

I verify the record from macOS with:

```bash
dig vote.courtneyfam.com CNAME +short
```

Expected result:

```text
newtguppy.github.io.
```

### Legacy redirect

In GoDaddy forwarding for `courtney.biz`, I use this configuration:

| Setting | Value |
|---|---|
| Subdomain | `vote` |
| Destination | `https://vote.courtneyfam.com` |
| Type | Permanent (`301`) |
| Forwarding | Forward only |

I do not point `vote.courtney.biz` directly to GitHub Pages. GitHub Pages recognizes `vote.courtneyfam.com` as the repository's only custom domain.

GoDaddy automatically provisions HTTPS for the forwarding address. Activation can take a few hours, and global DNS propagation can take up to 48 hours.

I verify the redirect with:

```bash
curl -IL https://vote.courtney.biz
```

The response should contain a redirect to the production address followed by a successful response:

```text
HTTP/2 301
location: https://vote.courtneyfam.com

HTTP/2 200
```

## Dashboard behavior

### Before the first vote

The dashboard displays:

```text
Voting Status
Voting Is Open
```

### After voting begins

The dashboard displays one of these states:

```text
Current Leader
Team Boy +N
```

```text
Current Leader
Team Girl +N
```

```text
It's a Tie!
```

### Live refresh

The dashboard polls the Cloudflare Worker every `3000 ms`. New votes should appear without a manual refresh.

### Connection failure

If the API is unavailable, the dashboard displays:

```text
Connection lost. Reconnecting...
```

The dashboard continues retrying automatically.

## Responsive behavior

### Television and desktop

The full layout displays:

- Both baby photos
- Team Boy and Team Girl panels
- Vote counts and percentages
- Progress bars
- QR code and voting button
- Total predictions
- Voting status or current leader

### Mobile

The layout stacks vertically. The QR code may be hidden because a user cannot conveniently scan a QR code displayed on the same phone. The direct voting button remains available.

## Updating content

### Replace a baby photo

1. Upload the replacement image to GitHub.
2. Preserve the expected file name.
3. Increment the image's cache-busting version in `index.html`.
4. Commit the change to `main`.

Example:

```html
<img src="dad-baby.JPEG?v=5" alt="Dad as a baby">
```

### Change the voting form URL

1. Update every voting-form URL in `index.html`.
2. Generate a new QR code.
3. Replace `vote-qr.png`.
4. Increment the QR code's cache-busting version in `index.html`.
5. Commit the changes to `main`.
6. Test the QR code and direct voting button.

## Resetting test votes

Before the party:

1. Open the Google Sheet.
2. Open the `Data` sheet.
3. Delete test-response rows beginning with row 2.
4. Preserve the header row.
5. Preserve the `Data` sheet.
6. Preserve the formulas in `Dashboard Data`.

The dashboard should automatically return to:

```text
Team Boy: 0
Team Girl: 0
Predictions Cast: 0
Voting Is Open
```

## Party-day checklist

- Confirm `https://vote.courtneyfam.com` loads over HTTPS.
- Confirm `https://vote.courtney.biz` redirects to the production address.
- Confirm the Google Form accepts responses.
- Confirm the Cloudflare Worker returns current JSON.
- Submit one Boy test vote and confirm it appears.
- Submit one Girl test vote and confirm it appears.
- Delete all test votes and confirm the dashboard returns to `0 / 0`.
- Confirm the QR code scans from the television.
- Confirm the direct voting button works.
- Test the dashboard from a phone.
- Connect the laptop to the television through HDMI.
- Use browser full-screen mode at 100% zoom.
- Disable sleep, screen locking, display dimming, and notifications.
- Connect the laptop to power.
- Confirm Wi-Fi coverage at the display and guest area.
- Keep a phone hotspot available as a backup.

## Troubleshooting

### The legacy address returns a GitHub 404

Confirm that GoDaddy forwards `vote.courtney.biz` to `https://vote.courtneyfam.com`. The old address must not use a CNAME that points directly to `newtguppy.github.io`.

If the forwarding configuration is correct, allow a few hours for HTTPS activation and up to 48 hours for global propagation.

### The production domain does not resolve

Run:

```bash
dig vote.courtneyfam.com CNAME +short
```

Confirm that the result is:

```text
newtguppy.github.io.
```

Also confirm that the GitHub Pages custom domain is `vote.courtneyfam.com` and that the repository's `CNAME` file contains the same value.

### GitHub reports invalid DNS configuration

Confirm that the `courtneyfam.com` DNS zone contains:

```text
Type: CNAME
Name: vote
Value: newtguppy.github.io
```

### A vote appears in Google Sheets but not on the dashboard

Check the system in this order:

1. Confirm that the response exists in `Data`.
2. Confirm that `Dashboard Data` updated.
3. Open the Cloudflare Worker URL and inspect its JSON.
4. If the Worker is current, inspect the dashboard's browser console and network requests.
5. Force-refresh the page.

In Safari on macOS, use:

```text
Option + Command + R
```

### The dashboard displays `0 / 0`, but the Worker is correct

Confirm that `API_URL` in `index.html` contains the correct Cloudflare Worker URL.

### A baby photo does not update

Increment the image's `?v=` cache-busting parameter in `index.html`, commit the change, and force-refresh the page.

### The Worker cannot access Google Sheets

Confirm that:

- The Google Sheets API is enabled.
- The service account exists.
- The spreadsheet is shared with the service account as Viewer.
- `GOOGLE_SPREADSHEET_ID` is correct.
- `GOOGLE_SHEET_RANGE` is `Dashboard Data!A1:B4`.
- The Cloudflare private-key secret is valid.

## Security

I never store these values in the repository:

- Google service-account JSON files
- Google private keys
- Cloudflare secrets
- OAuth tokens
- Passwords

The dashboard is intentionally public. The Cloudflare Worker exposes aggregate voting data only and does not expose individual responses.

## Repository visibility

The repository may be public or private if the GitHub account plan supports GitHub Pages for that visibility level. The website remains publicly accessible even if the source repository is private.

Before changing repository visibility, I confirm that the GitHub account plan supports GitHub Pages from private repositories. I do not assume that a private repository creates a private website.

## Current architecture

Google Apps Script is not part of the production solution. I replaced the early Apps Script prototype after Google Advanced Protection blocked the required authorization flow.

The active architecture is:

```text
Google Form
  -> Google Sheet
  -> Google Sheets API
  -> Cloudflare Worker
  -> GitHub Pages
  -> https://vote.courtneyfam.com
```

## Maintenance requirements

I keep these components active:

1. Google Form
2. Google response Sheet
3. `Dashboard Data` formulas
4. Google Sheets API
5. Google service account
6. Cloudflare Worker and secrets
7. GitHub repository and Pages deployment
8. `vote.courtneyfam.com` CNAME
9. `vote.courtney.biz` permanent redirect
