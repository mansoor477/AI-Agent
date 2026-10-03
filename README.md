# Agent Control Room

A responsive, free-first web AI workspace for local chat, task automation, sourced web research, document reports, channel conversations, approvals, integration health, and agent activity. Ollama is the default model provider; it runs on your computer and does not require an API key or paid service. The server owns provider credentials and local conversation data.

## Requirements and local setup

- Node.js 20 or newer and npm
- Ollama for local AI responses (optional cloud provider keys are not required)

```powershell
npm install
Copy-Item .env.example apps/server/.env
# Install Ollama separately, then pull a local model such as: ollama pull llama3.2
npm run dev
```

Open `http://localhost:5173`. The API listens on `http://localhost:3001`. The root `dev` command starts both. Alternatively use separate terminals with `npm run dev -w @agent/server` and `npm run dev -w @agent/web`.

The default settings use `MODEL_PROVIDER=ollama`, `OLLAMA_BASE_URL=http://127.0.0.1:11434`, and `MODEL=llama3.2`. Install Ollama and pull the selected model before chatting. The app does not silently fall back to a paid service. If Ollama is unavailable, the chat reports a connection/model error. Optional providers are `openai` and a configured OpenAI-compatible endpoint. Conversation history and task records are currently stored in local JSON files, not SQLite.

## Use from other devices

For devices on the same trusted Wi-Fi, run the app on a computer that stays on and find its private IPv4 address with `ipconfig`. Set `WEB_ORIGIN` in `apps/server/.env` to the exact web origin you will use, for example `http://192.168.1.25:5173`, then restart the server and web app. Allow Node.js through the private-network firewall if prompted. On another device on that Wi-Fi, open `http://192.168.1.25:5173` using the computer's address. The browser and server must remain reachable; this is not available when the host sleeps or the devices are on separate networks.

For access from different networks, deploy the web app and API to a server you control behind HTTPS and a reverse proxy. Set `PUBLIC_BASE_URL` and `WEB_ORIGIN` to the HTTPS origins and configure webhooks to that public API host. Use persistent storage for `apps/server/data`. **Do not expose this learning build's administrative APIs to the public internet:** authentication and user authorization are not implemented. Use a private VPN/network until authentication has been added. Browser local storage keeps the selected conversation ID per device, while conversations are server-side; select any thread in the dashboard to open it on another device.

## Available areas

- **Dashboard:** live conversation and pending approval counts, channel health, and recent conversations.
- **AI Chat:** streaming responses, Markdown, cancellation, text attachments, and conversation history.
- **Automations:** submit a goal, generate a bounded plan, run safe research/summarization/report steps, and monitor step results. Tasks can be paused, resumed, cancelled, or retried; interval schedules run while the server is online.
- **Documents and reports:** preview TXT, Markdown, CSV, JSON, and log files locally in the browser; save Markdown reports in the server output folder and download them.
- **Voice:** browser speech recognition where the browser supports it, plus local-device speech synthesis. Speech recognition availability and processing depend on the browser/OS and may use browser services; this is not telephone calling.
- **Coding assistant:** use chat to ask for code review or explanations. The app does not edit project files or run generated code yet.
- - **Browser Research:** Tavily search and controlled Playwright page reading with public URL/IP restrictions.
- **Approvals:** inspect and approve or reject outbound WhatsApp/call requests.
- **Agent Activity:** recorded conversation and approval events.
- **Integrations and Settings:** provider configured state without exposing secrets.

The API includes `/api/health`, `/api/v1/health`, conversation list/detail/rename/delete at `/api/v1/conversations`, chat streaming at `/api/chat`, inbox at `/api/inbox` (channel endpoint retained for backend compatibility; no inbox UI), approvals at `/api/approvals`, activity at `/api/v1/activity`, provider state at `/api/v1/integrations`, task queue and engine controls at `/api/v1/tasks`, reports and CSV export at `/api/v1/reports`, WhatsApp webhooks at `/webhooks/whatsapp`, and Twilio voice webhooks at `/webhooks/voice`.

Task automation uses `POST /api/v1/tasks` with `{ "prompt": "..." }` or an optional interval schedule such as `{ "kind": "interval", "minutes": 1440 }`. Use `GET /api/v1/tasks` to inspect progress and `POST /api/v1/tasks/:id/control` with `pause`, `resume`, `cancel`, or `retry`. Task records are persisted in `apps/server/data/tasks.json`. Schedules are local to this server process and execute only while the server is running.

## Optional providers

Copy only the credentials for services you enable into `apps/server/.env`, then restart the server. Never put secrets in frontend environment variables or commit `.env` files.

- **Research:** set `TAVILY_API_KEY`. Install Playwright's browser with `npx playwright install chromium` to enable page reading.
- **WhatsApp:** set `WHATSAPP_TOKEN`, `WHATSAPP_PHONE_NUMBER_ID`, `WHATSAPP_VERIFY_TOKEN`, and `WHATSAPP_APP_SECRET`. Configure the Meta webhook callback at `/webhooks/whatsapp`. Incoming signatures and duplicate event IDs are checked; stop/start opt-outs are recorded. Free-form response window rules are enforced; approved templates are not implemented.
- **Phone:** set `TWILIO_ACCOUNT_SID`, `TWILIO_AUTH_TOKEN`, `TWILIO_PHONE_NUMBER`, and `PUBLIC_BASE_URL`. Configure Twilio Voice to `/webhooks/voice` and status callbacks to `/webhooks/voice/status`. Calls use Gather/Say; real-time voice streaming and barge-in are not supported.
- Outbound WhatsApp messages and calls require a pending approval and an explicit approval action. WhatsApp free-form sends require a recent inbound message.

## Data and current limits

Conversations, approvals, opt-outs, call logs, provider event IDs, and tasks use local JSON storage. The repository helper serializes writes within this server process, but JSON storage is for single-instance learning and is not a multi-instance production database. SQLite is not implemented in this build. `DATABASE_URL` and `REDIS_URL` are reserved only. The task scheduler is an in-process timer, not a distributed queue; it requires the server to stay on. Task operations are limited to public web search, summarization, and Markdown reports in `apps/server/data/output`. They do not modify code, run shell commands, submit forms, or send external messages. No authentication or role authorization is implemented yet. Add authentication before public deployment. Browser page reading runs in a timed child process on the API host; production should isolate it in a restricted worker. Tavily web search requires an optional external account/key and internet; without it, research search is disabled, though direct page reading can be available for supplied public URLs.

`GET /api/v1/integrations` and the dashboard show provider readiness from server configuration. They do not validate third party account health until the provider is used.

## Development commands

```powershell
npm run dev
npm run typecheck
npm run build
```

No automated test suite is configured yet. The build and typecheck commands validate compilation only; they do not prove provider credentials, webhooks, or a public deployment work.

## Deployment notes

Build with `npm run build`. Serve `apps/web/dist` through Nginx or another HTTPS web server and proxy `/api` and `/webhooks` to the Fastify server on port 3001. Keep `apps/server/.env` readable only by the service account, use a persistent private data directory, set exact origins, and run the server under a process manager such as systemd or PM2. Keep the API behind authentication/VPN until administrative authentication and authorization have been implemented.
#   A I - A g e n t  
 