# rl.english

An interactive experiment in evolving text-producing agents, with a Python simulation backend and a live web dashboard.

## Overview

The project explores how agents improve through mutation, selection, imitation, neural text models, scoring feedback, and shared pattern memory. A FastAPI backend runs the simulation and streams updates to a React frontend.

This is a research prototype. It combines locally defined vocabulary, pretrained models, and optional GPT-assisted scoring; it is not a demonstration of agents discovering English entirely from scratch or a benchmarked claim of language fluency.

## Features

- Multiple agent implementations, including character/word neural agents and word-genome agents.
- Genetic evolution with mutation, crossover, elitism, and exploration schedules.
- Curriculum phases and shared memory of discovered patterns.
- Live population, text, score, and conversation views over WebSockets.
- Controls to start, stop, reset, configure, and switch simulation models.
- Saved interesting conversations, best-agent records, and pattern/checkpoint support.
- Optional GPT-assisted scoring and chat.

## Architecture

1. The frontend calls the backend REST API for state and simulation controls.
2. Backend simulation loops generate text with the selected agent type.
3. The scorer evaluates output using local rules and enabled local/API models.
4. Evolution selects agents and applies mutation/crossover or imitation updates.
5. Pattern memory and selected results are recorded locally.
6. `/ws` streams simulation updates back to the dashboard.

The scorer includes optional `distilgpt2` perplexity and `all-MiniLM-L6-v2` relevance models. GPT-assisted paths use OpenAI. The committed common-word lists and pretrained models supply prior linguistic knowledge.

## Tech stack

| Layer | Implementation |
| --- | --- |
| Backend | Python, FastAPI, Uvicorn, asyncio, WebSockets |
| Simulation | NumPy, PyTorch, Transformers, Sentence Transformers |
| API scoring/chat | OpenAI |
| Frontend | React 18, TypeScript, Vite 5 |
| Charts and motion | Recharts, Framer Motion |
| Storage | Local JSON records and model checkpoint files |

## Project structure

- `backend/main.py` — API, WebSocket, and simulation orchestration.
- `backend/agents.py`, `backend/neural_agent.py`, `backend/word_agent.py`, and `backend/word_genome_agent.py` — agent implementations.
- `backend/scorer.py` — local and GPT-assisted evaluation.
- `backend/evolution.py` — evolution machinery.
- `backend/config.py` — simulation defaults and options.
- `backend/memory.py` — shared pattern memory.
- `backend/data/` — saved simulation records.
- `frontend/` — dashboard and backend connection hooks.

## Run locally

Prerequisites: Python compatible with the backend dependencies and Node.js/npm compatible with the frontend dependencies. Use separate terminals for backend and frontend. Create and activate a Python virtual environment before installing packages.

```bash
git clone https://github.com/anishkganesh/rl.english.git
cd rl.english/backend
python -m venv .venv
pip install -r requirements.txt
```

Create a backend `.env` for the features you want to enable:

```dotenv
OPENAI_API_KEY=your-openai-key
PORT=8000
SKIP_LOCAL_MODELS=false
```

Set `SKIP_LOCAL_MODELS=true` to skip optional local scoring-model loading. Local and GPT-assisted paths have different capabilities; an OpenAI key is needed for the GPT-dependent operations.

```bash
python main.py
```

In a second terminal, from the repository root:

```bash
cd frontend
npm ci
npm run dev
```

Open `http://localhost:3000`. The development server proxies to the backend on port 8000.

## Configuration and data

For a separately hosted backend, set `VITE_BACKEND_URL` in the frontend environment to a hostname with an optional port, such as `localhost:8000`. **Do not include `http://`, `https://`, `ws://`, or `wss://`**: the connection code constructs the protocol. Rebuild the frontend after changing Vite environment variables.

Default evolution configuration includes a population of 12, mutation rate 0.15, mutation strength 0.3, elitism 0.4, crossover 0.4, and an exploration schedule. Inspect `backend/config.py` for the authoritative defaults and model-specific settings.

First-time local model loading can download model files and use significant memory. Local output files need writable storage; persistent hosting requires a durable volume rather than an ephemeral filesystem.

## Usage

Choose an agent model, start the simulation, and inspect population output and score changes. Adjust the available configuration, compare runs, and inspect saved conversations or best agents. Scores are feedback signals defined by this experiment, rather than a general measure of linguistic competence.

API entry points include `/`, `/models`, `/state`, `/start`, `/stop`, `/reset`, `/switch-model`, `/config`, `/score-text`, `/chat`, `/saved-conversations`, `/best-agents`, `/agent/{agent_id}/viz`, and `/ws`. Per-model start/stop routes are also implemented.

## Validation

```bash
cd frontend
npm run build
```

No automated research evaluation or regression suite is configured. Check backend health, WebSocket connection, each enabled model's start/stop/reset behavior, score availability, and saved output. A successful frontend build does not establish simulation quality. This documentation review did not execute the simulation or model downloads.

## Deployment

The repository includes backend Railway configuration/Procfile and frontend Vercel configuration. Run the Python backend as a long-lived service capable of WebSockets and background simulation, and serve `frontend/dist` as the frontend build output. Configure `VITE_BACKEND_URL` for the deployed backend before building.

## Limitations

- Results depend on scoring choices, vocabulary, pretrained models, and API availability.
- API scoring consumes credits; local neural models consume compute and memory.
- The backend exposes simulation controls without a complete authentication boundary and permits broad CORS. Restrict access before running it as a shared service.
- JSON and checkpoint persistence is local to the backend instance.

## Attribution and license

Pretrained models and dependencies have their own licenses. No standalone project license file is included; this README does not grant a new license.
