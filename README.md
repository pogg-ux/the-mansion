# The Mansion

A local-first, multi-persona AI platform that runs open-source LLMs entirely on my own hardware. No cloud dependency, no subscriptions, no telemetry.

I'm an industrial machinery field service technician (12 years). I started The Mansion in April 2026 as a traveling AI assistant for myself, and it grew into a test bed for running, routing, and evaluating local models. The source is private while it's under active development; this repo documents the architecture, how I evaluate it, and where it's going.

## Architecture

```
React/Vite UI  ──►  FastAPI backend  ──►  llama.cpp (router / supervisor mode)
                         │                      └─ one model resident at a time,
                         │                         load/evict on demand
                         ├──►  ChromaDB (RAG collections, per-persona access)
                         ├──►  Whisper (speech-to-text)
                         └──►  Scheduled jobs (overnight story generation)
PyQt6 desktop tools ──► live RAM / VRAM / GPU monitor
```

| Layer | What it does |
|---|---|
| Model serving | llama.cpp in router/supervisor mode. Each model runs as a child process on a dynamic port. Hard policy: one model resident at a time; ~95% of VRAM reclaimed on sleep. |
| Routing | A single-source-of-truth `MODEL_ROUTING` config maps seven personas to models. |
| Retrieval | ChromaDB knowledge bases with per-persona collection access. |
| Voice | Whisper speech-to-text for hands-free use. |
| App | FastAPI backend, React/Vite frontend, PyQt6 desktop launchers and a hardware monitor (psutil + pynvml). |

## Evaluating reliability

The hardest problem so far isn't wiring, it's **confabulation**: personas confidently inventing facts, memories, or workflows.

- I probe each persona with fixed test questions and count fabrications across repeated runs.
- For one persona, I traced fabrication to a specific block of its system prompt that described scheduled jobs. Removing it cut the prompt from 11,495 to 8,866 characters and the fabrication rate on the probe from **4 of 4 runs to 1 of 3**. Reduced, not cured.
- Lesson: prompt prose can model invention as good behavior. Some fabrication is dispositional to the persona's written character, not just the model.

**Next:** an in-domain test battery per task category, used to choose models *before* tuning personalities ("models before personalities"), plus an audit gate so generated content is checked before it's stored.

## How I build

I architect the system and direct AI coding agents through written work orders with explicit guardrails:

- Back up before any edit
- Verify state on disk before patching ("don't patch a ghost")
- Stop on any discrepancy instead of improvising
- No new silent error handlers

A full codebase audit surfaced 12 issues (routing-key mismatches, name-based process kills, an open CORS policy on an unauthenticated backend, duplicate file watchers), which I turned into a prioritized fix plan.

## Roadmap

1. Model-selection methodology and in-domain test battery
2. Hallucination-audit gate for stored content
3. Unify the streaming and non-streaming chat paths
4. **Field-service assistant:** a stripped-down build of this stack to help technicians identify components from a photo, walk a diagnosis, and log work orders hands-free

## Contact

[LinkedIn](https://www.linkedin.com/in/philip-ogg-77b84a446/)
