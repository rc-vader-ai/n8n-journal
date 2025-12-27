<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# Act as experience writer, you task to write a journal for me I tested on "Get Started with Google Sheets in n8n". I repalced OpenAI Chat to Deepseek Chat. And I hit an issue on Google Sheet on access denied. I fix it by Add Scopes to OAuth Consent Screen. I create a "n8n-google-sheets-complete-guide" to have a step by step on the Google OAuth with scope to address this case.  This journal will post on github and linked to Linkedin. Below is the written style:

"Stop padding sentences with empty filler phrases.
Ban obvious Al vocabulary immediately.
Vary your pacing. Don't write like a metronome.
Kill the meta-commentary; just say the thing.
Write as you speak, using T and "you.
Delete 'In conclusion."
Use formatting like salt; don't look like a template.
Take a stance. Stop hedging.
Ditch the overused 'Rule of Three.'
Be concrete. Specific writing sounds human."
Tone: Fun but professional.

Swapped OpenAI for DeepSeek, hit a 403 with Google Sheets, fixed it by tightening up scopes, and turned the whole thing into a reusable guide you can drop into any n8n stack.[^1_1][^1_2][^1_3]

## Why This Test Run Mattered

This wasn’t “yet another hello world workflow.”
The goal was to sanity‑check the official **“Get Started with Google Sheets in n8n”** template, swap the LLM from OpenAI Chat to **DeepSeek Chat**, and prove that the whole thing still works end‑to‑end: input, reasoning, Sheets read/write.[^1_2][^1_3]

Along the way, Google slapped the workflow with **access_denied / insufficient authentication scopes**, which is exactly the kind of silent landmine that will bite you in production if you just copy‑paste credentials and move on.[^1_4][^1_1]

## The Stack You Actually Ran

- n8n workflow based on the official **Get Started with Google Sheets** template.[^1_2]
- Google Sheets node using OAuth2 with a custom Google Cloud project and consent screen.[^1_5][^1_1]
- DeepSeek Chat wired in as the LLM instead of OpenAI Chat, using n8n’s DeepSeek Chat Model integration / compatible OpenAI-style API.[^1_6][^1_7][^1_3]

The idea:

- n8n pulls rows from a Sheet,
- DeepSeek processes / enriches them,
- n8n writes results back to the same Sheet.[^1_3][^1_5]


## Where It Broke: Google Sheets 403

The failure was clean and annoying:

- Google Sheets node refused to write, returning **403 / access_denied / insufficient authentication scopes** while reads might still behave or partially work.[^1_8][^1_1][^1_4]
- Credentials looked “correct”: OAuth client existed, redirect URI matched, Sheets/Drive APIs were enabled.[^1_9][^1_10][^1_1]

Root cause:
The **OAuth consent screen didn’t include the right data access scopes**, so Google issued a token that simply wasn’t allowed to perform write operations on Sheets.[^1_1][^1_8]

In other words: the app was authenticated, but not authorized to do what the node was asking it to do.

## What You Built: The Scopes-First Google OAuth Guide

Instead of papering over the problem, you wrote a focused guide: **“n8n-google-sheets-complete-guide”**.[^1_1]
The spine of that guide is simple:

- Explain **what scopes are** in plain terms: permission keycards that tell Google exactly what this n8n workflow is allowed to touch.[^1_1]
- Spell out the minimal **scope set that actually works** for read/write Sheets automation from n8n.[^1_11][^1_1]

Concretely, the guide locks in:

- For read/write/update Sheets from n8n:
    - **Required:** `https://www.googleapis.com/auth/spreadsheets`
    - **Strongly recommended:** `https://www.googleapis.com/auth/drive.file` for Drive file selection and creation.[^1_1]
    - **Optional read‑only mode:** `https://www.googleapis.com/auth/spreadsheets.readonly` if you truly never write.[^1_1]

And then it walks through, step‑by‑step:

- Creating the Google Cloud project and enabling **Google Sheets API** and **Google Drive API**.[^1_12][^1_9][^1_1]
- Configuring the **OAuth consent screen** with those scopes explicitly added, plus test users.[^1_10][^1_1]
- Wiring the **n8n redirect URI** into the OAuth client, then creating **Google Sheets OAuth2** credentials in n8n and re‑authorizing to get a fresh token with the updated scopes.[^1_5][^1_9][^1_1]

The troubleshooting section closes the loop:

- If you still see **403 / insufficient authentication scopes**, the guide forces you to:
    - Re‑check scopes on the consent screen,
    - Re‑authorize the credential in n8n so the token is re‑issued with the new scope set,
    - Confirm the Google account actually has Editor access on the Sheet.[^1_8][^1_4][^1_1]


## Why This Is Worth Posting

