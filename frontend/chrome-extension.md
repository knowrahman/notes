
### **Step by step guide to learning Chrome Extensions**


* * * * *

### **Stage 1: Foundations**

#### 1\. **Understanding the Manifest File (manifest.json)**

-   **Explanation:** The manifest defines your extension's metadata and configuration.

-   **Example:**

```
{
  "manifest_version": 3,
  "name": "My First Extension",
  "version": "1.0",
  "action": {
    "default_popup": "popup.html"
  },
  "permissions": ["storage"]
}

```

-   **Mini Task:** Create a `manifest.json` and a simple `popup.html` that says "Hello, Chrome!"

* * * * *

#### 2\. **Popup Page UI**

-   **Explanation:** The popup is the UI that opens when you click the extension icon.

-   **Mini Task:** Style your `popup.html` with CSS and add a JavaScript button that changes text on click.

* * * * *

### **Stage 2: Core Functionality**

#### 3\. **Content Scripts**

-   **Explanation:** JavaScript that runs in the context of web pages.

-   **Example:**

```
// content.js
document.body.style.backgroundColor = "lightblue";

```

-   **Mini Task:** Inject a content script that highlights all paragraphs on any visited website.

* * * * *

#### 4\. **Background Scripts**

-   **Explanation:** Run in the background and handle long-lived tasks like alarms or fetch requests.

-   **Mini Task:** Use a background script to log a message every time a tab is updated.

* * * * *

#### 5\. **Permissions**

-   **Explanation:** Define access needs like tabs, storage, activeTab, etc.

-   **Mini Task:** Add the `"tabs"` permission and fetch the current tab's title.

* * * * *

#### 6\. **Messaging (between popup, background, and content)**

-   **Explanation:** Enables communication between different parts of your extension.

-   **Mini Task:** Send a message from `popup.js` to `content.js` to change the background color of the current page.

* * * * *

### **Stage 3: Data Handling and APIs**

#### 7\. **Chrome Storage API**

-   **Explanation:** Store and retrieve data using `chrome.storage.sync` or `local`.

-   **Example:**

```
chrome.storage.sync.set({ color: "blue" });

```

-   **Mini Task:** Create a popup form that lets you save and retrieve a favorite color.

* * * * *

#### 8\. **Working with the DOM in Content Scripts**

-   **Mini Task:** Build a content script that counts and displays the number of images on a webpage.

* * * * *

### **Stage 4: Advanced Features**

#### 9\. **Context Menus**

-   **Explanation:** Add right-click options to the browser context menu.

-   **Mini Task:** Add a context menu to copy selected text to clipboard.

* * * * *

#### 10\. **Alarms and Timers**

-   **Mini Task:** Create an alarm that notifies the user every 30 minutes.

* * * * *

#### 11\. **Using External APIs**

-   **Mini Task:** Fetch a quote or weather info from a public API and show it in your popup.

* * * * *

#### 12\. **Chrome Identity API (Optional)**

-   **Mini Task:** Use Chrome identity to get user info (if needed for auth-related projects).

* * * * *

### **Stage 5: Deployment**

#### 13\. **Icons and Assets**

-   Add icons at 16x16, 48x48, 128x128 sizes and link them in your manifest.

#### 14\. **Publishing to Chrome Web Store**

-   Prepare your ZIP file, create a Chrome Developer account, and follow the publishing steps.

* * * * *

### **Final Project: Real-World Chrome Extension**

**Project Idea:** *Quick Language Translator*

-   **Features:**

    -   Highlight text → Translate with context menu

    -   Store history of translations using `chrome.storage`

    -   Use a public translation API

    -   Clean popup UI with history and settings

* * * * *


Great question. Let's break down the `manifest.json` file, which is the *blueprint* of every Chrome extension. It tells Chrome everything about your extension---its name, version, functionality, permissions, and the files it needs to run.

* * * * *

### **Overview**

-   It must be named `manifest.json`

-   It is required and must be in the root folder

-   It follows strict JSON format (no comments allowed)

* * * * *

### **Detailed Breakdown of a Sample Manifest File**

```
{
  "manifest_version": 3,
  "name": "My First Extension",
  "description": "A basic Chrome extension popup example",
  "version": "1.0",
  "action": {
    "default_popup": "popup.html",
    "default_icon": {
      "16": "icon16.png",
      "48": "icon48.png",
      "128": "icon128.png"
    }
  },
  "permissions": [],
  "icons": {
    "16": "icon16.png",
    "48": "icon48.png",
    "128": "icon128.png"
  }
}

```

* * * * *

### **Field-by-Field Explanation**

| Field | Purpose |
| --- | --- |
| `"manifest_version"` | **REQUIRED**. Must be `3` for modern extensions. Chrome requires this to understand the format. |
| `"name"` | **REQUIRED**. The name that appears in Chrome's Extensions list and Chrome Web Store. |
| `"description"` | Optional but recommended. A short explanation of what your extension does. |
| `"version"` | **REQUIRED**. Follows semantic versioning (`major.minor.patch`). Used for updates. |

* * * * *

### **Action Block**

```
"action": {
  "default_popup": "popup.html",
  "default_icon": {
    "16": "icon16.png",
    "48": "icon48.png",
    "128": "icon128.png"
  }
}

```

