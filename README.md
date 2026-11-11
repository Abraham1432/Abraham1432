<div align="center">
  <a href="https://zyrabit.com">
    <img src="https://assets.zyrabit.com/logo_zyrabit.png" alt="Zyrabit" width="120" />
  </a>
  <br />
  <img src="https://readme-typing-svg.demolab.com?font=Funnel+Display&amp;weight=600&amp;size=22&amp;duration=3500&amp;pause=1200&amp;color=6090B4&amp;center=true&amp;vCenter=true&amp;width=740&amp;lines=Founder+of+Zyrabit;Sovereign+AI+for+regulated+teams;Your+data+never+leaves.+Prove+me+wrong." alt="Founder of Zyrabit. Sovereign AI for regulated teams. Your data never leaves. Prove me wrong." />
</div>

<h2 align="center">Your data never leaves. Prove me wrong.</h2>
<p align="center"><em>Tus datos no salen de tu perímetro. Demuéstrame lo contrario.</em></p>

<p align="center">
  <a href="https://github.com/Zyrabit-tech/zyrabit-SLM/blob/main/LICENSE"><img src="https://img.shields.io/badge/License-MIT-6090B4?style=for-the-badge" alt="License: MIT" /></a>
  <img src="https://img.shields.io/badge/NVIDIA-Inception-76B900?style=for-the-badge&amp;logo=nvidia&amp;logoColor=white" alt="NVIDIA Inception member" />
  <a href="https://www.youtube.com/@Zyrabit-SLM"><img src="https://img.shields.io/badge/YouTube-Zyrabit--SLM-FF0000?style=for-the-badge&amp;logo=youtube&amp;logoColor=white" alt="Zyrabit SLM on YouTube" /></a>
</p>

I'm **Abraham Gómez**, Founder of [Zyrabit](https://zyrabit.com). I build sovereign AI infrastructure so banks, law firms and compliance teams can run private models **inside their own perimeter** — no token rent, and no data sent to a public API.

After 6+ years leading engineering in fintech, edtech and payments, this is the problem I chose to spend my time on.

---

## The 10-minute challenge