For anyone automating with n8n, this kind of bug is the difference between “demo ready” and “actually shippable”:

- It shows that **DeepSeek is a drop‑in alternative to OpenAI** for this template, as long as you get the API endpoint and key configured correctly in n8n.[^1_7][^1_6][^1_3]
- It turns a vague “Google says no” 403 into a concrete fix: **fix your scopes on the OAuth consent screen, then re‑issue the token**.[^1_4][^1_8][^1_1]

On GitHub, the **n8n‑google‑sheets‑complete‑guide** becomes your living documentation: people can clone it, follow the checklist, and avoid wasting an afternoon on authentication trivia.[^1_9][^1_1]

On LinkedIn, this same story reads as what it is: you didn’t just “set up n8n”; you debugged the identity layer, swapped the LLM vendor, and shipped a repeatable recipe others can use in their own stacks.[^1_3][^1_2][^1_1]
<span style="display:none">[^1_13][^1_14][^1_15][^1_16][^1_17][^1_18][^1_19][^1_20][^1_21]</span>

<div align="center">⁂</div>

[^1_1]: n8n-google-sheets-complete-guide.md

[^1_2]: https://n8n.io/workflows/7156-get-started-with-google-sheets-in-n8n/

[^1_3]: https://docs.n8n.io/integrations/builtin/cluster-nodes/sub-nodes/n8n-nodes-langchain.lmchatdeepseek/

[^1_4]: https://community.n8n.io/t/getting-403-n-error-n-code-403-n-message-request-had-insufficient-authentication-scopes-n-status-permission-denied-n-n-n-when-connecting-to-google-photo-via-http-node/148573

[^1_5]: https://n8n.io/integrations/google-sheets/

[^1_6]: https://onedollarvps.com/blogs/how-to-use-deepseek-in-n8n

[^1_7]: https://n8n.io/integrations/deepseek-chat-model/

[^1_8]: https://community.make.com/t/google-sheets-insufficient-authentication-scope-error/33122

[^1_9]: https://www.codecademy.com/article/n8n-google-sheets-integration

[^1_10]: https://www.linkedin.com/posts/lakshit-ukani_connect-any-google-api-to-n8n-in-5-minutes-activity-7332050309810204672-noSj

[^1_11]: https://www.compliancehub.wiki/deepseeks-training-data-underscores-systemic-privacy-and-compliance-gaps/

[^1_12]: https://www.hostinger.com/in/tutorials/how-to-integrate-n8n-to-google-sheets

[^1_13]: https://www.scribd.com/document/752533028/cektitle

[^1_14]: https://djamgatech.web.app

[^1_15]: https://news.ycombinator.com/item?id=43815523

[^1_16]: https://dbaman.com

[^1_17]: https://blog.elest.io/how-to-use-deepseek-r1-in-n8n/

[^1_18]: https://www.youtube.com/watch?v=pWGXlZBGu4k

[^1_19]: https://hub.ability.ai/workflows/deepseek-ai-integration-quickstart-n8n

[^1_20]: https://stackoverflow.com/questions/65184355/error-403-access-denied-from-google-authentication-web-api-despite-google-acc

[^1_21]: https://www.facebook.com/groups/makeautomation/posts/1383072009337715/


---

# export as markdown

Swapped OpenAI for DeepSeek, hit a 403 with Google Sheets, fixed it by tightening up scopes, and turned the whole thing into a reusable guide you can drop into any n8n stack.[^2_1][^2_2][^2_3]

## Why This Test Run Mattered

This wasn’t “yet another hello world workflow.”
The goal was to sanity‑check the official **“Get Started with Google Sheets in n8n”** template, swap the LLM from OpenAI Chat to **DeepSeek Chat**, and prove that the whole thing still works end‑to‑end: input, reasoning, Sheets read/write.[^2_2][^2_3]

Along the way, Google slapped the workflow with **access_denied / insufficient authentication scopes**, which is exactly the kind of silent landmine that will bite you in production if you just copy‑paste credentials and move on.[^2_4][^2_1]

## The Stack You Actually Ran

- n8n workflow based on the official **Get Started with Google Sheets** template.[^2_2]
- Google Sheets node using OAuth2 with a custom Google Cloud project and consent screen.[^2_5][^2_1]
- DeepSeek Chat wired in as the LLM instead of OpenAI Chat, using n8n’s DeepSeek Chat Model integration / compatible OpenAI-style API.[^2_3][^2_6][^2_7]

The idea:

- n8n pulls rows from a Sheet,
- DeepSeek processes / enriches them,
- n8n writes results back to the same Sheet.[^2_3][^2_5]


## Where It Broke: Google Sheets 403

The failure was clean and annoying:

