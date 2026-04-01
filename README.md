# 🎯 DayZ Config Editor

A free, browser-based editor for DayZ Nitrado server configuration files — with an optional AI assistant powered by Claude.

> No installs. No downloads. Just open the link and drag in your files.

---

## ✨ Features

- **Multi-file support** — Load multiple config files at once with tabbed navigation
- **Dedicated editors** for `types.xml`, `events.xml`, `globals.xml`, and `cfgplayerspawnpoints.xml`
- **Raw text editor** for any other file (`cfggameplay.json`, `cfgspawnabletypes.xml`, `serverDZ.cfg`, etc.)
- **Inline editing** — Edit values directly in the table without opening separate dialogs
- **Change tracking** — Every modification is highlighted and tracked with a before/after diff view
- **Export** — Download your modified file as properly formatted XML/JSON, ready to upload to Nitrado
- **AI Assistant** *(optional)* — Ask questions about your files or apply bulk edits using natural language

---

## 📁 Supported Files

| File | Editor Type | Description |
|------|------------|-------------|
| `types.xml` | Full editor | Loot economy — item spawns, quantities, lifetimes |
| `events.xml` | Full editor | Dynamic events — heli crashes, animals, vehicles |
| `globals.xml` | Card editor | Economy-wide multipliers and limits |
| `cfgplayerspawnpoints.xml` | Table editor | Coastal/inland player spawn points |
| `cfggameplay.json` | Raw editor | Stamina, base building, inventory settings |
| `cfgspawnabletypes.xml` | Raw editor | Cargo/attachment presets per item |
| `cfgeventspawns.xml` | Raw editor | Zone positions for dynamic events |
| Any other `.xml`, `.json`, `.cfg` | Raw editor | Edit and export any config file |

---

## 🚀 Using the Editor

1. Open the editor URL in any browser
2. Drag and drop your config file onto the upload area, or click to browse
3. Edit your values — changes highlight in real time
4. Switch to the **CHANGES** tab to review your diff before exporting
5. Click **EXPORT** to download your modified file
6. Upload the exported file back to your Nitrado server via FTP or the file manager

---

## 🤖 Setting Up the AI Assistant (Optional)

The AI assistant lets you ask questions about your files and apply bulk edits using plain English — for example:

- *"What does restock do?"*
- *"Make all rifles 30% rarer"*
- *"Set zombie nominal count to 400"*
- *"Explain the difference between nominal and min"*

To enable it, you need two things: an **Anthropic API key** and a **Cloudflare Worker proxy**.

---

### Step 1 — Get an Anthropic API Key

1. Go to [console.anthropic.com](https://console.anthropic.com) and sign up
2. Click **API Keys** in the left sidebar
3. Click **Create Key** — give it any name you like
4. **Copy the key immediately** — it starts with `sk-ant-...` and is only shown once
5. Add a small credit balance under **Billing** (as little as $5 lasts a very long time at this usage level)

> **Cost estimate:** Each AI message costs roughly $0.001–$0.003. Heavy daily use by two people is unlikely to exceed $2–3/month.

---

### Step 2 — Create a Cloudflare Worker Proxy

The proxy is required because browsers block direct API calls from web pages (a security restriction called CORS). The Worker sits in the middle and forwards your requests to Anthropic.

**Cloudflare Workers are free** for this level of usage (100,000 requests/day on the free tier).

#### 2a. Create a Cloudflare account

Go to [workers.cloudflare.com](https://workers.cloudflare.com) and sign up for free.

#### 2b. Create a new Worker

1. In the Cloudflare dashboard, click **Workers & Pages**
2. Click **Create** → **Create Worker**
3. Choose the **Hello World** template
4. Click **Deploy** once to get into the editor
5. Click **Edit Code**

#### 2c. Paste the Worker code

Select all the existing code in the editor and replace it with the following:

```javascript
export default {
  async fetch(request, env) {

    // Handle CORS preflight requests
    if (request.method === "OPTIONS") {
      return new Response(null, {
        headers: {
          "Access-Control-Allow-Origin": "*",
          "Access-Control-Allow-Methods": "POST, OPTIONS",
          "Access-Control-Allow-Headers": "Content-Type, x-api-key, anthropic-version",
        },
      });
    }

    // Forward the request to the Anthropic API
    const body = await request.json();
    const apiKey = request.headers.get("x-api-key");

    const response = await fetch("https://api.anthropic.com/v1/messages", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        "x-api-key": apiKey,
        "anthropic-version": "2023-06-01",
      },
      body: JSON.stringify(body),
    });

    const data = await response.json();

    // Return response with CORS headers so the browser accepts it
    return new Response(JSON.stringify(data), {
      headers: {
        "Content-Type": "application/json",
        "Access-Control-Allow-Origin": "*",
      },
    });
  },
};
```

#### 2d. Deploy the Worker

1. Click **Deploy** in the top right
2. Your Worker will be assigned a URL like:
   ```
   https://your-worker-name.your-username.workers.dev
   ```
3. **Copy this URL** — you'll need it in the next step

---

### Step 3 — Connect Everything in the Editor

1. Open the editor in your browser
2. On first launch, an **AI Setup** screen will appear automatically
3. Paste your **Anthropic API key** into the API Key field
4. Paste your **Cloudflare Worker URL** into the Proxy URL field
5. Click **Test Connection** — you should see a green ✅
6. Click **Save & Continue**

Your settings are saved in your browser's local storage — you won't need to enter them again on this device.

To update your settings later, click the **⚙️** icon in the top right corner of the editor.

---

### Sharing With Others

Anyone can use the editor to load and edit files for free — the AI assistant is the only part that requires a key.

If you want to share AI access with a teammate:
- Share your Cloudflare Worker URL with them — it is safe to share
- Share your Anthropic API key with them privately (treat it like a password)
- They enter both in ⚙️ Settings on their device
- All usage rolls up to your single Anthropic billing account

> **Tip:** You can revoke or regenerate your API key at any time at `console.anthropic.com → API Keys` without affecting the Worker.

---

## 🔒 Privacy & Security

- **Your files never leave your browser.** All parsing, editing, and exporting happens locally.
- **Your API key is stored only in your browser's local storage.** It is never sent to GitHub or any third party — only to the Anthropic API (via your own Cloudflare Worker).
- **The Cloudflare Worker does not log your requests or store any data.** It only forwards them.
- **Do not share your API key publicly** (e.g. don't paste it into a GitHub issue or public chat).

---

## 🛠️ Running Locally (Optional)

If you prefer to run it offline without GitHub Pages, just download `index.html` and open it directly in your browser. All file editing features work offline. The AI assistant requires an internet connection regardless of how you run it.

---

## 📝 License

Free to use and modify. If you improve it, feel free to open a pull request.
