# Troubleshooting notes

These are practical checks from operating a separate Open WebUI container and Ollama service. Use read-only diagnostics first.

## Open WebUI loads, but no models appear

1. Check Ollama is active: `systemctl is-active ollama`.
2. Check the model is downloaded: `ollama list`.
3. Check the API is listening: `sudo ss -ltnp | grep ':11434'`.
4. Verify the container's network mode is `host`.
5. Verify Open WebUI is configured to use `http://127.0.0.1:11434`.
6. Review Open WebUI's administrator model visibility and user/group permissions if only some accounts cannot see models.

Do not post the complete environment variable dump: it can include `OPENAI_API_KEY` and `WEBUI_SECRET_KEY` values.

## Ollama is active but nothing appears in `ollama ps`

`ollama list` shows downloaded models; `ollama ps` shows models **currently loaded**. An empty running-model list is not a service failure.

## The public browser times out

- Check the public HTTPS URL and current Funnel status: `sudo tailscale funnel status`.
- Confirm that the local Open WebUI port matches Funnel's configured target.
- Check Open WebUI's listening socket and relevant firewall rules.
- Check Funnel's HTTPS/certificate and DNS setup if the public URL is unavailable.
- No router port forwarding is required.

## Docker shows no PORTS mapping

With `--network host`, Docker doesn't use `-p` mappings. Inspect the host's listening sockets; a blank Docker `PORTS` column alone does not show that the application is private.

## Review logs carefully

```bash
sudo docker logs --tail 40 open-webui
journalctl -u ollama --no-pager -n 40
```

Logs may disclose request paths, IPs, or user-supplied content. Redact anything private before copying excerpts into issues or public documentation.
