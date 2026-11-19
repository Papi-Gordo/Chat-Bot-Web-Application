# Reproducing the setup and remaining gaps

## What is reproducible from verified information

The observed architecture is:

1. Ollama installed **natively** on Ubuntu and managed by `systemd`.
2. A model (`qwen3:1.7b`) downloaded into Ollama's storage.
3. Open WebUI in Docker with `host` networking.
4. Open WebUI using `OLLAMA_BASE_URL=http://127.0.0.1:11434`.
5. Tailscale Funnel used for public HTTPS access to the web interface without router port forwarding (reported as the current setup; live target not rechecked).

On a new Ubuntu machine, consult the official Ollama and Docker installation documentation for current installation instructions. Those commands and package versions change over time. The **observed unit file** is in `examples/ollama.service`, but it should not be blindly substituted on another computer.

On a compatible Ollama installation:

```bash
ollama pull qwen3:1.7b
ollama list
systemctl is-active ollama
```

The optional [`examples/docker-compose.optional.yml`](../examples/docker-compose.optional.yml) illustrates how **only Open WebUI** could be managed through Compose while Ollama stays on the host. It is **not** evidence that the inspected server uses Compose. The previously drafted Compose example that ran Ollama in a second container was removed from the documented deployment path to avoid misrepresenting the setup.

## What I still need to collect before writing a full installation walkthrough

1. How Open WebUI was originally started (`docker run` command or another deployment method).
2. Persistent storage mount source/destination and backup strategy.
3. Open WebUI container restart policy and exact image reference.
4. Current listener for the web app, including host address and port (security audit).
5. Whether the Docker Compose plugin is installed; the earlier `compose ls -a` invocation errored.
6. The current Tailscale Funnel public route/target and Open WebUI authentication settings, with URLs redacted before sharing.
7. Any additional Ollama environment overrides not yet inspected.

## Suggested safe commands for follow-up investigation

```bash
sudo docker inspect open-webui --format '{{.Config.Image}}'
sudo docker inspect open-webui --format '{{.HostConfig.RestartPolicy.Name}}'
sudo docker inspect open-webui --format '{{json .Mounts}}'
sudo ss -ltnp | grep -E ':(8080|11434)\b'
sudo tailscale funnel status
sudo docker compose version
```

For mounts and Tailscale output, replace identifying paths or hostnames before publishing. Do **not** share full Docker environment values or complete inspection JSON without reviewing for secrets.

## Future improvements

- Record backups and restores for Open WebUI persistent data.
- Document upgrading and rolling back both services.
- Add resource monitoring, especially because local models have noticeable memory requirements.
- Document how multi-user model permissions work in Open WebUI.
- Design separate routing for the website and AI interface so they do not compete for `/`.
