# Conference Check-In System

## 1. Overview

The Visionnaire Leadership Conference (VLC) Check-In System enables attendees to gain entry using personalized QR codes.

- Before the event, each attendee is emailed a unique QR code
- At the venue, volunteers scan QR codes with their phone camera
- The system validates against a Google Sheet database, updates their check-in status, and logs the time
- A GitHub Pages dashboard displays real-time check-ins and prevents duplicate entries

## 2. Components

### 2.1 Google Sheet (Attendee Database)

Central source of truth.

**Columns used:**

| Column | Field | Example |
|--------|--------|---------|
| A | Name | John Doe |
| B | Email | john@example.com |
| C | Status | Sent / Error |
| D | Check-in Status | ✅ Checked in / blank |
| E | Check-in Time | 10:15:22 AM |
| F | Pass Type | VIP / Regular |
| G | Embed (QR content) | john-doe-vip-unique123 |

### 2.2 Google Apps Script (Backend)

#### QR Generator Script
- Creates QR codes using the Embed field
- Sends via Gmail with plain text + HTML body
- Marks email as "Sent" once delivered

#### Check-In Script
- Exposes a web endpoint
- Validates scanned QR against Embed column
- Updates Check-in Status + Time
- Prevents duplicates
- Returns JSON response

**Example JSON Response:**
```json
{
  "success": true,
  "message": "✅ John Doe (VIP) checked in",
  "time": "2025-09-06T10:15:22.000Z"
}
```

### 2.3 GitHub Pages (Frontend)

- Hosted static site
- Uses html5-qrcode library
- On scan:
  - Sends POST request to Apps Script endpoint
  - Updates UI with check-in confirmation
  - Adds entry to live log table

**Important Change:**
Switched from URLSearchParams to JSON requests.

```javascript
fetch("https://script.google.com/macros/s/<DEPLOYMENT_ID>/exec", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ name })
});
```

## 3. QR Code Generation

**Function:** `generateQRImagesToDrive`

- Iterates over attendee list
- Uses [QuickChart.io QR API](https://quickchart.io/)
- Sends QR to attendee's email
- Updates Column C (Status) with "Sent"

**Key Variables:**
```javascript
const nameCol = 0;   // A: Name
const emailCol = 1;  // B: Email
const statusCol = 2; // C: Email sent status
const embedCol = 6;  // G: Embed (QR content)
```

## 4. Check-In Backend

**Function:** `doPost(e)`

- Validates scanned QR
- If valid → marks attendee as checked in
- If already checked in → returns "already checked in"
- If invalid → returns error

**Deployment:**
- Execute as: Me
- Access: Anyone
- URL format: `https://script.google.com/macros/s/<DEPLOYMENT_ID>/exec`

## 5. Frontend

**File:** `index.html`

- Loads camera scanner
- Displays confirmation or error
- Updates table log dynamically

**Sample Flow:**
1. Volunteer scans QR
2. Browser sends request to Apps Script
3. JSON response is displayed on screen
4. Log table updates in real-time

## 6. Workflow

### Prepare Attendee Sheet
1. Consolidate names & emails
2. Add Embed column: `=Name & " - " & Pass Type`
3. Copy & paste values (not formulas)

### Run QR Generator Script
1. Emails QR codes
2. Updates Status column

### Deploy Check-In Backend
1. Deploy as web app
2. Copy web app URL

### Configure GitHub Pages Frontend
1. Update index.html with backend URL
2. Push changes to GitHub

### Event Day
1. Volunteers open GitHub Pages link
2. Scan attendee QR codes
3. Sheet + dashboard update live

## 7. Error Handling

| Error Type | Message Example |
|------------|-----------------|
| Invalid QR | ⚠ Invalid or empty QR code |
| Not Found | ❌ Name not found |
| Already Checked In | John Doe already checked in |
| Network Issue | ⚠ Network error |
| Email Failure | Error logged in Column C |

## 8. Key Benefits

✅ Works on any smartphone browser  
✅ Supports multiple scanners at once  
✅ Live sheet prevents duplicate check-ins  
✅ Easy to reuse for future events  

## 9. Future Improvements

- Admin dashboard with live stats (total attendees, % checked in)
- Manual override for failed scans
- Pass type filter (VIP / Regular)
- Protect Google Sheet from accidental edits
