# how-backend-and-frontend-connect
# How Backend Connects to Frontend

The short answer: **they're usually two separate programs running on different machines, and they talk over a network using HTTP.** The frontend doesn't "call" the backend like a function — it sends a request across the internet, and the backend sends back a response.

Here's the full picture.

## 1. The fundamental model: request → response

Almost every web app works like this:

```
Browser (frontend)                     Server (backend)
   JS code          ── HTTP request ──▶   route handler
   fetch('/api/users')                     queries DB
   waits...         ◀── HTTP response ──   { "users": [...] }
   renders UI
```

The frontend needs to know **where** the backend is (a URL) and **what** it accepts (the API contract). That's the entire connection.

---

## 2. What actually happens on the wire

When your frontend calls `fetch("https://api.myapp.com/users")`:

1. **DNS** resolves `api.myapp.com` → an IP address
2. **TCP** connection opens (port 443)
3. **TLS** handshake (HTTPS encryption)
4. **HTTP request** is sent: method, path, headers, body
5. Backend receives it, runs your code, queries the database
6. **HTTP response** comes back: status code, headers, body (usually JSON)
7. Connection may be reused for the next request

All of this is hidden behind `fetch()` — but it's why **CORS, HTTPS, and DNS config** matter so much.

---

## 3. The main communication patterns

| Pattern | How it works | Best for |
|---|---|---|
| **REST API** | Frontend calls `GET /users`, `POST /users`, etc. Returns JSON. | Most apps — the default choice |
| **GraphQL** | Frontend sends one query describing exactly what it wants | Complex data, mobile apps, avoiding over-fetching |
| **WebSockets** | Persistent two-way connection; server can push anytime | Chat, live notifications, collaborative editing |
| **Server-Sent Events (SSE)** | One-way stream from server to client | Live feeds, progress updates, LLM streaming |
| **gRPC** | Binary protocol over HTTP/2 | Internal microservices, high performance |
| **tRPC** | TypeScript types shared end-to-end, no schema needed | Full-stack TypeScript apps |

---

## 4. What the frontend code looks like

```js
// REST
const res = await fetch("https://api.myapp.com/users", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    "Authorization": `Bearer ${token}`
  },
  body: JSON.stringify({ name: "Alice" })
});
const data = await res.json();
```

```js
// WebSocket
const ws = new WebSocket("wss://api.myapp.com/chat");
ws.onmessage = (event) => console.log(event.data);
```

The frontend typically stores the backend's base URL in an **environment variable** (`VITE_API_URL`, `NEXT_PUBLIC_API_URL`) so it can point at `localhost:3000` in dev and `api.myapp.com` in production.

---

## 5. The glue that makes it work

These are the things that actually bite you in practice:

- **CORS** — if frontend and backend are on different origins (`app.com` vs `api.app.com`), the backend must send `Access-Control-Allow-Origin` headers, or the browser blocks the response.
- **Authentication** — either cookies (sent automatically, need `credentials: "include"` + CORS config) or JWT in the `Authorization` header.
- **Serialization** — JSON is the standard; both sides must agree on the shape.
- **HTTPS** — required in production; also blocks mixed-content requests.
- **Error handling** — HTTP status codes (`4xx` = client problem, `5xx` = server problem) plus an error body convention.
- **Rate limiting / timeouts** — the network can fail; frontend needs retries and loading states.

---

## 6. When there's no network boundary

Sometimes the "connection" is just a function call:

- **Server-side rendering** (Next.js, Rails, Django, PHP) — the backend renders HTML with data already in it. The browser just displays it.
- **Next.js Server Actions / Remix loaders** — you write a function, the framework handles the HTTP round-trip for you.
- **Monolith with templates** (EJS, Jinja, Blade) — backend generates the full page; JS is optional.
- **Electron / Tauri desktop apps** — frontend and backend run on the same machine, often over a local IPC channel instead of HTTP.

---

## 7. Typical deployments

**Separate origins (most common for SPAs):**
```
app.myapp.com  →  static React build (Netlify/Vercel/S3)
api.myapp.com  →  Node/Go/Python server (Railway/Fly/AWS)
```
Requires CORS config.

**Same origin (simpler):**
```
myapp.com/api/*  →  proxied to backend
myapp.com/*      →  static frontend
```
No CORS issues. Often done with Nginx, a reverse proxy, or Next.js API routes.

---

## TL;DR

The frontend and backend are separate programs connected by **HTTP requests over a network**. The frontend sends a request to a URL with data and headers; the backend routes it to code, does work (usually touching a database), and returns JSON. Everything else — REST vs GraphQL, CORS, auth tokens, WebSockets — is a variation or a detail on top of that one idea.
