# Architecture and investigation notes

*Want to eventually place ollama into container to decrease chance of hanging entire server, need to weigh against losing speed if max cpu use limited*

## Confirmed connections

```text
[Public browser]
       |
       v
[Tailscale Funnel: public HTTPS]
       |
       v
[Open WebUI in Docker, host network]
       |
       | OLLAMA_BASE_URL=http://127.0.0.1:11434
       v
[Ollama service on Ubuntu, port 11434]
       |
       v
[Downloaded model: qwen3:1.7b]
```

Open WebUI and Ollama are **not two containers on an isolated Docker Compose network**. `docker inspect` reported `host` for Open WebUI's network mode. Ollama is running as an enabled Ubuntu `systemd` service.

## Public HTTPS access

Tailscale **Funnel** is used to make the Open WebUI service reachable from the public internet over HTTPS **without traditional router port forwarding**. Public reachability is different from access permission: Open WebUI must enforce its own authentication. The specific live Funnel route and listening address have not yet been rechecked.

## Evidence gathered

| Check | Observation | Confidence |
| --- | --- | --- |
| `systemctl is-active ollama` | `active` | Confirmed |
| `systemctl status ollama` | Enabled, running with `/usr/local/bin/ollama serve` | Confirmed |
| `systemctl show ollama` | Service account `ollama`; no drop-in paths reported | Confirmed |
| `systemctl cat ollama` | Service file reproduced in `examples/ollama.service` | Confirmed |
| `sudo ss -ltnp \| grep ':11434'` | Listener at `127.0.0.1:11434` | Confirmed for IPv4 socket at check time |
| `ollama --version` | `0.40.16` | Confirmed |
| `ollama list` | `qwen3:1.7b` (~1.4 GB) | Confirmed |
| `ollama ps` | Nothing loaded at check time | Confirmed |
| `sudo docker ps` | One healthy container named `open-webui` at inspection time | Confirmed |
| `docker inspect ... NetworkMode` | `host` | Confirmed |
| `docker inspect ... Config.Labels` | Open WebUI `main-slim` metadata; no Compose labels in output | Confirmed |
| Open WebUI `OLLAMA_BASE_URL` | `http://127.0.0.1:11434` | Confirmed |
| Tailscale Funnel | Public HTTPS access without router port forwarding | Confirmed |

## Open questions

- Exact `docker run` command used to create Open WebUI, including volumes, restart policy, and binding behavior.
- Whether Docker Compose is installed or in use elsewhere. `sudo docker compose ls -a` failed with a Docker CLI flag error; this does not prove the plugin is absent.
- How Open WebUI's port `8080` is **currently** bound, and whether local firewall rules restrict direct LAN access.
- Whether Ollama has additional systemd/environment settings not covered by the checks already run.
- Current Tailscale Funnel route/target and Open WebUI authentication policies (not yet independently verified).

## Time-specific observations

Runtime durations, timestamps, PIDs, memory peaks, model loaded state, and container IDs are operational snapshots, not stable setup requirements. They are excluded from reproducible configuration examples.