**Run [Zyrabit SLM](https://github.com/Zyrabit-tech/zyrabit-SLM) on your machine, unplug the internet, and ask a question about your own document.**

If the answer comes back, the data never left. If it doesn't, [open an issue](https://github.com/Zyrabit-tech/zyrabit-SLM/issues/new) or a [discussion](https://github.com/Zyrabit-tech/zyrabit-SLM/discussions) with a screenshot — or [star the repo](https://github.com/Zyrabit-tech/zyrabit-SLM) if it worked.

This is the open-source trial of the node (MIT). The commands below match the live [Docker Hub](https://hub.docker.com/r/zyrabitcore/zyrabit-slm) quick start and the [API reference](https://docs.zyrabit.com/docs/api-reference). The API listens on **port 8088**.

<p align="center">
  <img src="https://assets.zyrabit.com/public/images/zyrabit-CLI.png" alt="Zyrabit SLM CLI — run the open-source trial on your machine" width="880" />
</p>

### Before you start the clock

You need Docker running, and [Ollama](https://ollama.com) already serving a model. On Linux, Ollama must listen on all interfaces so the container can reach it:

```bash
OLLAMA_HOST=0.0.0.0:11434 ollama serve
ollama pull qwen2.5:0.5b
ollama pull mxbai-embed-large    # needed to ingest a document
```

`qwen2.5:0.5b` is the light option from the Docker Hub docs. If you have about 16 GB of RAM, use `qwen2.5:7b` (or `qwen2.5:1.5b` in between) and set `MODEL_NAME` to match. First-time pulls don't count toward the 10 minutes. The image itself is about 500 MB.

### 1. Run the API

```bash
mkdir -p documents db_data
```

**Linux only** — the container runs as UID `10001`:

```bash
sudo chown -R 10001:10001 documents db_data
```

**Mac (Docker Desktop):** skip the `chown`. If you already changed the folders, run `sudo chown -R $(whoami) documents db_data`.

Then pull and run. Change `my-secure-token` to your own secret. `EMBEDDING_URL` is required to ingest documents:

```bash
docker pull zyrabitcore/zyrabit-slm

docker run -d \
  --name zyrabit-api \
  --add-host=host.docker.internal:host-gateway \
  -p 8088:8080 \
  -e INFERENCE_PROVIDER=ollama \
  -e SLM_URL=http://host.docker.internal:11434 \
  -e EMBEDDING_URL=http://host.docker.internal:11434 \
  -e EMBEDDING_MODEL=mxbai-embed-large \
  -e MODEL_NAME=qwen2.5:0.5b \
  -e ZYRABIT_API_KEY_WEB=my-secure-token \
  -v ./documents:/app/document_source \
  -v ./db_data:/app/db_data \
  zyrabitcore/zyrabit-slm
```

`--add-host=host.docker.internal:host-gateway` is required on Linux and harmless on Mac.

### 2. Check that it is actually healthy

Wait about 10 seconds, then:

```bash
curl -s http://localhost:8088/v1/health
```

**Read the JSON, not only the HTTP code.** A healthy API shows `"status": "OPERATIONAL"`, with `vector-db` and `slm-engine` both `ONLINE`. The model may show `WARMING_UP` until the first question. `"status": "DEGRADED"` also returns HTTP 200.

### 3. Ingest a document of yours

Use a file you are comfortable testing with — nothing confidential. Then:

```bash
curl -X POST http://localhost:8088/v1/ingest \
  -H "Authorization: Bearer my-secure-token" \
  -F "file=@/path/to/YOUR_DOCUMENT.pdf"
```

PDF, DOCX and TXT are all fine. (`POST /v1/sources/import` is the same endpoint.)

### 4. Unplug the internet

Turn off Wi-Fi or unplug the cable. Ollama and the container are already local, so they should keep running.

### 5. Ask a question only that document can answer

```bash
curl -X POST http://localhost:8088/v1/chat \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer my-secure-token" \
  -d '{"text": "What is the retention period in my document?"}'
```

A grounded answer cites your file (for example `"decision": "hybrid-rag"` and a `sources` entry). If you see `"decision": "library-empty-guidance"`, the API is up but it has not indexed the document yet.

### 6. Show your work

Reconnect, then [open a discussion](https://github.com/Zyrabit-tech/zyrabit-SLM/discussions) or [an issue](https://github.com/Zyrabit-tech/zyrabit-SLM/issues/new) with a screenshot of the answer — or [star the repo](https://github.com/Zyrabit-tech/zyrabit-SLM) if it did what I said.

Prefer a full workspace with a web UI at `http://localhost:8080`? That's the other official path:

```bash
git clone https://github.com/Zyrabit-tech/zyrabit-SLM.git
cd zyrabit-SLM
./zyra.sh install
```

(`zyra-up.sh` still exists as a legacy shim. The current command is `./zyra.sh`.)

---

## Watch it run

<p align="center">
  <a href="https://www.youtube.com/watch?v=0BrvlX_UjRg"><img src="https://img.youtube.com/vi/0BrvlX_UjRg/hqdefault.jpg" width="560" alt="Zyrabit SLM: Local RAG on Your Own Infrastructure (Open Source)" /></a>
</p>

More on [youtube.com/@Zyrabit-SLM](https://www.youtube.com/@Zyrabit-SLM).

---

## Stack I actually use

<p align="center">
  <a href="https://skillicons.dev"><img src="https://skillicons.dev/icons?i=react,nextjs,nodejs,kubernetes,docker,aws,gcp,azure,python,linux" alt="React, Next.js, Node.js, Kubernetes, Docker, AWS, GCP, Azure, Python, Linux" /></a>
</p>

React, Next.js, Node.js, Kubernetes, Docker, and AWS / GCP / Azure. Local-first AI on small hardware when I can get away with it.

---

## Where to find me

- Website: [zyrabit.com](https://zyrabit.com)
- Gravity: [gravity.zyrabit.com](https://gravity.zyrabit.com)
- Docs: [docs.zyrabit.com](https://docs.zyrabit.com)
- Org: [github.com/Zyrabit-tech](https://github.com/Zyrabit-tech)
- Docker: [`zyrabitcore/zyrabit-slm`](https://hub.docker.com/r/zyrabitcore/zyrabit-slm)
- YouTube: [@Zyrabit-SLM](https://www.youtube.com/@Zyrabit-SLM)
- Medium: [@gomezabraham1432](https://medium.com/@gomezabraham1432)
- LinkedIn: [abraham-gomez1432](https://www.linkedin.com/in/abraham-gomez1432)
- X: [@jgomez1432](https://x.com/jgomez1432)
- Email: [contact@zyrabit.com](mailto:contact@zyrabit.com)

If you're running a meetup or a conference on local-first AI, privacy in regulated environments, or on-prem infrastructure, I'm happy to show a real stack — not just slides.

When I'm not shipping Zyrabit, I'm usually hunting for good coffee or travelling like a local.

<!-- Snake appears after the generate animation workflow runs once on master.
<p align="center">
  <img src="https://raw.githubusercontent.com/Abraham1432/Abraham1432/output/github-contribution-grid-snake.svg#gh-light-mode-only" alt="GitHub contribution snake animation" />
  <img src="https://raw.githubusercontent.com/Abraham1432/Abraham1432/output/github-contribution-grid-snake-dark.svg#gh-dark-mode-only" alt="GitHub contribution snake animation" />
</p>
-->