-   **`default_popup`**: Specifies the HTML file that opens when the extension icon is clicked.

-   **`default_icon`**: Icons for different DPI screens. Optional but recommended for branding.

* * * * *

### **Permissions**

```
"permissions": ["storage", "tabs"]

```

-   This tells Chrome what APIs or features your extension will use.

-   Common examples:

    -   `"storage"` -- Access to the Chrome Storage API

    -   `"tabs"` -- Interact with the browser's open tabs

    -   `"activeTab"` -- Temporary access to the current tab after user action

**Important**: Only request what you need --- Chrome will warn users based on these permissions.

* * * * *

### **Icons**

```
"icons": {
  "16": "icon16.png",
  "48": "icon48.png",
  "128": "icon128.png"
}

```

Used for:

-   Extension toolbar icon

-   Chrome Web Store listing

-   Context menus or notifications

* * * * *

### **Other Common Fields (We'll Use Later)**

| Field | Description |
| --- | --- |
| `"background"` | For background scripts (event handlers, persistent tasks) |
| `"content_scripts"` | To inject JS/CSS into web pages |
| `"host_permissions"` | Needed to access specific URLs (`"<all_urls>"` or `"https://example.com/*"`) |
| `"options_page"` | Points to an options/settings page |
| `"web_accessible_resources"` | Files accessible to web pages (images, scripts) |

* * * * *
Absolutely! Here's a **full-fledged example of a `manifest.json` file** that includes **content scripts**, **background service worker**, **permissions**, **host permissions**, **web accessible resources**, **options page**, **action (popup)**, and **icons**—everything you might typically need for a mid-sized Chrome extension.

---

### 📄 `manifest.json` (Complete Example for Manifest V3)

```json
{
  "manifest_version": 3,
  "name": "Productivity Booster",
  "description": "A Chrome extension that boosts productivity by blocking distractions and tracking usage.",
  "version": "1.0",
  
  "icons": {
    "16": "icons/icon16.png",
    "48": "icons/icon48.png",
    "128": "icons/icon128.png"
  },

  "action": {
    "default_popup": "popup.html",
    "default_icon": {
      "16": "icons/icon16.png",
      "48": "icons/icon48.png",
      "128": "icons/icon128.png"
    }
  },

  "background": {
    "service_worker": "background.js"
  },

  "content_scripts": [
    {
      "matches": ["<all_urls>"],
      "js": ["content.js"],
      "css": ["content.css"],
      "run_at": "document_idle"
    }
  ],

  "permissions": [
    "storage",
    "tabs",
    "notifications",
    "scripting"
  ],

  "host_permissions": [
    "<all_urls>"
  ],

  "web_accessible_resources": [
    {
      "resources": ["icons/icon48.png", "assets/sound.mp3"],
      "matches": ["<all_urls>"]
    }
  ],

  "options_page": "options.html",

  "content_security_policy": {
    "extension_pages": "script-src 'self'; object-src 'self';"
  }
}
```

---

### ✅ What Each Section Does:

| Field                                  | Description                                                                  |
| -------------------------------------- | ---------------------------------------------------------------------------- |
| `"manifest_version"`                   | Always `3` for latest extensions                                             |
| `"name"`, `"description"`, `"version"` | Basic metadata                                                               |
| `"icons"`                              | Sets icons used throughout Chrome                                            |
| `"action"`                             | Defines popup UI and toolbar icons                                           |
| `"background"`                         | Uses a service worker (`background.js`) for persistent event handling        |
| `"content_scripts"`                    | Injects JS & CSS into matching web pages (`<all_urls>` = all sites)          |
| `"permissions"`                        | Grants access to APIs like `tabs`, `storage`, `notifications`, etc.          |
| `"host_permissions"`                   | Specifies what URLs the extension is allowed to access                       |
| `"web_accessible_resources"`           | Allows listed files (like icons or audio) to be used inside the web page DOM |
| `"options_page"`                       | Settings page users can open from the Extensions list                        |
| `"content_security_policy"`            | CSP for Manifest V3 (must use `self`, no inline JS allowed)                  |

---
Great! Here's a full example based on the manifest we discussed. This extension will:

-   Inject content into pages (via `content.js`)

-   Show a popup with a button and display time

-   Use a background service worker (`background.js`)

-   Save and load settings via `options.html`

-   Show notifications

-   Store user preferences

* * * * *

📁 Project Structure
--------------------

```
productivity-booster/
│
├── manifest.json
├── background.js
├── content.js
├── content.css
├── popup.html
├── popup.js
├── options.html
├── options.js
├── icons/
│   ├── icon16.png
│   ├── icon48.png
│   └── icon128.png
├── assets/
│   └── sound.mp3

```

* * * * *

### 1️⃣ `popup.html`

```
<!DOCTYPE html>
<html>
<head>
  <title>Productivity Booster</title>
  <script src="popup.js" defer></script>
</head>
<body>
  <h2>Welcome Back!</h2>
  <p id="time">Loading time...</p>
  <button id="notify">Remind Me</button>
</body>
</html>

```

* * * * *

### 2️⃣ `popup.js`

