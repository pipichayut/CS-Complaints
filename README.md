# Supersports Complaint Radar

A one-page dashboard that shows where customers are unhappy with the Supersports Virtual AI assistant. It reads the chat log straight from a published Google Sheet every time it opens, tags each customer message for complaint scenarios, and shows:

- **Complaint watch:** each scenario in the chosen time frame compared with the previous period of the same length, spike days, and a follow-up list of the riskiest messages
- **What to fix first:** 13 issues in priority order, with counts for the time frame
- **Time frame analysis:** summary figures, complaint scenarios with deep dives (what customers write vs the real issue), trends, mix over time, time of day, bot handoffs and friction requests

Everything runs in the browser. There is no server and no build step.

## Files

| File | What it is |
| --- | --- |
| `index.html` | The whole dashboard: page, styles, tagging rules and code |
| `.nojekyll` | Tells GitHub Pages to serve the files as they are |
| `README.md` | This guide |

## Publish on GitHub Pages

1. Create a new repository on GitHub, for example `complaint-radar`.
2. Upload `index.html`, `.nojekyll` and `README.md` to the main branch (Add file > Upload files).
3. Open **Settings > Pages**. Under **Build and deployment**, choose **Deploy from a branch**, branch `main`, folder `/ (root)`, then **Save**.
4. After a minute the site is live at `https://<your-user-name>.github.io/complaint-radar/`.

## Connect the Google Sheet

The page reads the sheet set in `CONFIG.DATA_URL` near the top of the script in `index.html`:

```js
const CONFIG = {
  DATA_URL: 'https://docs.google.com/spreadsheets/d/e/2PACX-.../pub?output=csv',
  DATE_ORDER: 'DMY',
  DEFAULT_RANGE: '30'
};
```

To use a different sheet:

1. In Google Sheets, open **File > Share > Publish to web**.
2. Choose the tab with the chat log and the format **Comma-separated values (.csv)**, then **Publish**.
3. Copy the link (it ends in `output=csv`) and paste it into `DATA_URL`.

The sheet needs these column headers in the first row. Extra columns are ignored.

| Column | Used for |
| --- | --- |
| `Timestamp (BKK)` | Date and hour of each message, for example `2026-04-16 00:23:02` |
| `Question` | The customer's message, which is tagged for complaints |
| `Session ID` | Groups messages into chats |
| `Tool Used` | Order-status lookups |
| `Agent Response` | Detects handoffs to the call center and order statuses |
| `Time Taken (s)` | Slow bot replies (over 15 seconds) |

If dates are written like `16/04/2026 0:23:02`, keep `DATE_ORDER: 'DMY'`. Use `'MDY'` for `04/16/2026`. Thai Buddhist years (2569) are converted automatically.

## How it updates

- Google refreshes a published sheet a few minutes after it changes. Reload the page or press **Refresh data** to pull the latest rows.
- The page saves a summary in the viewer's browser, so it opens instantly next time and still works if the sheet can't be reached.
- **Load a file instead** reads a CSV or XLSX export with the same columns. Choose **Replace current data** or **Add to current data**. Files are processed in the browser only and are not uploaded anywhere.

## Privacy

A sheet published to the web can be opened by anyone who has the link. A GitHub Pages site is public on most GitHub plans, even when the repository is private. The follow-up list shows customer messages, with emails, phone numbers, names after a title (คุณ, นาย, นาง, Mr, Ms) and house numbers hidden. Check that this fits your company's data policy before you publish, or keep the site private (for example with GitHub Enterprise private Pages, or by opening `index.html` from your own computer).

## How messages are tagged

Each message is matched against Thai and English keyword rules for 10 complaint scenarios: late or not received delivery, missing or wrong items, damaged products, slow refunds, orders cancelled after payment, promo codes, payment problems, OTP and login problems, staff unreachable, and an unhelpful chatbot. It is also matched for 3 friction requests: cancel, return or exchange, and asking for a human. A message also counts as a complaint when it uses strong negative wording.

- **Spike day:** a scenario is at least 1.5 times its 7-day average and 8 or more messages above it.
- **Complaint watch alert:** a scenario is at least 1.5 times the previous period and 8 or more messages higher, or has spike days on at least 1 in 10 days of the time frame.

Counts are keyword-based estimates. They are reliable for comparing scenarios and periods, not as exact totals.
