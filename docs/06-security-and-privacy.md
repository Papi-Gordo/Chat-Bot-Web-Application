# Security and GitHub publishing checklist

This repository documents the setup without copying live secrets, account data, or complete server inspection output.

## Safe to publish

Software versions, public image names, standard Linux paths, localhost URLs (`127.0.0.1`), the `ollama` service account, Docker networking mode, model names, and non-secret service settings.

## Keep private

- Passwords; `.env` files; API keys; `WEBUI_SECRET_KEY`; `OPENAI_API_KEY`; Tailscale auth keys
- SSH/private TLS keys; cookies, sessions, user databases, chat history, and private model data
- Raw Docker inspections, environment values, logs, backups, and identifying usernames/hostnames
- Actual public Funnel URL and private network details unless intentionally disclosed

## Before pushing to GitHub

- [ ] `.env` is not tracked; `.env.example` contains placeholders only
- [ ] No credentials, keys, databases, chat exports, or raw inspection dumps
- [ ] Logs and screenshots are reviewed for personal information
- [ ] The optional Compose example is clearly marked **not deployed**

The root [`.gitignore`](../.gitignore) helps exclude new local files, but **does not remove files already committed**. Check with:

```bash
git status --short
git ls-files
git check-ignore -v .env
```

If a secret was committed, rotate it; deleting the latest copy alone does not remove it from Git history.

## Public endpoint precautions

**Tailscale Funnel exposes Open WebUI to the internet.** HTTPS protects traffic in transit; it does not restrict who can open the login page. Keep authentication enforced, review signup/account approval and model permissions, and keep components updated. Ollama's API was confirmed bound to `127.0.0.1:11434`, but Open WebUI's own listening interface is still unverified and should be checked separately.
