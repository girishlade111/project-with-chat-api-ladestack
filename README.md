# LinkedIn Network Analyzer (AI Chat)

**Chat with your LinkedIn network.** Upload a CSV export of your LinkedIn connections and ask an AI assistant questions about your professional network — find warm intros, group contacts by company or theme, and spot under-leveraged relationships.

## What it does

1. **Upload** your LinkedIn "Connections.csv" export (the file LinkedIn gives you via Settings → Data Privacy → Get a copy of your data)
2. The app parses names, titles, companies, emails, profile URLs, and connection dates (`/api/upload` with a robust LinkedIn CSV parser)
3. Contacts are stored in Vercel KV and a network ID is returned to the browser
4. **Chat** with a GPT-4o-powered assistant (Vercel AI SDK, streaming responses) that knows your roster and answers questions like:
   - "How many unique companies are in my network?"
   - "Who works at Google? Suggest a warm intro path."
   - "Group my contacts by theme and point out who I'm under-leveraging."
5. If the OpenAI call fails, a **built-in mock responder** still answers basic network questions so the demo keeps working.

## Features

- LinkedIn CSV upload with validation and error details
- Client-side roster parsing + server-side storage (Vercel KV)
- Streaming AI chat (`/api/chat`, Vercel AI SDK `streamText` + `gpt-4o`)
- Mock-response fallback when the OpenAI API is unavailable
- Toast notifications, alert banners, responsive card UI
- Dark / light theme toggle

## Tech stack

- **Framework:** Next.js 15 (App Router, Node runtime — requires a server)
- **UI:** React 19, Tailwind CSS 3, shadcn/ui (Radix UI), lucide-react
- **AI:** Vercel AI SDK (`ai` + `@ai-sdk/openai`), model `gpt-4o`
- **Storage:** Vercel KV (Redis) for uploaded contact rosters
- **CSV parsing:** csv-parse with a custom LinkedIn export parser (`lib/parse-linkedin-csv.ts`)

## Quick start

```bash
npm install --legacy-peer-deps

# copy and fill in your keys
cp .env.example .env   # (create this file; see variables below)

npm run dev
# open http://localhost:3000
```

## Environment variables

| Variable | Required | Purpose |
|---|---|---|
| `OPENAI_API_KEY` | Yes | OpenAI API key for the chat assistant (gpt-4o) |
| `KV_REST_API_URL` | Yes | Vercel KV REST URL (auto-set on Vercel) |
| `KV_REST_API_TOKEN` | Yes | Vercel KV REST token (auto-set on Vercel) |

`.env` files are git-ignored — never commit keys. Without `OPENAI_API_KEY`, the chat falls back to the built-in mock responder; without KV credentials, uploads cannot persist.

## Project structure

```
app/
  page.tsx              # Main UI — upload + chat (client component)
  api/chat/route.ts     # POST /api/chat — streaming AI answers over the roster
  api/upload/route.ts   # POST /api/upload — parse + store LinkedIn CSV
  layout.tsx            # Root layout
components/             # UI (shadcn/ui pieces + toast/theme providers)
lib/
  parse-linkedin-csv.ts # LinkedIn export CSV parser
  storage.ts            # Vercel KV get/set for contact rosters
  openai-mock.ts        # Mock responder fallback
  utils.ts              # Shared helpers
```

## Deployment

This app **requires a Node server** (API routes + streaming + secrets) and **cannot be statically exported** — deploy to Vercel (recommended) or another Node host:

```bash
# on Vercel, the KV env vars are attached automatically; just set OPENAI_API_KEY
vercel --prod
```

Vercel KV setup: create a KV store in the Vercel dashboard and link it to the project — `KV_REST_API_URL` / `KV_REST_API_TOKEN` are then injected automatically.

## Security notes

- Next.js pinned at **15.2.8+** (patched against CVE-2025-55182 React2Shell — 15.2.4 is affected).
- Never commit `.env` files or API keys. The mock-responder fallback exists for demos; real chat needs a server-side `OPENAI_API_KEY`.

---

Built by Girish Lade — https://ladestack.in
