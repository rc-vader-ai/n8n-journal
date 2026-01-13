<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# how to n8n ai agent connect to notion database, please share in detail step by step. The follow is the Notion database requireemnt: "Configure Credentials \& Notion Database:

Notion Database:
Create or designate a Notion database (like the example "Knowledge Database") where articles will be saved.
Ensure this database has the following properties (fields):
Name (Type: Text) - This will store the article title.
URL (Type: URL) - This will store the original article link.
Description (Type: Text) - This can store the AI-generated summary.
Tags (Type: Multi-select) - Optional, for categorization.
Publication Date (Type: Date) - *Optional,
store the date the article was published."

To connect an **n8n AI Agent** to a **Notion Database** and fulfill your specific field requirements, you need to set up Notion credentials, configure the n8n Notion node to map your fields correctly, and then attach that node as a "Tool" to your AI Agent.

### **Quick Answer**

1. **Notion Side:** Create an "Internal Integration" in Notion settings to get an API Secret. "Share" your "Knowledge Database" with this new integration.
2. **n8n Side:** Create a **Notion** node. Set **Resource** to `Database Page` and **Operation** to `Create`.
3. **Field Mapping:** Map the AI's output to the Notion fields: `Name` (Title), `URL` (URL), `Description` (Text), `Tags` (Multi-select), and `Publication Date` (Date).
4. **Agent Connection:** Connect this Notion node to the **Tools** input of your **AI Agent** node.

***

### **Step-by-Step Configuration Guide**

#### **Step 1: Configure Notion Credentials \& Database Access**

Before n8n can "see" your database, you must give it permission.

