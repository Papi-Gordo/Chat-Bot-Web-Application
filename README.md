# Self-Hosted AI Chatbot on Ubuntu Server

A home-lab project documenting how I connected a local AI model to a browser-based chat interface, then made that interface accessible on the public internet over HTTPS.

I run **Ollama directly on Ubuntu as a systemd service**, while **Open WebUI runs in a Docker container with host networking**. Tailscale Funnel provides public HTTPS access to the browser interface without traditional router port forwarding. The goal of this repository is to document the configuration I actually investigated, including what I confirmed, what remains unverified, and what I learned while troubleshooting it.

> **Configuration snapshot:** Investigated through **November 15, 2026**. This repository is sanitized documentation, **not a backup of the running server**. Some configuration details still need to be checked. It is not an instruction to redeploy the live host.

## Architecture

```text
                       Public browser
                             |
                  Tailscale Funnel (HTTPS)
                             |
                  Open WebUI web interface
                   Docker: network=host
                             |
                  http://127.0.0.1:11434
                             |
                    Ollama API on Ubuntu
                    native systemd service
                             |
                       qwen3:1.7b
```

The **Ollama API** is confirmed to listen on IPv4 loopback (`127.0.0.1:11434`). Open WebUI uses `OLLAMA_BASE_URL` to connect to it. Docker host networking allows a container to use the host's loopback address. The **Open WebUI listener itself has not yet been independently audited for LAN exposure**. Funnel makes the web interface publicly reachable; authentication and user permissions must be enforced by Open WebUI.

## Verified configuration

| Component | What I observed |
| --- | --- |
| Host | Ubuntu Server (hostname deliberately omitted) |
| Ollama | Native installation; version `0.40.16` |
| Ollama service | `/etc/systemd/system/ollama.service`, enabled and active |
| Ollama command | `/usr/local/bin/ollama serve` |
| Ollama account | Dedicated `ollama` user and group |
| Ollama API listener | `127.0.0.1:11434` (verified using `ss`) |
| Model storage | `/usr/share/ollama/.ollama/models` |
| Installed model | `qwen3:1.7b` (~1.4 GB on disk) |
| Model loaded at last check | None (`ollama ps` was empty) |
| Open WebUI | Healthy Docker container; image labels identify `main-slim` |
| Docker networking | `host` |
| Open WebUI → Ollama | `OLLAMA_BASE_URL=http://127.0.0.1:11434` |
| Docker Compose | **Not confirmed to be used**; no Compose labels found on the inspected container |
| Remote access | Tailscale Funnel provides public HTTPS access without router port forwarding (current setup reported; live target needs reconfirmation) |

**Why no top-level `docker-compose.yml`?** The first draft assumed both applications were Docker Compose services. Investigation showed a different architecture: Ollama is a native systemd service and the observed Open WebUI container has no Docker Compose labels. I moved Compose to an explicitly **optional, untested example** under `examples/` so it cannot be mistaken for the configuration I deployed.

## Documentation

| File | What it covers |
| --- | --- |
| [Architecture and findings](docs/01-architecture.md) | Confirmed vs. unverified parts of the setup |
| [Ollama configuration](docs/02-ollama-setup.md) | The actual systemd unit, storage, model, and API listener |
| [Open WebUI / Docker](docs/03-open-webui-docker.md) | Container image, host networking, connection to Ollama |
| [Tailscale access](docs/04-remote-access.md) | Public HTTPS access through Funnel and remaining checks |
| [Troubleshooting](docs/05-troubleshooting.md) | Diagnostics for models, connectivity, and permissions |
| [Security and publishing](docs/06-security-and-privacy.md) | What to redact before pushing to GitHub |
| [Reproducing the setup / gaps](docs/07-reproduction-and-gaps.md) | Verified commands, unresolved questions, and possible future improvements |

## Quick, read-only verification commands

On an Ubuntu host that already has these services installed:

```bash
systemctl is-active ollama
systemctl is-enabled ollama
ollama --version
ollama list
ollama ps
sudo ss -ltnp | grep ':11434'
sudo docker ps
sudo docker inspect open-webui --format '{{.HostConfig.NetworkMode}}'
sudo tailscale funnel status
```

**Review command output before sharing.** Full Docker inspection, environment variables, logs, and Tailscale status can expose tokens, private addresses, or user data. The commands above are intended for inspection, not configuration changes.

## What I learned

This project helped me understand the difference between a **containerized web interface** and a **native AI inference service**. I also learned why Docker's networking mode matters: because Open WebUI shares the host network, it can reach Ollama over localhost. Keeping Ollama bound to loopback avoids exposing its API directly on the LAN, while the publicly reachable chat interface relies on Open WebUI authentication and permissions.

Another useful lesson was not to assume that a container was created with Docker Compose just because it runs in Docker. Actual systemd units, Docker inspection results, and listening sockets tell a more accurate story.

## Safety

Do **not** publish real `.env` files, authentication tokens, `WEBUI_SECRET_KEY`, `OPENAI_API_KEY`, SSH private keys, Tailscale auth keys, user databases, or chat histories. This repository intentionally uses generic host descriptions and placeholders instead of the real Funnel URL or individual user accounts.

Do **not** run the optional Compose file on the live server without checking for existing containers, ports, persisted data, and backups. The examples are educational and do **not** claim to reproduce every detail of the running Open WebUI container.

## References

- [Ollama on Linux](https://github.com/ollama/ollama/blob/main/docs/linux.mdx)
- [Ollama FAQ](https://github.com/ollama/ollama/blob/main/docs/faq.mdx)
- [Open WebUI documentation](https://docs.openwebui.com/)
- [Docker host networking](https://docs.docker.com/engine/network/drivers/host/)
- [Tailscale Funnel](https://tailscale.com/docs/features/tailscale-funnel)
