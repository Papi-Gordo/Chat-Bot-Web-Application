# Public remote access through Tailscale Funnel

**Tailscale Funnel provides public HTTPS access to services hosted on my Ubuntu server without traditional router port forwarding.** Anyone on the internet can reach the published URL, including people who are **not** members of my Tailscale network. Open WebUI's login and user permissions are therefore essential.

## Connection path

```text
Public browser (no Tailscale client required)
               |
         HTTPS / Funnel
               |
  Open WebUI on Ubuntu (Docker: host)
               |
       127.0.0.1:11434
               |
         Ollama (systemd)
```

## Read-only verification

```bash
sudo tailscale funnel status
sudo ss -ltnp | grep -E ':(8080|11434)\b'
```

Check the URL, target port, and local listening sockets. Funnel itself does not require router port forwarding, but a host-networked Open WebUI process could also listen on a LAN-facing interface; that remains to be checked.

## Security considerations

- **Funnel makes the web endpoint public.** Tailscale network membership does not restrict visitors to this URL.
- Keep Open WebUI authentication enabled; review new-user registration, account approval, and model permissions.
- Leave the Ollama API bound to localhost; do not separately expose port `11434`.
- Review logs and service updates, and use placeholders when documenting public URLs or hostnames.
- If hosting multiple websites or applications, verify which Funnel target receives each route so one service does not displace another.

Reference: [Tailscale Funnel documentation](https://tailscale.com/docs/features/tailscale-funnel).
