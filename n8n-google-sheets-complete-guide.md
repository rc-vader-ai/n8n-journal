# Complete Guide: n8n + Google Sheets OAuth Setup with Data Access Scopes

## Understanding Data Access Scopes (Crucial!)

**What are scopes?** Scopes are permission "keys" that define exactly what your n8n workflow can do with Google Sheets. Think of them like keycards in a building—you only grant access to the rooms you actually need.

### Which Scopes Do You Need?

| Operation | Scope Required | Scope URL | Read-Only? |
|-----------|---------|-----------|---------|
| **Read data** (fetch rows, get values) | `spreadsheets` OR `spreadsheets.readonly` | `https://www.googleapis.com/auth/spreadsheets.readonly` | ✅ Yes |
| **Write data** (add/update/delete rows, cells) | `spreadsheets` (MUST HAVE) | `https://www.googleapis.com/auth/spreadsheets` | ❌ No |
| **Create spreadsheets** | `spreadsheets` + `drive.file` | `https://www.googleapis.com/auth/spreadsheets` `https://www.googleapis.com/auth/drive.file` | ❌ No |
| **Browse/manage Google Drive files** | `drive.file` | `https://www.googleapis.com/auth/drive.file` | ❌ No |

**TL;DR for n8n workflows with read/write/update:**
- **Minimum required:** `https://www.googleapis.com/auth/spreadsheets` (MUST add this to GCP)
- **Recommended:** Also add `https://www.googleapis.com/auth/drive.file` (for file browsing in Drive)
- **Optional:** Only if read-only: use `https://www.googleapis.com/auth/spreadsheets.readonly`

---

## Complete Step-by-Step Setup

### Phase 1: Google Cloud Console Setup

#### Step 1: Create a Google Cloud Project
1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Click the **Project Selector** (top-left, next to "Google Cloud")
3. Click **NEW PROJECT**
4. Enter Project Name: `n8n-sheets` (or your preferred name)
5. Click **CREATE** and wait 1-2 minutes for the project to initialize
6. Select the project from the dropdown

#### Step 2: Enable Required APIs
1. Go to **APIs & Services → Library**
2. Search for and enable these APIs (click each, then click **ENABLE**):
   - **Google Sheets API**
   - **Google Drive API** (important for file operations and scopes)
3. Verify both show "API enabled" with a checkmark

#### Step 3: Configure OAuth Consent Screen
The consent screen is where Google shows what permissions n8n is requesting. This **DIRECTLY AFFECTS** what scopes are available.

1. Go to **APIs & Services → OAuth consent screen**
2. **User Type Selection:**
   - Choose **External** (allows any Google account; better for most n8n setups)
   - Click **CREATE**
3. **Fill in the form:**
   - **App name:** `n8n Automation` (or your workflow name)
   - **User support email:** Your email
   - **Developer contact:** Your email
   - Click **SAVE AND CONTINUE**

4. **Add Scopes (THIS IS CRITICAL FOR READ/WRITE):**
   - You're now on the **Scopes** page
   - Click **ADD OR REMOVE SCOPES**
   - In the right panel, search for `Google Sheets API`
   - **Check these boxes:**
     - ✅ `.../auth/spreadsheets` (See, edit, create, and delete all your Google Sheets spreadsheets)
     - ✅ `.../auth/drive.file` (Create, edit, and delete files in Google Drive)
   - Click **UPDATE**
   - Back on Scopes page, click **SAVE AND CONTINUE**

5. **Add Test Users (Essential for External apps in testing):**
   - Click **ADD USERS**
   - Enter the email of the Google account you'll use for n8n
   - Click **ADD**
   - Click **SAVE AND CONTINUE**

**Why these scopes matter:**
- `spreadsheets`: Allows reading, writing, creating, updating, and deleting Google Sheets data
- `drive.file`: Allows n8n to browse and manage files in Google Drive
- Without these, n8n cannot perform write operations (403 access denied error)