```
document.getElementById("notify").addEventListener("click", () => {
  chrome.runtime.sendMessage({ type: "notify" });
});

document.addEventListener("DOMContentLoaded", () => {
  const now = new Date().toLocaleTimeString();
  document.getElementById("time").innerText = `Current time: ${now}`;
});

```

* * * * *

### 3️⃣ `background.js`

```
chrome.runtime.onInstalled.addListener(() => {
  console.log("Extension installed.");
});

chrome.runtime.onMessage.addListener((msg, sender, sendResponse) => {
  if (msg.type === "notify") {
    chrome.notifications.create({
      type: "basic",
      iconUrl: "icons/icon48.png",
      title: "Time to Refocus",
      message: "Take a quick break or refocus your energy!"
    });
  }
});

```

* * * * *

### 4️⃣ `content.js`

```
console.log("Content script injected.");
document.body.style.border = "5px solid #4CAF50";

```

* * * * *

### 5️⃣ `content.css`

```
body {
  font-family: sans-serif;
}

h1, h2 {
  color: darkgreen !important;
}

```

* * * * *

### 6️⃣ `options.html`

```
<!DOCTYPE html>
<html>
<head>
  <title>Extension Settings</title>
  <script src="options.js" defer></script>
</head>
<body>
  <h2>Settings</h2>
  <label>
    Reminder Interval (minutes):
    <input type="number" id="interval" min="1" />
  </label>
  <button id="save">Save</button>
</body>
</html>

```

* * * * *

### 7️⃣ `options.js`

```
document.addEventListener("DOMContentLoaded", () => {
  chrome.storage.sync.get("interval", (data) => {
    document.getElementById("interval").value = data.interval || 30;
  });

  document.getElementById("save").addEventListener("click", () => {
    const interval = parseInt(document.getElementById("interval").value);
    chrome.storage.sync.set({ interval });
    alert("Settings saved!");
  });
});

```

* * * * *

### 🔔 Extras (optional to test)

-   Add a small MP3 file in `/assets/sound.mp3` if you want to later play sound via `Audio`.

-   Test your content script on any website like `example.com`.

* * * * *

### ✅ What You Can Test Now

-   Load this folder in `chrome://extensions`

-   Click your extension: see the popup and time

-   Click **Remind Me** → see a notification

-   Right-click the extension → **Options** → Save interval setting

-   Open any webpage → a green border appears from `content.js`

* * * * *


Great! Let's move to the **next topic: Messaging Between Scripts**, which is essential for communication between:

* **Popup** ↔ **Background**
* **Content script** ↔ **Background**
* **Popup** ↔ **Content script** (via background)

---

## 🔄 Messaging Between Scripts

### **Why Messaging Matters**

Chrome extensions run in isolated environments:

* Popup closes when not focused.
* Background script runs independently.
* Content scripts run in the context of the page.

Messaging connects these parts.

---

### 🔧 Use Case:

You click a button in your **popup**, and it tells the **content script** to highlight all `<p>` elements on the current page.

---

### 1️⃣ Modify `popup.js` to send a message:

```javascript
document.getElementById("notify").addEventListener("click", () => {
  chrome.tabs.query({ active: true, currentWindow: true }, (tabs) => {
    chrome.tabs.sendMessage(tabs[0].id, { action: "highlightParagraphs" });
  });
});
```

---

### 2️⃣ Update `content.js` to receive the message:

```javascript
chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
  if (message.action === "highlightParagraphs") {
    document.querySelectorAll("p").forEach(p => {
      p.style.backgroundColor = "yellow";
      p.style.fontWeight = "bold";
    });
  }
});
```

---

### 3️⃣ Permissions Update (Optional)

Your `manifest.json` already has:

```json
"permissions": ["tabs", "scripting"]
```

No change needed unless you removed them.

---

### ✅ Test It:

1. Open any site with paragraphs (e.g., [https://example.com](https://example.com))
2. Click your extension
3. Click the **Remind Me** button (or rename it to "Highlight Paragraphs" if you'd like)
4. Watch all `<p>` tags turn yellow

---

### 🧠 Bonus Task:

Try sending a message *from* the content script back to the background script or popup saying:
“Paragraphs highlighted successfully.”


Awesome — let’s dive into the **Chrome Storage API**, which lets your extension save and retrieve data, such as user preferences, notes, themes, or time logs.

---

## 💾 Chrome Storage API

### **Why It's Important**

It allows you to persist data **across sessions** and **sync** it between devices (with `storage.sync`), or store it locally (`storage.local`).

---

### 🔧 Basic Syntax

```javascript
// Set data
chrome.storage.sync.set({ key: "value" }, () => {
  console.log("Data saved");
});

// Get data
chrome.storage.sync.get("key", (data) => {
  console.log("Fetched:", data.key);
});
```

> Use `chrome.storage.sync` for data that should follow the user across devices
> Use `chrome.storage.local` for large or device-specific data

---

### 🛠 Mini Task: Build a Simple Note Saver

### 1️⃣ Add to `popup.html`

```html
<textarea id="note" placeholder="Write your note here..."></textarea>
<button id="saveNote">Save Note</button>
```

### 2️⃣ Update `popup.js`

```javascript
document.addEventListener("DOMContentLoaded", () => {
  chrome.storage.sync.get("userNote", (data) => {
    document.getElementById("note").value = data.userNote || "";
  });
});

document.getElementById("saveNote").addEventListener("click", () => {
  const note = document.getElementById("note").value;
  chrome.storage.sync.set({ userNote: note }, () => {
    alert("Note saved!");
  });
});
```

### 3️⃣ Permissions in `manifest.json`

Make sure you have `"storage"` in your permissions:

```json
"permissions": ["storage"]
```

---

### ✅ Test It:

1. Open your extension.
2. Type a note and click **Save Note**.
3. Close and reopen the extension popup — your note should persist.

---

### 🔄 Bonus Task:

Use `chrome.storage.onChanged` in the background to **listen for changes** and trigger a notification every time a note is saved.

Great! Now let’s dive into **Working with the DOM using Content Scripts** — one of the most powerful and common use cases in Chrome extension development.

---

## 🧱 Working with the DOM (via Content Scripts)

### **What Are Content Scripts?**

Content scripts are JS files that run **in the context of web pages**, allowing you to read, modify, or interact with the page’s DOM — like changing text, highlighting elements, adding buttons, etc.

They don’t have direct access to Chrome APIs (like `chrome.storage`), but they **can message** background/popup scripts that do.

---

### 🔧 Mini Project: Inject a “Focus Mode” Button on Any Webpage

This button will:

* Appear on every page you visit
* When clicked, it will hide all images and ads (simple DOM elements)

---

### 1️⃣ `content.js`

```javascript
function createFocusButton() {
  const button = document.createElement("button");
  button.textContent = "🧘 Focus Mode";
  button.style.position = "fixed";
  button.style.bottom = "20px";
  button.style.right = "20px";
  button.style.zIndex = "9999";
  button.style.padding = "10px";
  button.style.background = "#4CAF50";
  button.style.color = "white";
  button.style.border = "none";
  button.style.borderRadius = "5px";
  button.style.cursor = "pointer";

  button.addEventListener("click", () => {
    document.querySelectorAll("img, iframe, .ad, [class*='ad']").forEach(el => el.style.display = "none");
    alert("Focus mode activated!");
  });

  document.body.appendChild(button);
}

window.addEventListener("load", createFocusButton);
```

---

### 2️⃣ Add to `manifest.json`

Ensure this block exists:

```json
"content_scripts": [
  {
    "matches": ["<all_urls>"],
    "js": ["content.js"],
    "run_at": "document_idle"
  }
],
```

> `"document_idle"` means the script runs after the page is mostly loaded.

---

### ✅ Test It:

1. Visit any website (news sites work well)
2. See a green “Focus Mode” button appear
3. Click it → all images/ads disappear

---

### 🧠 Pro Tips:

* Avoid affecting the user’s normal experience unintentionally (always use clear UI).
* Try namespacing your injected elements using unique `id`s or `class` names to avoid collisions.
* Always clean up if the script reinjects.

---

Awesome! Let’s move to the next topic: **Context Menus** — a super useful feature to let users interact with your extension via right-click options.

---

## 🖱️ Context Menus in Chrome Extensions

### **What Is It?**

Context menus are custom right-click menu options you can add for:

* Selected text
* Links
* Images
* Pages
* Specific contexts (editable fields, videos, etc.)

---

## 🔧 Mini Project: Translate Selected Text with Right Click

We’ll add a context menu item that appears when the user selects text. When clicked, it opens Google Translate with that text.

---

### 1️⃣ Update `manifest.json`

Add `"contextMenus"` to permissions:

```json
"permissions": ["contextMenus"]
```

Add this to your `"background"` section if not already present:

```json
"background": {
  "service_worker": "background.js"
}
```

---

### 2️⃣ `background.js`

```javascript
chrome.runtime.onInstalled.addListener(() => {
  chrome.contextMenus.create({
    id: "translateText",
    title: "Translate selected text",
    contexts: ["selection"]
  });
});

chrome.contextMenus.onClicked.addListener((info, tab) => {
  if (info.menuItemId === "translateText" && info.selectionText) {
    const query = encodeURIComponent(info.selectionText);
    const url = `https://translate.google.com/?sl=auto&tl=en&text=${query}`;
    chrome.tabs.create({ url });
  }
});
```

---

### ✅ Test It:

1. Right-click on selected text on any webpage.
2. You’ll see a new option: **“Translate selected text”**
3. Clicking it opens Google Translate in a new tab with the selected text pre-filled.

---

### 🧠 You Can Also:

* Use `contexts: ["image"]` to do something when a user right-clicks an image.
* Add multiple menu options using different `id`s.
* Store user preferences about context menu behavior using `chrome.storage`.

---
Let’s continue to the next exciting topic: **Alarms & Timers** — ideal for building reminders, periodic actions, or scheduled background tasks in Chrome extensions.

---

## ⏰ Alarms & Timers in Chrome Extensions

### **Why Use It?**

Chrome's `alarms` API lets you schedule events:

* Repeating reminders (e.g., "drink water every 30 min")
* Time-based triggers (e.g., auto save every 5 mins)
* Long-lived background jobs (only possible in background/service workers)

---

### 🔧 Mini Project: Set a Custom Reminder Every X Minutes

---

### 1️⃣ Add `"alarms"` to your permissions in `manifest.json`:

```json
"permissions": ["alarms", "storage", "notifications"]
```

---

### 2️⃣ In `background.js`

```javascript
// Listen for alarm triggers
chrome.alarms.onAlarm.addListener((alarm) => {
  if (alarm.name === "productivityReminder") {
    chrome.notifications.create({
      type: "basic",
      iconUrl: "icons/icon48.png",
      title: "Take a Short Break",
      message: "Stand up, stretch, or grab water!"
    });
  }
});