1. **Create Integration:**
    * Go to [My Integrations](https://www.notion.so/my-integrations) in Notion.
    * Click **+ New integration**.
    * Name it (e.g., "n8n AI Agent") and select the workspace where your database lives.
    * Click **Submit**.
    * **Copy the "Internal Integration Secret"**. (You will need this for n8n).[^1][^2]
2. **Share Database:**
    * Open your specific "Knowledge Database" in Notion.
    * Click the **three dots (...)** in the top-right corner of the page.
    * Click **Connect to** (or "Add connections" depending on your version).
    * Search for and select the "n8n AI Agent" integration you just created.
    * *Note: If you don't do this, n8n will see "No databases found".*.[^3][^1]

#### **Step 2: Set Up n8n Credentials**

1. In your n8n workflow, click **Credentials** (sidebar) or open a Notion node.
2. Select **Notion API**.
3. Paste the **Internal Integration Secret** you copied in Step 1.
4. Click **Save**.

#### **Step 3: Configure the Notion Node (The Tool)**

This node will act as the "hand" the AI uses to save data.

1. **Add Node:** Add a **Notion** node to your canvas.
2. **Setup Action:**
    * **Credential:** Select the credential you just created.
    * **Resource:** `Database Page`
    * **Operation:** `Create`
    * **Database:** Click the dropdown. If you shared it correctly in Step 1, "Knowledge Database" should appear. Select it.
3. **Map Fields (Data Requirements):**
You will see the properties from your database appear. Configure them as follows to match your requirements:[^4][^5]
| Notion Property | n8n Field Setting | Configuration / Expression |
| :-- | :-- | :-- |
| **Name** (Title) | **Title** | Map to the Article Title (e.g., from AI input). |
| **URL** (URL) | **URL** | Map to the Article Link. |
| **Description** (Text) | **Description** | Map to the AI-generated summary. |
| **Tags** (Multi-select) | **Tags** | **For dynamic tags:** Toggle "Expression" mode. Pass an array of strings, e.g., `{{ ['AI', 'Tech'] }}`. If the AI outputs a comma-separated string, use `.split(',')`. |
| **Publication Date** | **Publication Date** | Use ISO 8601 format (e.g., `2025-12-27`). If the AI gives a fuzzy date like "today", use the **Date \& Time** node before this to format it strictly. |

*Tip: If fields don't appear automatically, click "Refresh List" at the top of the node properties.*

#### **Step 4: Connect to the AI Agent**

To make the AI "smart" enough to use this database, you must connect it as a **Tool**.

1. **Structure:** Ensure your **AI Agent** node is on the canvas.
2. **Connection:** Drag a connection from the **Notion** node you just configured to the **Tools** input (usually the bottom/secondary input) of the **AI Agent** node.
    * *Note: In some n8n versions, you use the "Tool" node wrapper, but often you can directly connect compatible nodes or use the "Custom n8n Workflow Tool" if you need complex logic.*
3. **System Prompt:** In the AI Agent's settings, update the **System Message** to instruct the agent on how to use the tool:
> "You are a knowledge assistant. When you find an article, use the 'Notion' tool to save it. Always extract the Title, URL, write a brief Description, assign relevant Tags, and find the Publication Date."

### **Troubleshooting Common Issues**

* **"Database not found":** 99% of the time, this is because you didn't click "Add connections" inside the specific Notion database page.[^6][^3]
* **Multi-select Errors:** Notion is picky about Multi-select fields. If passing a simple list fails, try passing an array of objects in Expression mode: `{{ [ {"name": "Tag1"}, {"name": "Tag2"} ] }}`.[^7]
* **Date Errors:** Ensure the date is a real date object or a string in `YYYY-MM-DD` format. Notion will reject "December 27th".[^8]
<span style="display:none">[^10][^11][^12][^13][^14][^15][^16][^17][^18][^19][^20][^21][^22][^23][^24][^25][^26][^27][^28][^29][^30][^31][^32][^33][^34][^35][^36][^37][^38][^39][^40][^41][^42][^43][^44][^45][^46][^47][^48][^49][^50][^51][^52][^53][^54][^9]</span>

<div align="center">⁂</div>

[^1]: https://www.youtube.com/watch?v=9U488r_63Xk

[^2]: https://www.youtube.com/watch?v=SarPuXVBANk

[^3]: https://www.gobuildwithai.com/topics/learn-n8n/n8n-notion-connection-tutorial

[^4]: https://www.youtube.com/watch?v=sGIKj1Y-ugU

[^5]: https://www.youtube.com/watch?v=Efadt0TnDWA

[^6]: https://community.n8n.io/t/cannot-make-a-connection-to-notion-database-no-mather-what-i-try/81721

[^7]: https://community.n8n.io/t/how-to-reliably-create-new-multi-select-options-in-notion/134857

[^8]: https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.datetime/

[^9]: https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/collection_07cb6b35-bb76-4405-9882-9df4b2828044/1d11fc81-8fda-42b0-a576-e49af933638c/Cloudflare-Zero-Trust-Network-Tunnel-with-Docker-n-2c933a6c2f2680c9a2aadf4f297665b6.md

[^10]: https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.notion/

[^11]: https://www.youtube.com/watch?v=giJFViLostM

[^12]: https://www.youtube.com/watch?v=XCq1DoasWbw

[^13]: https://github.com/n8n-io/n8n/issues/5050

[^14]: https://n8n.io/integrations/notion/

[^15]: https://hackceleration.com/tutorials/build-ai-agent-n8n/

[^16]: https://community.n8n.io/t/setting-up-notion-relation-on-a-record-creation/168601

[^17]: https://www.youtube.com/watch?v=VV0SW1b_3-U

[^18]: https://community.n8n.io/t/how-to-use-notion-tool-as-an-internal-knowledge-base-data-in-ai-agent-node/108628

[^19]: https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.set/

[^20]: https://n8n.io/workflows/2415-notion-ai-assistant-generator/

[^21]: https://n8n.io/ai-agents/

[^22]: https://www.reddit.com/r/n8n/comments/1iuivpk/new_to_n8n_and_trying_to_use_it_and_an_api_to/

[^23]: https://www.notion.com/templates/knowledge-base-ai-assistant-with-n8n

[^24]: https://www.simonesmerilli.com/business/notion-n8n-nodes

[^25]: https://www.youtube.com/watch?v=hj3luW5CsQ8

[^26]: https://www.youtube.com/watch?v=up2Wi3Y9a-4

[^27]: https://www.youtube.com/watch?v=Fyzpd9wjGb0

[^28]: https://www.youtube.com/watch?v=EgKa0Jdf_rc

[^29]: https://www.youtube.com/watch?v=PDBccLDXWvQ

[^30]: https://www.youtube.com/watch?v=MaY0roccdns

[^31]: https://www.youtube.com/watch?v=pqIdM_p9rR0

[^32]: https://www.youtube.com/watch?v=7OXgJ0F1kDQ

[^33]: https://www.youtube.com/watch?v=Z_IVUYigglI

[^34]: https://www.youtube.com/watch?v=Ue4sC-PuVxw

[^35]: https://www.youtube.com/watch?v=ynLZwS2Nhnc

[^36]: https://www.youtube.com/watch?v=Hk1aBqLbFzU

[^37]: https://www.youtube.com/watch?v=6VTI1m2ystc

[^38]: https://community.n8n.io/t/n8n-x-notion-get-database-id-with-ai-agent/155807

[^39]: https://www.facebook.com/groups/641559818473198/posts/675653515063828/

[^40]: https://www.youtube.com/watch?v=Fu_9INcwZqw

[^41]: https://www.youtube.com/watch?v=2FzKbyo2lh8

[^42]: https://community.n8n.io/t/how-to-make-ai-agent-search-notion-pages-and-their-content-in-a-database/81180

[^43]: https://community.n8n.io/t/n8n-works-bad-with-notion-date-based-formulas-get-all-filtering-secretly-iimplements-timezone/15829

[^44]: https://community.n8n.io/t/create-notion-page-blocks-from-ai-response/110905

[^45]: https://www.notion.com/help/autofill

[^46]: https://www.youtube.com/watch?v=sFfWfpz65CY

[^47]: https://www.youtube.com/watch?v=z-gDDajSSNE

[^48]: https://www.youtube.com/watch?v=hkLaTtE0PSU

[^49]: https://www.youtube.com/watch?v=lK3veuZAg0c

[^50]: https://www.youtube.com/watch?v=xvxD_Q5iQJc

[^51]: https://www.youtube.com/watch?v=VJiu1rPxLzg

[^52]: https://www.youtube.com/watch?v=WCWw5CJF36U

[^53]: https://www.youtube.com/watch?v=CVdnZZ-QejE

[^54]: https://www.youtube.com/watch?v=GchXMRwuWxE

