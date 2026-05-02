# LangGraph Chat Frontend

A production-grade chat UI built with **Vite + React** for a LangGraph backend.
Supports standard AI streaming, Human-In-The-Loop (HITL) interrupts, and persistent
thread management — all with zero runtime internet dependencies.

---

## Features

| Feature | Details |
|---|---|
| **HITL Interrupts** | When the graph pauses, a dynamic form renders inline. Supports `text`, `textarea`, `select`, `number`, `email`, `date` fields. |
| **Thread Management** | Each conversation has a `thread_id`. "New Chat" clears state and starts fresh. |
| **Typing Indicator** | Animated dots while awaiting a backend response. |
| **Error Handling** | Dismissible error banner; failed messages show inline error state. |
| **Input Guard** | The text input is disabled while an unresolved HITL form is present. |
| **Auto-grow textarea** | Chat input grows with content, capped at 180px. |
| **No CDN / no internet** | All assets and fonts are system-local or inline SVG. |

---

## Project Structure

```
langgraph-chat/
├── index.html
├── vite.config.js          ← proxy /api → localhost:8000
├── mock-server.js          ← Express mock backend (dev only)
├── package.json
└── src/
    ├── main.jsx
    ├── App.jsx / App.css
    ├── styles/
    │   └── global.css      ← CSS variables, resets, animations
    ├── hooks/
    │   └── useChat.js      ← ALL state logic (messages, HITL, thread)
    ├── utils/
    │   └── api.js          ← Backend communication (3 endpoints)
    └── components/
        ├── Header.jsx/css        ← Logo, thread badge, New Chat button
        ├── MessageList.jsx/css   ← Scrollable chat area + empty state
        ├── Message.jsx/css       ← Bubble (user/assistant/system) + typing dots
        ├── HITLForm.jsx/css      ← Dynamic form for graph interrupts
        ├── ChatInput.jsx/css     ← Auto-grow textarea + send button
        └── ErrorBanner.jsx/css  ← Dismissible error strip
```

---

## Backend API Contract

The frontend expects these three endpoints:

### `POST /thread/new`
```json
// Response
{ "thread_id": "uuid-string" }
```

### `POST /chat`
```json
// Request
{ "thread_id": "...", "message": "user text" }

// Normal response
{ "type": "ai", "thread_id": "...", "content": "assistant reply" }

// HITL interrupt response
{
  "type": "hitl",
  "thread_id": "...",
  "fields": [
    {
      "name": "fieldKey",
      "label": "Human-readable label",
      "type": "text | textarea | select | number | email | date",
      "required": true,
      "placeholder": "optional hint",
      "options": [                          // only for type=select
        { "label": "Display", "value": "val" }
      ]
    }
  ]
}
```

### `POST /resume`
```json
// Request
{ "thread_id": "...", "fields": { "fieldKey": "value", ... } }

// Response: same shape as /chat (type: "ai" or "hitl")
```

---

## Getting Started

### 1. Install dependencies
```bash
npm install
```

### 2. Start the mock backend (optional, for dev)
```bash
# Install mock server deps first
npm install express cors uuid

node mock-server.js
# → running at http://localhost:8000
```

### 3. Start the frontend
```bash
npm run dev
# → http://localhost:5173
```

The Vite dev server proxies `/api/*` → `http://localhost:8000/*` automatically.

### 4. Production build
```bash
npm run build
```

---

## Connecting Your Real LangGraph Backend

1. Point `vite.config.js` proxy target at your backend URL
2. Ensure your backend returns the exact JSON shapes above
3. The `fields` array in HITL responses is fully dynamic — add as many fields
   as needed; the UI renders them automatically

---

## Customisation

All visual tokens live in `src/styles/global.css` as CSS variables.
Key values to change:

```css
--accent:      #eab308;   /* amber — change to your brand color */
--bg-base:     #0c0c0e;   /* main background */
--font-display: 'Georgia'; /* heading font */
```
