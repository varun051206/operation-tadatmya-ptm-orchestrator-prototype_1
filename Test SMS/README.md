# SRM MCET Dept of AIDS — PTM Orchestrator

## SMS Automation Prototype

A complete, mobile-first SMS automation prototype for the **Department of Artificial Intelligence & Data Science, SRM MCET**, designed to orchestrate and send official Parents-Teachers Meeting (PTM) invitation SMS messages to recipients listed in Google Sheets using the **Fast2SMS bulkV2 API** and **Google Apps Script**.

---

## 1. Project Folder Structure

```text
Test SMS/
│
├── index.html          # Mobile-first frontend UI (Vanilla HTML, CSS, JavaScript)
├── Code.gs             # Google Apps Script Web App backend logic
├── README.md           # Setup, deployment, and configuration guide
│
└── asset/
    └── Image.png       # SRM MCET Dept of AIDS PTM Orchestrator official logo
```

---

## 2. Technology Stack

* **Frontend**: Vanilla HTML5, Vanilla CSS, Vanilla JavaScript (System font stack, zero external font/CDN requests).
* **Backend**: Google Apps Script Web App (`Code.gs`).
* **Database / Recipient Store**: Google Sheets (`TD SMS Test`).
* **SMS Gateway**: Fast2SMS API (`https://www.fast2sms.com/dev/bulkV2`, Route `q`).
* **Hosting**: Vercel (static frontend hosting).

---

## 3. Google Sheet Setup

Spreadsheet: **`TD SMS Test`**

The spreadsheet uses two sheet tabs:

### Recipient Sheet: `Sheet1`
Contains the recipient records. Row 1 has the column headers:
```text
S.No | Name | Number | Status
```
The application reads all valid recipient numbers dynamically starting from row 2 until the last populated row.

### Log Sheet: `Msg`
Stores the permanent audit log of sent messages. Row 1 has the column headers:
```text
S.No | Time Stamp | Msg
```
*(Rows below are populated automatically with server-side timestamps each time an SMS request is submitted).*

---

## 4. Google Apps Script Setup & Configuration

1. In your Google Spreadsheet (`TD SMS Test`), go to **Extensions** → **Apps Script** (or open [script.google.com](https://script.google.com)).
2. Delete any boilerplate code in `Code.gs` and paste the complete content of [Code.gs](file:///c:/Users/CSE-DEPT/Desktop/Test%20SMS/Code.gs).
3. **Configure the Spreadsheet ID**:
   - Look at your Google Sheet's URL in the browser address bar:
     ```text
     https://docs.google.com/spreadsheets/d/1BxiMVs0XRA5nFMdKvBdBZjgmUUqptlbs74OgvE2upms/edit
     ```
   - Copy the ID string between `/d/` and `/edit` (in the example above: `1BxiMVs0XRA5nFMdKvBdBZjgmUUqptlbs74OgvE2upms`).
   - Open `Code.gs` and paste it into `SPREADSHEET_ID` near the top:
     ```javascript
     const SPREADSHEET_ID = "YOUR_COPIED_SPREADSHEET_ID";
     ```
4. Save the project (**Ctrl + S** or disk icon).

---

## 5. Deploying Apps Script as a Web App

1. In the Apps Script editor, click the blue **Deploy** button (top-right) → **New deployment**.
2. Click the gear icon (**Select type**) and choose **Web app**.
3. Fill in the deployment details:
   - **Description**: `PTM Orchestrator Web App`
   - **Execute as**: `Me (your email)`
   - **Who has access**: `Anyone` *(Crucial: allows the Vercel frontend to reach the API)*
4. Click **Deploy**.
5. Grant permissions if prompted by Google (click *Advanced* → *Go to PTM Orchestrator (unsafe)* → *Allow*).
6. Copy the generated **Web App URL**:
   ```text
   https://script.google.com/macros/s/AKfycbx.../exec
   ```

> [!TIP]
> Whenever you make code modifications to `Code.gs`, deploy a **New version** (Deploy → Manage deployments → Edit → New version → Deploy).

---

## 6. Connecting Web App to Frontend

1. Open [index.html](file:///c:/Users/CSE-DEPT/Desktop/Test%20SMS/index.html) in your code editor.
2. The deployed Web App URL is configured near line 590:
   ```javascript
   const APPS_SCRIPT_URL = "https://script.google.com/macros/s/AKfycbxWTUVZNVXOVhUuIDKe4ZcRQ770K2wQsF2ZxO65FbloT7Ip11ocDwQNMOOc_A5rOrqkiA/exec";
   ```
3. If you ever redeploy with a new version, update `APPS_SCRIPT_URL` with your new Web App URL.
4. Save `index.html`.

---

## 7. Vercel Deployment

The frontend is a 100% static project requiring **no build step**:

1. Log in to [Vercel](https://vercel.com).
2. Import the project folder `Test SMS/` (via GitHub repository or using the Vercel CLI `vercel`).
3. Build & Development Settings:
   - **Framework Preset**: `Other`
   - **Build Command**: *(None / Leave blank)*
   - **Output Directory**: *(None / Leave blank or `./`)*
4. Click **Deploy**.
5. Your application will be live on an HTTPS domain (e.g., `https://srm-ptm-orchestrator.vercel.app`).

---

## 8. How SMS Sending Works

```text
User enters message (up to 2500 chars)
                 ↓
      User clicks [SEND SMS]
                 ↓
Frontend validates input & recipient count
                 ↓
Frontend sends CORS-safe POST to Apps Script
                 ↓
Apps Script reads 'Sheet1' in 'TD SMS Test'
                 ↓
Validates 10-digit recipient phone numbers
                 ↓
Calls Fast2SMS bulkV2 (Route 'q', sms_details: '1')
                 ↓
Evaluates Fast2SMS API response:
  - If Accepted: Updates Sheet1 Status = 'Sent'
                 Logs record in 'Msg' sheet (S.No, Server Timestamp, Msg)
                 Frontend displays: "SMS request submitted successfully"
  - If Rejected: Updates Sheet1 Status = 'Failed'
                 Frontend displays: "SMS request was rejected"
```

### Key Distinctions & Security Rules
* **API Acceptance vs. Delivery**: The system accurately reports that the SMS request has been accepted by Fast2SMS. It does not falsely claim confirmed handset delivery unless real-time carrier delivery receipts are received.
* **Clean Status Values**: The `Status` column only records clean, concise values (`Sent` or `Failed`), never raw JSON or API keys.
* **Server-Side Security**: The Fast2SMS API key (`aBEmHsi...`) is isolated entirely in `Code.gs` and is **never** sent to the client browser or exposed in `index.html`.
