# snow-cat

An animated snow-cat companion that listens via VAD, thinks via an LLM routed through OpenRouter, talks back via in-browser Kokoro ONNX voice synthesis, and reacts visually through animated states (idle, listening, thinking, speaking, happy).

Built on **Turborepo + bun workspaces + Next.js 16**.

---

## Prerequisites

- **bun** >= 1.3.14 ([install](https://bun.sh/docs/installation))
- An **OpenRouter API key** ([get one](https://openrouter.ai/keys))

---

## Getting Started

```sh
# Install dependencies
bun install

# Start dev (all apps + packages in watch mode)
turbo dev

# Or just the web app
turbo dev --filter=web
```

---

## Monorepo Layout

```
snow-cat/
├── apps/
│   ├── web/              # Next.js 16 app — the only deployable app
│   └── docs/             # Starter scaffold (can be removed)
├── packages/
│   ├── types/            # @snow-cat/types — shared TS types (chat, cat state, TTS contract)
│   ├── store/            # @snow-cat/store — Zustand cat state store
│   ├── vad/              # @snow-cat/vad — VAD manager + useVAD hook
│   ├── tts/              # @snow-cat/tts — Kokoro ONNX + WebSpeech TTS engines
│   ├── ai/               # @snow-cat/ai — OpenRouter client + persona prompt (server-only)
│   ├── ui/               # @snow-cat/ui — SnowCat, ParticleField, Visualizer, etc.
│   ├── eslint-config/    # @repo/eslint-config — shared ESLint configs
│   └── typescript-config/# @repo/typescript-config — shared tsconfigs
├── turbo.json
└── package.json
```

---

## Environment

Create `apps/web/.env.local`:

```env
OPENROUTER_API_KEY=sk-or-v1-your-key-here
```

This is the only env var — it stays server-side (no `NEXT_PUBLIC_` prefix) and is never exposed to the browser.

---

## Available Scripts

| Command | Description |
|---|---|
| `turbo dev` | Run all packages in dev mode |
| `turbo build` | Build all packages |
| `turbo lint` | Lint all packages |
| `turbo check-types` | Type-check all packages |

---

## Package Reference

| Package | Scope | Description |
|---|---|---|
| `@snow-cat/types` | shared | Chat types, cat state union, TTS engine interface, persona config |
| `@snow-cat/store` | client | Zustand store for cat state, transcript, reply, VAD intensity |
| `@snow-cat/vad` | client-only | Silero VAD wrapper (`@ricky0123/vad-web`), `useVAD` hook, PCM→WAV utility |
| `@snow-cat/tts` | client-only | `WebSpeechTTS` (fallback), `KokoroTTS` (ONNX), `VoiceCloner`, pitch-shifting |
| `@snow-cat/ai` | server-only | OpenRouter `chat/completions` client, system prompt, model fallback logic |
| `@snow-cat/ui` | client | Animated cat, particle field, audio visualizer, chat bubble, control panel |
| `@repo/eslint-config` | shared | ESLint flat configs (base, next-js, react-internal) |
| `@repo/typescript-config` | shared | tsconfig presets (base, nextjs, react-library) |

---

## Architecture Notes

- **Server-only**: `@snow-cat/ai` — keeps `OPENROUTER_API_KEY` out of the browser bundle. Only imported by `apps/web/app/api/chat/route.ts`.
- **Client-only**: `@snow-cat/vad`, `@snow-cat/tts`, `@snow-cat/ui` — keep ONNX runtime and mic access out of the server bundle.
- **COOP/COEP headers**: required in `next.config.js` for Kokoro's ONNX `SharedArrayBuffer` usage. Compatible with Vercel; not compatible with GitHub Pages.

---

Originally scaffolded from `create-turbo` (bun + Next.js).
