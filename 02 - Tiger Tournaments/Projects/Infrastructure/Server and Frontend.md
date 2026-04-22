---
title: Server and Frontend
aliases: [Server, Frontend, React, Vite, server.js]
tags: [infrastructure, server, frontend, react, vite, evo-draw]
created: 2026-04-21
updated: 2026-04-21
server_port: 3000
server_file: server.js
frontend_dir: frontend-v2/
---

# Server and Frontend

---

## Server

| Attribute   | Value                            |
| ----------- | -------------------------------- |
| Runtime     | Node.js                         |
| Entry point | `server.js`                      |
| Port        | 3000                            |
| Serves      | API endpoints + static files from `dist/` |

The server is a plain Node.js HTTP server (no Express) that handles both API routes and static file serving for the built frontend.

---

## Frontend

| Attribute   | Value                                    |
| ----------- | ---------------------------------------- |
| Framework   | React + Vite                             |
| Directory   | `frontend-v2/`                           |
| Branding    | "Evo Draw"                               |
| Colors      | Emerald-to-cyan gradient (`#34d399` to `#06b6d4`) |

### Build Process

```bash
cd frontend-v2 && npm run build && cd .. && rm -rf dist && cp -r frontend-v2/dist .
```

> [!warning] Build Overwrites dist/
> The build process deletes the existing `dist/` directory and replaces it with a fresh copy from `frontend-v2/dist`. Make sure the server is restarted after building to serve the updated files.

---

## Key API Endpoints

| Endpoint                                        | Method | Purpose                      |
| ------------------------------------------------ | ------ | ---------------------------- |
| `/api/managed-events/:slug`                      | GET    | Event metadata               |
| `/api/managed-events/:slug/generate-brackets`    | POST   | Generate tournament brackets |
| `/api/managed-events/:slug/team-history`         | GET    | Single team game history     |
| `/api/managed-events/:slug/all-team-history`     | GET    | All teams game history       |
| `/api/h2h-violations`                            | GET    | H2H rank violations          |

All API endpoints that query games include `WHERE is_hidden = false` — see [[Database Schema]] and [[Dedup Strategy]].

---

## Branding

The "Evo Draw" brand uses an **emerald-to-cyan gradient**:
- Emerald: `#34d399`
- Cyan: `#06b6d4`
- Applied to headers, buttons, and key UI elements
- Clean, modern design suitable for the youth soccer audience

---

## Development Workflow

1. Edit frontend code in `frontend-v2/`
2. Build: `cd frontend-v2 && npm run build`
3. Deploy: `cd .. && rm -rf dist && cp -r frontend-v2/dist .`
4. Restart server: `node server.js`
5. Access at `http://localhost:3000`

---

## Related Notes
- [[Database Schema]] — tables queried by the API
- [[Dallas Spring Showcase]] — primary managed event served
- [[Ranking Engine]] — produces data served via rankings API
- [[Evo Draw Home]] — platform overview with GitHub link
