# Google Sheets Integration Setup

Form submissions are sent to a Google Sheet. Follow these steps to set it up.

## 1. Create the Google Sheet

- Go to [sheets.google.com](https://sheets.google.com) and create a new blank spreadsheet
- Name it something like "Client Intake Submissions"

## 2. Add the Apps Script

- In your new sheet, go to **Extensions > Apps Script**
- Delete any code already there
- Paste the following:

```javascript
function doPost(e) {
  var sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
  var data = JSON.parse(e.postData.contents);

  // Add headers on first submission
  if (sheet.getLastRow() === 0) {
    var headers = ["Timestamp"].concat(Object.keys(data));
    sheet.appendRow(headers);
  }

  // Build row matching header order
  var headers = sheet.getRange(1, 1, 1, sheet.getLastColumn()).getValues()[0];
  var row = headers.map(function(header) {
    if (header === "Timestamp") return new Date().toISOString();
    var val = data[header];
    if (Array.isArray(val)) return val.join(", ");
    return val || "";
  });

  sheet.appendRow(row);

  return ContentService
    .createTextOutput(JSON.stringify({ status: "ok" }))
    .setMimeType(ContentService.MimeType.JSON);
}
```

- Click **Save** (Ctrl+S)

## 3. Deploy as Web App

1. Click **Deploy > New deployment**
2. Click the gear icon next to "Select type" and choose **Web app**
3. Set:
   - **Description**: "Intake form handler" (optional)
   - **Execute as**: "Me"
   - **Who has access**: "Anyone"
4. Click **Deploy**
5. Authorise the app when prompted (click through the "unsafe" warning - it's your own script)
6. **Copy the Web app URL** - it looks like: `https://script.google.com/macros/s/ABCDEF.../exec`

## 4. Add the URL to the Form

Open `index.html` and find this line near the top of the `<script>` section:

```javascript
var GOOGLE_SHEETS_URL = ""; // <-- Paste your Apps Script web app URL here
```

Paste your URL between the quotes:

```javascript
var GOOGLE_SHEETS_URL = "https://script.google.com/macros/s/YOUR_SCRIPT_ID/exec";
```

## 5. Test It

- Open `index.html` in a browser
- Fill out and submit the form
- Check your Google Sheet - a new row should appear within a few seconds

## How It Works

- On submit, the form sends a JSON object to your Apps Script URL via `fetch()`
- The Apps Script receives it, parses it, and appends a row to your sheet
- The first submission auto-creates column headers from the field names
- Arrays (like the features checklist) are joined with commas
- A timestamp is added automatically
- The data is also logged to the browser console as a backup

## Updating the Script

If you add or remove form fields, the script handles it automatically - new fields get new columns on the next submission. You don't need to redeploy.

However, if you change the script code itself, you need to:
1. Go to Apps Script
2. Click **Deploy > Manage deployments**
3. Click the pencil icon on your deployment
4. Change **Version** to "New version"
5. Click **Deploy**