// On install, set default alarm
chrome.runtime.onInstalled.addListener(() => {
  chrome.alarms.create("productivityReminder", {
    delayInMinutes: 1,      // First fire
    periodInMinutes: 30     // Repeat every 30 minutes
  });
});
```

---

### 3️⃣ Optional: Let user customize the interval from `options.html`

#### `options.html`

```html
<h2>Reminder Settings</h2>
<input type="number" id="interval" placeholder="Enter minutes" min="1" />
<button id="setReminder">Set Reminder</button>
```

#### `options.js`

```javascript
document.getElementById("setReminder").addEventListener("click", () => {
  const mins = parseInt(document.getElementById("interval").value);
  if (isNaN(mins) || mins < 1) return alert("Enter a valid number!");

  chrome.storage.sync.set({ interval: mins }, () => {
    chrome.alarms.clear("productivityReminder", () => {
      chrome.alarms.create("productivityReminder", {
        delayInMinutes: mins,
        periodInMinutes: mins
      });
      alert(`Reminder set every ${mins} minutes`);
    });
  });
});
```

---

### ✅ Test It:

1. Load the extension
2. Open the Options page and set an interval (e.g., 1 minute for testing)
3. Wait for the notification to appear

---

### 🧠 Pro Tips:

* Alarms work only when the **background script is alive** (in Manifest V3, that’s the **service worker**)
* You can check upcoming alarms with: `chrome.alarms.getAll(callback)`
* Use alarms for **data syncing**, **notifications**, or **regular cleanup jobs**

---


Awesome! Let’s now explore **Using External APIs** — a key skill to bring dynamic, real-world data into your Chrome extension.

---

## 🌐 Using External APIs in Chrome Extensions

### **Why This Matters**

You can:

* Fetch live data (weather, quotes, news, prices)
* Enhance features (language translation, AI tools)
* Add dynamic content to your popup or injected scripts

Chrome extensions use the `fetch()` API just like normal JS. The only caveat: if you're accessing a third-party API, you might need to add **CORS permissions** or use a **proxy** if the API doesn’t support CORS.

---

## 🔧 Mini Project: Fetch a Random Programming Quote in the Popup

---

### 1️⃣ Use This API (no API key required):

📘 \[Programming Quotes API]
`https://programming-quotes-api.vercel.app/api/random`

---

### 2️⃣ Update `popup.html`

```html
<h3>Quote of the Moment</h3>
<blockquote id="quote">Loading...</blockquote>
<small id="author"></small>
<button id="newQuote">New Quote</button>
```

---

### 3️⃣ Update `popup.js`

```javascript
async function fetchQuote() {
  try {
    const res = await fetch("https://programming-quotes-api.vercel.app/api/random");
    const data = await res.json();
    document.getElementById("quote").textContent = `"${data.en}"`;
    document.getElementById("author").textContent = `— ${data.author}`;
  } catch (err) {
    document.getElementById("quote").textContent = "Failed to load quote.";
    console.error(err);
  }
}

document.getElementById("newQuote").addEventListener("click", fetchQuote);
document.addEventListener("DOMContentLoaded", fetchQuote);
```

---

### 4️⃣ No Permissions Required for This API

But if you use other APIs, you may need to update `manifest.json`:

```json
"host_permissions": [
  "https://api.example.com/*"
]
```

---

### ✅ Test It:

1. Click your extension
2. A quote should appear
3. Click “New Quote” to load another one

---

### 🧠 Bonus Ideas:

* Fetch weather using OpenWeather API
* Add currency converter using exchangerate.host
* Use GPT or Google Translate for language features (requires API keys)

---
Great! Since you answered "yes", I’ll continue with the **next topic: Chrome Identity API**, which lets your extension authenticate users using their Google accounts or OAuth-based services (like GitHub, Discord, etc.).

---

## 🔐 Chrome Identity API (User Authentication)

### **Why Use It?**

If your extension has personalized features or needs to:

* Authenticate users with Google
* Call APIs that require OAuth (e.g., Gmail, Calendar, GitHub)
* Manage user sessions without forcing manual logins

---

### ⚠️ Requirements:

* You **must publish** your extension (even privately) in the Chrome Web Store.
* You need to configure **OAuth2 client credentials** in Google Cloud Console or a third-party service.

---

## 🔧 Mini Demo: Google OAuth Sign-In Flow (Read Profile Info)

---

### 1️⃣ Add Permissions in `manifest.json`

```json
"oauth2": {
  "client_id": "YOUR_CLIENT_ID.apps.googleusercontent.com",
  "scopes": ["https://www.googleapis.com/auth/userinfo.profile"]
},
"permissions": ["identity"]
```

