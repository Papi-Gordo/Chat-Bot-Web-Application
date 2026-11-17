# Ollama: native Ubuntu service

Ollama is installed directly on Ubuntu, not in the Docker container. The installed binary is started by `systemd` using the service file at:

```text
/etc/systemd/system/ollama.service
```

## Verified service definition

The relevant unit was inspected with `sudo systemctl cat ollama`. A sanitized copy is stored at [`examples/ollama.service`](../examples/ollama.service); it contains the actual **non-secret** `PATH` setting seen on the host.

```ini
[Unit]
Description=Ollama Service
After=network-online.target

[Service]
ExecStart=/usr/local/bin/ollama serve
User=ollama
Group=ollama
Restart=always
RestartSec=3
Environment="PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin"

[Install]
WantedBy=default.target
```

`Restart=always` tells systemd to restart the process after it exits, with a three-second delay. The dedicated `ollama` account avoids running the server as root. `WantedBy=default.target` is the actual value in this installation; a generic tutorial may show a different target.

**Important:** This is a documentation copy, **not a command to overwrite another machine's service file**. Installation layout and binary paths may differ between systems and releases.

## Version and startup

```bash
ollama --version
systemctl is-active ollama
systemctl is-enabled ollama
systemctl show ollama -p FragmentPath -p DropInPaths -p User -p Group -p ExecStart
```

At inspection time, version `0.40.16` was installed; the service was `active` and `enabled`. No drop-in override paths were reported by `systemctl show`.

## Network listener

```bash
sudo ss -ltnp | grep ':11434'
```

The observed Ollama listener was:

```text
127.0.0.1:11434
```

That is the IPv4 loopback interface. It accepts requests from software sharing the host network namespace, including the Open WebUI container in this setup, but not direct inbound connections to the server's LAN IP through this socket.

`systemctl cat` did not show an `OLLAMA_HOST` override. The observed binding is more reliable evidence than assuming a default.

## Model storage and installed model

```text
/usr/share/ollama/.ollama/models/
├── blobs/
├── manifests/
├── manifests-v2/
└── metadata/
```

The filesystem check showed that these directories are owned by `ollama:ollama`. The server reported one downloaded model:

| Model | Reported disk size | State |
| --- | --- | --- |
| `qwen3:1.7b` | ~1.4 GB | Installed; not loaded at last check |

Useful commands:

```bash
ollama list         # Models available on disk
ollama ps           # Models currently loaded for inference
ollama pull qwen3:1.7b  # Download this model on another Ollama installation
```

An empty `ollama ps` output is normal when no model is currently loaded. It does **not** mean that the Ollama service is stopped.

## Data and logs

Model blobs should **not** be uploaded to this repository. They take substantial space and can be downloaded independently. Logs may include request paths, client IP addresses, or user content; inspect and redact before publishing.

```bash
journalctl -u ollama --no-pager -n 30
```

The server's earlier logs showed HTTP `200` responses from localhost, consistent with local API requests, but the raw request logs were deliberately excluded from this public documentation.
