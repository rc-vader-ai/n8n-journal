# DeepSeek \& Google Sheets in n8n: Fixing the OAuth Scopes

Swapped OpenAI for DeepSeek, hit a 403 with Google Sheets, fixed it by tightening up scopes, and turned the whole thing into a reusable guide you can drop into any n8n stack.

## Why This Test Run Mattered

This wasn’t “yet another hello world workflow.” The goal was to sanity‑check the official **“Get Started with Google Sheets in n8n”** template, swap the LLM from OpenAI Chat to **DeepSeek Chat**, and prove that the whole thing still works end‑to‑end: input, reasoning, and Sheets read/write.

Along the way, Google slapped the workflow with `access_denied / insufficient authentication scopes`. This is exactly the kind of silent landmine that will bite you in production if you just copy‑paste credentials and move on.

## The Stack You Actually Ran

* **n8n Workflow**: Based on the official [Get Started with Google Sheets](https://n8n.io/workflows/7156-get-started-with-google-sheets-in-n8n) template.
* **Google Sheets Node**: Using OAuth2 with a custom Google Cloud project and consent screen.
* **DeepSeek Chat**: Wired in as the LLM instead of OpenAI Chat, using n8n’s DeepSeek Chat Model integration (compatible with OpenAI-style API).

**The Idea:**

1. n8n pulls rows from a Sheet.
2. DeepSeek processes and enriches them.
3. n8n writes results back to the same Sheet.

## Where It Broke: Google Sheets 403

The failure was clean and annoying:

* **The Error**: The Google Sheets node refused to write, returning `403 / access_denied / insufficient authentication scopes`, while read operations might still behave or partially work.
* **The Confusion**: Credentials looked “correct.” The OAuth client existed, the redirect URI matched, and the Sheets/Drive APIs were enabled in the Cloud Console.

**Root Cause:**
The **OAuth consent screen** didn’t include the right data access scopes. Google issued a token that simply wasn’t allowed to perform *write* operations on Sheets. In other words, the app was authenticated (logged in) but not authorized (allowed to do the job).

## The Fix: A Scopes-First Approach

Instead of papering over the problem, the solution is to lock in the correct permissions explicitly. Here is the minimal scope set that actually works for read/write Sheets automation in n8n.

### 1. Required Scopes

Add these URLs to your OAuth Consent Screen in Google Cloud Console:

* **Required (Read/Write):**

```text
https://www.googleapis.com/auth/spreadsheets
```

* **Strongly Recommended (File Selection):**

```text
https://www.googleapis.com/auth/drive.file
```

*Required if you want to use the file selector (Drive) in n8n nodes.*
* **Optional (Read-Only):**

```text
https://www.googleapis.com/auth/spreadsheets.readonly
```

*Only use this if you truly never intend to write data.*


### 2. Implementation Steps

1. **Create Project**: Set up a Google Cloud project and enable the **Google Sheets API** and **Google Drive API**.
2. **Configure Scopes**: In the **OAuth consent screen** setup, explicitly add the scopes listed above. Don't forget to add your email as a "Test User" if the app is in Testing mode.
3. **Setup n8n**:
    * Add your **n8n redirect URI** to the OAuth client in Google Cloud.
    * Create a **Google Sheets OAuth2** credential in n8n.
    * **Re-authorize**: If you updated scopes on an existing project, you *must* reconnect the account in n8n to force Google to issue a new token with the updated permissions.

### 3. Troubleshooting

If you still see `403 / insufficient authentication scopes`:

* **Re-check Scopes**: Go back to the Google Cloud Console and ensure the scopes are saved on the consent screen.
* **Re-authorize**: Delete the credential in n8n and recreate it to ensure you aren't using a cached, limited token.
* **Check Access**: Confirm the Google account you are logging in with actually has **Editor** access to the specific Spreadsheet you are trying to modify.


## Why This Is Worth Posting

For anyone automating with n8n, this kind of bug is the difference between "demo ready" and "actually shippable."

* It proves **DeepSeek** is a drop-in alternative to OpenAI for this template, provided the API endpoint and key are configured correctly.
* It turns a vague "Google says no" error into a concrete fix: **fix your scopes, then reissue the token.**

By documenting this, you avoid wasting an afternoon on authentication trivia and get a repeatable recipe for the identity layer of your stack.
<span style="display:none">[^1][^2]</span>

<div align="center">⁂</div>

[^1]: https://github.com/rc-vader-ai/n8n-journal/blob/main/n8n-google-sheets-complete-guide.md

[^2]: https://n8npro.in/integrations/n8n-google-sheets-understanding-required-api-permissions/