#### Step 4: Create OAuth Client Credentials
1. Go to **APIs & Services → Credentials**
2. Click **+ CREATE CREDENTIALS**
3. Select **OAuth 2.0 Client ID**
4. **Application type:** Choose **Web application**
5. **Name:** `n8n-oauth-client` (or your preference)
6. **Authorized JavaScript origins:** Leave empty
7. **Authorized redirect URIs:** 
   - Click **ADD URI**
   - You'll add the n8n redirect URI in the next section
   - For now, leave this blank (we'll fill it from n8n)
8. Click **CREATE**
9. **Copy and save these (you'll need them soon):**
   - **Client ID**
   - **Client Secret**
   - ⚠️ Keep these secret—never commit to version control!

---

### Phase 2: Configure n8n Credentials

#### Step 5: Get Your n8n Redirect URI
1. Open your n8n instance in a browser tab
   - Cloud: `https://n8n.cloud/` (if using n8n Cloud)
   - Local Docker: `http://localhost:5678`
   - Self-hosted: Your domain URL
2. Click **Credentials** (left sidebar)
3. Click **+ New Credential**
4. Search for and select **Google Sheets OAuth2** (or **Google OAuth2** for generic access)
5. You should see a field labeled **OAuth Redirect URL** or similar
6. **Copy this URL exactly** (e.g., `https://your-n8n-url.com/rest/oauth2-credential/callback` or `http://localhost:5678/rest/oauth2-credential/callback`)

#### Step 6: Add Redirect URI to Google Cloud Console
1. Return to **Google Cloud Console → APIs & Services → Credentials**
2. Click on your OAuth client (the one you just created)
3. Under **Authorized redirect URIs**, click **ADD URI**
4. Paste the n8n redirect URL you copied in Step 5
5. Click **SAVE**

#### Step 7: Create n8n Google Credential
Back in n8n (the New Credential window from Step 5):

1. **Credential name:** `Google Sheets - Main` (or identifiable name)
2. **Client ID:** Paste the Client ID from Google Cloud Console
3. **Client Secret:** Paste the Client Secret from Google Cloud Console
4. **Click "Connect"** or **"Sign in with Google"**
5. **A Google authorization popup appears:**
   - You'll see a consent screen showing:
     - "n8n wants access to:"
     - Permission list (should show Sheets and Drive permissions)
   - Click **Allow** or **Continue** to grant permissions
   - Google redirects back to n8n
6. **Save the credential** in n8n

**What just happened?** n8n requested the scopes you defined in GCP (Step 3 scopes), and Google issued an OAuth token containing those permissions. This token is what allows write operations.

---

### Phase 3: Build Your Workflow and Test

#### Step 8: Create a Test Workflow
1. In n8n, create a new workflow or open existing one
2. Add a **Google Sheets** node
3. **Set the Operation:**
   - For **READ** only: `Read` or `Fetch (Get All)` 
   - For **WRITE/UPDATE**: `Append Row` or `Append or Update Row`
   - For **CREATE**: `Create a Spreadsheet`
4. **Select your credential:** Pick the credential you just created (`Google Sheets - Main`)
5. **Select spreadsheet:** Choose from dropdown (n8n will list your sheets)
6. **Configure fields** based on operation type

#### Step 9: Test Each Operation Type

**Test 1: Read Operation**
```
Operation: Fetch (Get All)
Spreadsheet: Your test sheet
Sheet name: Sheet1
Execute the node
```
✅ If successful: You have read permission working

**Test 2: Write Operation**
```
Operation: Append Row
Spreadsheet: Your test sheet
Sheet name: Sheet1
Columns: name, email, date
Add test data
Execute the node
```
✅ If successful: You have write permission working
❌ If you get `403 access_denied` or `insufficient authentication scopes`: See troubleshooting below

**Test 3: Update Operation**
```
Operation: Append or Update Row
Spreadsheet: Your test sheet
Sheet name: Sheet1
Key column: email (or ID)
Update condition: email == "test@example.com"
New values: Add your data
Execute the node
```
✅ If successful: Full read/write/update working

---

## Troubleshooting: Common Errors & Fixes

### Error: "403 access_denied" or "Insufficient Authentication Scopes"

**Root cause:** The scopes in GCP weren't properly added or the token was issued before scopes were added.

**Fix:**
1. Verify scopes are in GCP:
   - **APIs & Services → OAuth consent screen → Scopes**
   - Confirm `spreadsheets` and `drive.file` are listed ✓
2. **Re-authorize in n8n:**
   - Delete the existing credential
   - Create a new credential with same client ID/secret
   - Click **Connect** again
   - This forces a fresh token request with current scopes
3. If still failing, confirm the Google account has **Editor access** to the spreadsheet

### Error: "The caller does not have permission"

**Root cause:** The Google account doesn't have edit access to the spreadsheet.

**Fix:**
1. Open the Google Sheet in Google Sheets UI
2. Click **Share**
3. Confirm the email of the Google account you used in n8n is listed with **Editor** access
4. If not, add it with Editor permission
5. Return to n8n and retry the operation

### Error: "Redirect URI mismatch"

**Root cause:** The redirect URI in n8n doesn't match GCP credentials.

**Fix:**
1. Copy n8n redirect URI again (from n8n credentials page)
2. Check GCP: **Credentials → Your OAuth client → Authorized redirect URIs**
3. Ensure the n8n URI is listed exactly (including `https://` or `http://`)
4. Save in GCP, then retry in n8n

### Error: "Invalid Client ID or Secret"

**Root cause:** Typo or copied whitespace in Client ID/Secret.

**Fix:**
1. Go back to **Google Cloud Console → Credentials**
2. Re-copy both Client ID and Client Secret (triple-check for spaces)
3. Delete and recreate the n8n credential
4. Paste values again carefully
5. Retry connection

---

## Key Takeaways: Are Scopes Necessary?

**YES.** Data access scopes in GCP are **absolutely necessary** if you want to:
- ✅ Write data to Google Sheets (add rows, update cells)
- ✅ Create new spreadsheets
- ✅ Delete data
- ✅ Access files in Google Drive

**What if you only read?**
- You can use `spreadsheets.readonly` scope (more secure)
- But if you ever need to write, you must upgrade to `spreadsheets`

**Why doesn't it work without scopes?**
- Without scopes in GCP, the OAuth token issued to n8n doesn't include write permissions
- Google sees the request and returns `403 access_denied`
- Scopes are how you tell Google "this app is allowed to do this"

---

## Summary Checklist

- [ ] Created Google Cloud project
- [ ] Enabled Google Sheets API
- [ ] Enabled Google Drive API
- [ ] Set OAuth consent screen to **External**
- [ ] Added `spreadsheets` scope in GCP
- [ ] Added `drive.file` scope in GCP
- [ ] Added your email as **Test User** in GCP
- [ ] Created OAuth 2.0 client credentials in GCP
- [ ] Copied Client ID and Client Secret
- [ ] Got n8n redirect URI
- [ ] Added n8n redirect URI to GCP authorized URIs
- [ ] Created credential in n8n with Client ID/Secret
- [ ] Clicked **Connect** and authorized with Google
- [ ] Tested **Read** operation ✓
- [ ] Tested **Write** operation ✓
- [ ] Tested **Update** operation ✓

**Once all items are checked, your n8n ↔ Google Sheets integration is production-ready!**

---

## Reference: n8n Google Sheets Operations

| Operation | What It Does | Scope Needed |
|-----------|------------|---------|
| Fetch (Get All) | Reads all rows from a sheet | `spreadsheets.readonly` or `spreadsheets` |
| Read Rows (Paginated) | Reads rows with pagination | `spreadsheets.readonly` or `spreadsheets` |
| Create a Spreadsheet | Creates a new Google Sheet | `spreadsheets` + `drive.file` |
| Append Row | Adds a new row to a sheet | `spreadsheets` |
| Append or Update Row | Adds a row or updates if exists | `spreadsheets` |
| Update Rows | Updates specific rows | `spreadsheets` |
| Delete Row | Removes rows from a sheet | `spreadsheets` |
| Get Columns | Lists columns in a sheet | `spreadsheets.readonly` or `spreadsheets` |

All write/update/delete operations **require** the full `spreadsheets` scope.

---

## Additional Resources

- [n8n Google OAuth Documentation](https://docs.n8n.io/integrations/builtin/credentials/google/oauth-single-service/)
- [Google API Scopes Reference](https://developers.google.com/identity/protocols/oauth2/scopes)
- [Google Sheets API Scopes](https://developers.google.com/workspace/sheets/api/scopes)