> You’ll get `client_id` from [Google Cloud Console → OAuth 2.0 Client IDs](https://console.cloud.google.com/apis/credentials)

---

### 2️⃣ Add Sign-In Button to `popup.html`

```html
<button id="signin">Sign In with Google</button>
<pre id="profile"></pre>
```

---

### 3️⃣ Add Logic in `popup.js`

```javascript
document.getElementById("signin").addEventListener("click", () => {
  chrome.identity.getAuthToken({ interactive: true }, (token) => {
    if (chrome.runtime.lastError || !token) {
      console.error("Auth failed:", chrome.runtime.lastError);
      return;
    }

    fetch("https://www.googleapis.com/oauth2/v1/userinfo?alt=json", {
      headers: {
        Authorization: `Bearer ${token}`
      }
    })
    .then(res => res.json())
    .then(data => {
      document.getElementById("profile").textContent = JSON.stringify(data, null, 2);
    });
  });
});
```

---

### ✅ Test Flow (After Publishing or Using Developer Mode w/ Credentials):

1. Load your extension
2. Click "Sign in with Google"
3. Grant access
4. View your Google profile info printed

---

### 🧠 Real-World Use Cases:

* Sync notes, preferences, or settings for signed-in users
* Use Google Drive or Gmail API
* Authenticate users before accessing a premium feature

---
Perfect — let’s wrap up your Chrome Extension journey with the **final topic: Publishing to the Chrome Web Store**.

---

## 🌍 Publishing a Chrome Extension to the Web Store

### **Step-by-Step Guide**

---

### ✅ 1. **Prepare Your Project**

* ✅ Make sure your extension folder contains:

  * `manifest.json`
  * All referenced JS, CSS, HTML, images
  * Icons (16x16, 48x48, 128x128)
  * No console errors or broken links
* ✅ Remove test/debug logs
* ✅ Minify files if needed (optional but recommended)

---

### ✅ 2. **Create a Developer Account**

* Go to: [https://chromewebstore.google.com](https://chromewebstore.google.com)
* Click **"Publish in Chrome Web Store"**
* Pay the **one-time \$5 USD registration fee**

---

### ✅ 3. **Zip Your Extension**

* Zip the contents *inside* the extension folder (not the folder itself):

```
productivity-booster/
├── manifest.json
├── popup.html
├── popup.js
├── background.js
├── content.js
├── options.html
...
```

→ Select all files inside `productivity-booster` and compress them.

---

### ✅ 4. **Upload to Developer Dashboard**

* Go to: [Chrome Web Store Developer Dashboard](https://chromewebstore.google.com/u/0/developer/dashboard)
* Click **"Add a new item"**
* Upload the `.zip` file

---

### ✅ 5. **Fill Out Store Listing**

* Title & Description
* Screenshots (at least one 1280x800 image)
* Icon (128x128 PNG)
* Short summary (max 132 characters)
* Full description
* Category (e.g., Productivity)
* Site link (optional)
* Support email and privacy policy link

---

### ✅ 6. **Privacy & Permissions**

* Clearly state what data your extension accesses and why
* Provide a privacy policy if using:

  * `identity`
  * `storage.sync`
  * Host permissions
  * Analytics

---

### ✅ 7. **Submit for Review**

* Click **"Publish"**
* Your extension goes into **Pending Review**
* Review time is usually **2 to 7 days**

---

### 🔒 Bonus: Publishing for Private Use Only

If you only want the extension for personal/team use:

* Set visibility to **"Unlisted"** or **"Private"** (linked to Google Workspace users only)

---

### 🧠 Power Tip:

Once published, your users can install/update it from the Web Store and Chrome will auto-update their extensions when you push a new version.

---

🎉 **You’ve now completed all core Chrome Extension topics**!

Would you like me to help you **start a real-world extension project** (like a productivity tool, price tracker, or quick translator) using everything you’ve learned?


Awesome choice! Let’s build a **Productivity Chrome Extension** that helps users stay focused. We’ll call it:

> **FocusMate** — *A Chrome extension that reminds users to take breaks, hides distractions, and tracks time on websites.*

---

## ✅ Feature Plan (What We’ll Build)

| Feature            | Description                                                |
| ------------------ | ---------------------------------------------------------- |
| 🧘 Focus Mode      | Hides images and distracting elements on websites          |
| ⏰ Break Reminder   | Sends notifications every X minutes                        |
| 📊 Website Tracker | Tracks how long the user spends on each site               |
| ⚙️ Options Page    | Allows customizing reminder interval & excluded sites      |
| 📋 Popup UI        | Shows current session time and a “Start Focus Mode” button |

We’ll build it in **phases** so you can test as you go.

---

## 🚀 Phase 1: Set Up FocusMate with Popup + Focus Mode

### 📁 Folder Structure

```
focusmate/
├── manifest.json
├── popup.html
├── popup.js
├── content.js
├── background.js
├── styles.css
├── icons/
│   ├── icon16.png
│   ├── icon48.png
│   └── icon128.png
```

---

### 1️⃣ `manifest.json`

```json
{
  "manifest_version": 3,
  "name": "FocusMate",
  "version": "1.0",
  "description": "Block distractions and get more done.",
  "action": {
    "default_popup": "popup.html",
    "default_icon": {
      "16": "icons/icon16.png",
      "48": "icons/icon48.png",
      "128": "icons/icon128.png"
    }
  },
  "background": {
    "service_worker": "background.js"
  },
  "permissions": ["storage", "tabs", "scripting", "alarms", "notifications"],
  "host_permissions": ["<all_urls>"],
  "content_scripts": [
    {
      "matches": ["<all_urls>"],
      "js": ["content.js"],
      "run_at": "document_idle"
    }
  ],
  "icons": {
    "16": "icons/icon16.png",
    "48": "icons/icon48.png",
    "128": "icons/icon128.png"
  }
}
```

---

### 2️⃣ `popup.html`

```html
<!DOCTYPE html>
<html>
<head>
  <title>FocusMate</title>
  <link rel="stylesheet" href="styles.css" />
</head>
<body>
  <h2>FocusMate</h2>
  <button id="toggleFocus">Activate Focus Mode</button>
</body>
<script src="popup.js"></script>
</html>
```

---

### 3️⃣ `popup.js`

```javascript
document.getElementById("toggleFocus").addEventListener("click", () => {
  chrome.tabs.query({ active: true, currentWindow: true }, (tabs) => {
    chrome.tabs.sendMessage(tabs[0].id, { action: "toggleFocus" });
  });
});
```

---

### 4️⃣ `content.js`

```javascript
let focused = false;

function toggleFocusMode() {
  focused = !focused;
  document.querySelectorAll("img, video, iframe, aside, .ad, [class*='ad']").forEach(el => {
    el.style.display = focused ? "none" : "";
  });
}

chrome.runtime.onMessage.addListener((msg, sender, sendResponse) => {
  if (msg.action === "toggleFocus") {
    toggleFocusMode();
  }
});
```

---

### 5️⃣ `styles.css`

```css
body {
  font-family: Arial, sans-serif;
  padding: 10px;
  width: 220px;
}

button {
  background-color: #4CAF50;
  color: white;
  padding: 10px;
  border: none;
  width: 100%;
  cursor: pointer;
  font-size: 16px;
}
```

---

### ✅ Test This:

1. Load the extension in `chrome://extensions`
2. Open any website (e.g., news site)
3. Click the extension → click "Activate Focus Mode"
4. Images, videos, and ads should disappear!

---

Once you're happy with Phase 1, I’ll guide you through **Phase 2: Adding Break Reminders with Alarms & Notifications**.

Would you like to proceed to Phase 2 now?

Great — let’s jump into **Phase 2: Adding Break Reminders** using Chrome’s `alarms` and `notifications` APIs.

---

## 🧘 Phase 2: Break Reminder Notifications

### 🧠 Goal:

Every X minutes (e.g., 30), the extension sends a Chrome notification:

> *"Take a short break and breathe. Your mind will thank you."*

We'll also allow the user to configure the interval from an **Options page**.

---

## ✅ Step-by-Step Guide

### 1️⃣ Add the `options.html` and `options.js` files

#### 📄 `options.html`

```html
<!DOCTYPE html>
<html>
<head>
  <title>FocusMate Settings</title>
</head>
<body>
  <h2>Reminder Settings</h2>
  <label for="interval">Interval (minutes):</label>
  <input type="number" id="interval" min="1" />
  <button id="save">Save</button>
  <p id="status"></p>
  <script src="options.js"></script>
</body>
</html>
```

#### 📄 `options.js`

```javascript
document.addEventListener("DOMContentLoaded", () => {
  chrome.storage.sync.get("interval", (data) => {
    document.getElementById("interval").value = data.interval || 30;
  });

  document.getElementById("save").addEventListener("click", () => {
    const interval = parseInt(document.getElementById("interval").value);
    if (isNaN(interval) || interval < 1) return alert("Please enter a valid number");

    chrome.storage.sync.set({ interval }, () => {
      chrome.runtime.sendMessage({ action: "setAlarm", interval });
      document.getElementById("status").textContent = `Saved! Reminder set every ${interval} minutes.`;
    });
  });
});
```

---

### 2️⃣ Update `manifest.json` to add:

```json
"options_page": "options.html",
```

And make sure `"alarms"` and `"notifications"` are in `"permissions"`.

---

### 3️⃣ Update `background.js`

```javascript
// Show notification when alarm triggers
chrome.alarms.onAlarm.addListener((alarm) => {
  if (alarm.name === "breakReminder") {
    chrome.notifications.create({
      type: "basic",
      iconUrl: "icons/icon48.png",
      title: "Time to Take a Break",
      message: "Step away, stretch, and rest your eyes!",
      priority: 2
    });
  }
});

// Set default alarm on install
chrome.runtime.onInstalled.addListener(() => {
  chrome.storage.sync.get("interval", (data) => {
    const interval = data.interval || 30;
    chrome.alarms.create("breakReminder", {
      delayInMinutes: interval,
      periodInMinutes: interval
    });
  });
});

// Handle manual updates from options page
chrome.runtime.onMessage.addListener((msg, sender, sendResponse) => {
  if (msg.action === "setAlarm" && msg.interval) {
    chrome.alarms.clear("breakReminder", () => {
      chrome.alarms.create("breakReminder", {
        delayInMinutes: msg.interval,
        periodInMinutes: msg.interval
      });
    });
  }
});
```

---

### ✅ How to Test It

1. Load your extension in `chrome://extensions`
2. Open the options page (right-click extension → Options)
3. Set the interval to **1 minute** for quick testing
4. Wait 60 seconds — a notification should appear
5. Set longer intervals (15–30 min) for real use

---

Would you like to continue to **Phase 3: Website Time Tracker**, where we record how long a user stays on each domain and show a daily summary?

Awesome — let's move into **Phase 3: Website Time Tracker**!

---

## ⏱️ Phase 3: Track Time Spent on Each Website

### 🧠 Goal:

* Record how much time a user spends on each domain
* Store it daily using `chrome.storage`
* Display the summary in a popup or options page later

---

## ✅ Feature Overview

| Feature                  | Function                              |
| ------------------------ | ------------------------------------- |
| Detect active tab domain | Track only one domain at a time       |
| Track active time        | Only while Chrome is focused          |
| Store data per day       | `storage.local` keeps time per domain |
| Show summary             | We’ll build this next phase           |

---

## 🛠 Implementation

### 1️⃣ Update `background.js`

Add this logic **below** your existing alarm code.

```javascript
let currentDomain = "";
let startTime = Date.now();

// Called when active tab changes or browser is focused
function updateActiveTab() {
  chrome.tabs.query({ active: true, currentWindow: true }, (tabs) => {
    if (!tabs[0] || !tabs[0].url) return;
    const url = new URL(tabs[0].url);
    const domain = url.hostname;

    if (domain !== currentDomain) {
      recordTime();
      currentDomain = domain;
      startTime = Date.now();
    }
  });
}

// Record time spent on previous site
function recordTime() {
  if (!currentDomain) return;

  const now = Date.now();
  const timeSpent = Math.round((now - startTime) / 1000); // seconds

  const today = new Date().toISOString().split("T")[0];

  chrome.storage.local.get([today], (data) => {
    const stats = data[today] || {};
    stats[currentDomain] = (stats[currentDomain] || 0) + timeSpent;
    chrome.storage.local.set({ [today]: stats });
  });
}

// Events to track tab changes
chrome.tabs.onActivated.addListener(updateActiveTab);
chrome.tabs.onUpdated.addListener((tabId, changeInfo, tab) => {
  if (changeInfo.status === "complete") updateActiveTab();
});
chrome.windows.onFocusChanged.addListener(() => updateActiveTab());

// Record time when browser is closed or extension is unloaded
chrome.runtime.onSuspend.addListener(() => {
  recordTime();
});
```

---

### 2️⃣ What We Just Did:

* Tracked `startTime` per site
* On tab/window changes, we update the active domain
* On exit (`onSuspend`), we save the last session
* Stored daily stats in `chrome.storage.local` under key = date

---

### ✅ To Test:

1. Load the extension
2. Browse a couple of sites for a few seconds
3. Go to `chrome://extensions`, click your extension's background page → Console
4. Run:

```js
chrome.storage.local.get(null, console.log)
```

You’ll see something like:

```json
{
  "2025-06-25": {
    "youtube.com": 64,
    "github.com": 31
  }
}
```

---

### 🧠 Notes:

* This approach is **lightweight** and doesn’t track every second — just entry/exit
* Perfect for building **daily summary dashboards**

---

Would you like to continue to **Phase 4: Show Time Summary in the Popup** or **Export stats to CSV** next?
Great! Let’s move to **Phase 4: Show Time Summary in the Popup** — so users can instantly see how much time they’ve spent on websites today.

---

## 📊 Phase 4: Time Summary in the Popup

### 🧠 Goal:

* Show a list of domains and time spent today
* Display it in a readable format inside your extension popup

---

### ✅ Step-by-Step Implementation

---

### 1️⃣ Update `popup.html`

Add this below your existing button:

```html
<hr />
<h3>Today's Focus Stats</h3>
<ul id="stats"></ul>
```

---

### 2️⃣ Update `popup.js`

Below your event listener, add:

```javascript
function formatSeconds(seconds) {
  const mins = Math.floor(seconds / 60);
  const secs = seconds % 60;
  return `${mins}m ${secs}s`;
}

function loadFocusStats() {
  const today = new Date().toISOString().split("T")[0];
  chrome.storage.local.get([today], (data) => {
    const stats = data[today];
    const list = document.getElementById("stats");
    list.innerHTML = "";

    if (!stats) {
      list.innerHTML = "<li>No data for today yet.</li>";
      return;
    }

    // Sort by time descending
    const sorted = Object.entries(stats).sort((a, b) => b[1] - a[1]);

    for (const [domain, seconds] of sorted) {
      const li = document.createElement("li");
      li.textContent = `${domain}: ${formatSeconds(seconds)}`;
      list.appendChild(li);
    }
  });
}

document.addEventListener("DOMContentLoaded", loadFocusStats);
```

---

### ✅ Result:

When you click your extension icon, you’ll see:

```
Today's Focus Stats
--------------------
github.com: 4m 20s
youtube.com: 1m 10s
```

---

### 🧠 Optional Enhancements (Next Phase Ideas):

* Add a **Reset Stats** button
* Add an **Export CSV** option
* Show a **bar chart** using Chart.js or similar
