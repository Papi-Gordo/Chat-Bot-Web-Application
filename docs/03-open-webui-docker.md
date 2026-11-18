# Open WebUI: Docker configuration

Open WebUI provides the browser-based chat interface. **Ollama runs separately as an Ubuntu systemd service.**

## Verified configuration

- Container: `open-webui` (healthy when inspected)
- Image metadata: Open WebUI `main-slim`
- Network mode: **`host`**
- Ollama endpoint: `OLLAMA_BASE_URL=http://127.0.0.1:11434`
- No `com.docker.compose.*` labels found; the original launch method is unconfirmed

With **host networking**, Open WebUI shares Ubuntu's network namespace and can reach the Ollama API through localhost. Docker port mappings (`-p`) are not used in this mode. The application's own listening address still needs checking; an empty `PORTS` column does **not** mean the app is private.

## Read-only inspection

```bash
sudo docker ps
sudo docker inspect open-webui --format '{{.Config.Image}}'
sudo docker inspect open-webui --format '{{.HostConfig.NetworkMode}}'
sudo docker inspect open-webui --format '{{.HostConfig.RestartPolicy.Name}}'
sudo docker inspect open-webui --format '{{json .Mounts}}'
```

To display **environment variable names only**, without exposing values:

```bash
sudo docker inspect open-webui \
  --format '{{range .Config.Env}}{{println .}}{{end}}' \
  | cut -d= -f1
```

Review mount paths before posting them. Never publish unredacted `docker inspect` output or secret values such as `WEBUI_SECRET_KEY` and `OPENAI_API_KEY`.

## Reproduction note

The exact original `docker run` command, persistent mounts, and restart policy have not been recovered. [`examples/docker-compose.optional.yml`](../examples/docker-compose.optional.yml) is an **untested alternative**, not the live deployment. Do not start it alongside the existing container or replace volumes without a verified backup.
