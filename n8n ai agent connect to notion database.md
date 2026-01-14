# The follow is the Notion database requireemnt: "Configure Credentials \& Notion Database:

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
You will see the properties from your database appear. Configure them as follows to match your requirements:

| Notion Property     | n8n Field Setting | Configuration / Expression                                                                                                                                     |
| ------------------- | ----------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Name (Title)        | Title             | Map to the Article Title (e.g., from AI input).                                                                                                                |
| URL (URL)           | URL               | Map to the Article Link.                                                                                                                                       |
| Description (Text)  | Description       | Map to the AI-generated summary.                                                                                                                               |
| Tags (Multi-select) | Tags              | For dynamic tags: Toggle "Expression" mode. Pass an array of strings, e.g., {{ ['AI', 'Tech'] }}. If the AI outputs a comma-separated string, use .split(','). |
| Publication Date    | Publication Date  | Use ISO 8601 format (e.g., 2025-12-27). If the AI gives a fuzzy date like "today", use the Date & Time node before this to format it strictly.                 |
*Tip: If fields don't appear automatically, click "Refresh List" at the top of the node properties.*

#### **Step 4: Connect to the AI Agent**

To make the AI "smart" enough to use this database, you must connect it as a **Tool**.

1. **Structure:** Ensure your **AI Agent** node is on the canvas.
2. **Connection:** Drag a connection from the **Notion** node you just configured to the **Tools** input (usually the bottom/secondary input) of the **AI Agent** node.
    * *Note: In some n8n versions, you use the "Tool" node wrapper, but often you can directly connect compatible nodes or use the "Custom n8n Workflow Tool" if you need complex logic.*
3. **System Prompt:** In the AI Agent's settings, update the **System Message** to instruct the agent on how to use the tool:
> "You are a knowledge assistant. When you find an article, use the 'Notion' tool to save it. Always extract the Title, URL, write a brief Description, assign relevant Tags, and find the Publication Date."

### **Troubleshooting Common Issues**

* **"Database not found":** 99% of the time, this is because you didn't click "Add connections" inside the specific Notion database page.