- Google Sheets node refused to write, returning **403 / access_denied / insufficient authentication scopes** while reads might still behave or partially work.[^2_8][^2_1][^2_4]
- Credentials looked “correct”: OAuth client existed, redirect URI matched, Sheets/Drive APIs were enabled.[^2_9][^2_10][^2_1]

Root cause:
The **OAuth consent screen didn’t include the right data access scopes**, so Google issued a token that simply wasn’t allowed to perform write operations on Sheets.[^2_1][^2_8]

In other words: the app was authenticated, but not authorized to do what the node was asking it to do.

## What You Built: The Scopes-First Google OAuth Guide

Instead of papering over the problem, you wrote a focused guide: **“n8n-google-sheets-complete-guide”**.[^2_1]
The spine of that guide is simple:

- Explain **what scopes are** in plain terms: permission keycards that tell Google exactly what this n8n workflow is allowed to touch.[^2_1]
- Spell out the minimal **scope set that actually works** for read/write Sheets automation from n8n.[^2_11][^2_1]

Concretely, the guide locks in:

- For read/write/update Sheets from n8n:
    - **Required:** `https://www.googleapis.com/auth/spreadsheets`
    - **Strongly recommended:** `https://www.googleapis.com/auth/drive.file` for Drive file selection and creation.[^2_1]
    - **Optional read‑only mode:** `https://www.googleapis.com/auth/spreadsheets.readonly` if you truly never write.[^2_1]

And then it walks through, step‑by‑step:

- Creating the Google Cloud project and enabling **Google Sheets API** and **Google Drive API**.[^2_12][^2_9][^2_1]
- Configuring the **OAuth consent screen** with those scopes explicitly added, plus test users.[^2_10][^2_1]
- Wiring the **n8n redirect URI** into the OAuth client, then creating **Google Sheets OAuth2** credentials in n8n and re‑authorizing to get a fresh token with the updated scopes.[^2_5][^2_9][^2_1]

The troubleshooting section closes the loop:

- If you still see **403 / insufficient authentication scopes**, the guide forces you to:
    - Re‑check scopes on the consent screen,
    - Re‑authorize the credential in n8n so the token is re‑issued with the new scope set,
    - Confirm the Google account actually has Editor access on the Sheet.[^2_4][^2_8][^2_1]


## Why This Is Worth Posting

For anyone automating with n8n, this kind of bug is the difference between “demo ready” and “actually shippable”:

- It shows that **DeepSeek is a drop‑in alternative to OpenAI** for this template, as long as you get the API endpoint and key configured correctly in n8n.[^2_6][^2_7][^2_3]
- It turns a vague “Google says no” 403 into a concrete fix: **fix your scopes on the OAuth consent screen, then re‑issue the token**.[^2_8][^2_4][^2_1]

On GitHub, the **n8n‑google‑sheets‑complete‑guide** becomes your living documentation: people can clone it, follow the checklist, and avoid wasting an afternoon on authentication trivia.[^2_9][^2_1]

On LinkedIn, this same story reads as what it is: you didn’t just “set up n8n”; you debugged the identity layer, swapped the LLM vendor, and shipped a repeatable recipe others can use in their own stacks.[^2_2][^2_3][^2_1]

<div align="center">⁂</div>

[^2_1]: https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/56011711/7654fd79-7bcf-4cf1-88d7-89a9fe4d0fa6/n8n-google-sheets-complete-guide.md

[^2_2]: https://n8n.io/workflows/7156-get-started-with-google-sheets-in-n8n/

[^2_3]: https://docs.n8n.io/integrations/builtin/cluster-nodes/sub-nodes/n8n-nodes-langchain.lmchatdeepseek/

[^2_4]: https://community.n8n.io/t/getting-403-n-error-n-code-403-n-message-request-had-insufficient-authentication-scopes-n-status-permission-denied-n-n-n-when-connecting-to-google-photo-via-http-node/148573

[^2_5]: https://n8n.io/integrations/google-sheets/

[^2_6]: https://onedollarvps.com/blogs/how-to-use-deepseek-in-n8n

[^2_7]: https://n8n.io/integrations/deepseek-chat-model/

[^2_8]: https://community.make.com/t/google-sheets-insufficient-authentication-scope-error/33122

[^2_9]: https://www.codecademy.com/article/n8n-google-sheets-integration

[^2_10]: https://www.linkedin.com/posts/lakshit-ukani_connect-any-google-api-to-n8n-in-5-minutes-activity-7332050309810204672-noSj

[^2_11]: https://www.compliancehub.wiki/deepseeks-training-data-underscores-systemic-privacy-and-compliance-gaps/

[^2_12]: https://www.hostinger.com/in/tutorials/how-to-integrate-n8n-to-google-sheets

