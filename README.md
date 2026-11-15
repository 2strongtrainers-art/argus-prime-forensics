# ARGUS PRIME — Private Forensic Worker

This is the standalone GitHub repository created during the Cloudflare deployment process. It contains the **full forensic service**, restored from the already-tested `2strongtrainers-art/Nicholas-AI-OS/services/osint-forensics` source at commit `3b19186e3d04af80a83ddde46018fa9f889572a8`.

**A GitHub repository is not proof that Cloudflare has deployed a working Worker or Container.** Verify the live Cloudflare deployment before using it.

## Implementation

- `edge/index.js`: HTTP gateway, requires a separate encrypted `ARGUS_FORENSICS_API_KEY` secret of 32 or more characters before analyzing any evidence.
- `wrangler.jsonc`: independent Cloudflare Worker `argus-prime-forensics`, with a Durable Object and a Container set to `enableInternet:false`.
- `Dockerfile`, `analyze.py`, `http_server.py`: read-only, offline local ExifTool/C2PA parsing and SHA-256 integrity.
- `tests/` and GitHub CI: synthetic-only tests; **never use private media or credentials in public CI**.

The default output omits GPS and other sensitive EXIF metadata, and uploaded bytes are processed in temporary files. Cloudflare is still a cloud processor subject to its ingress and telemetry rules. No face search, automatic public web crawler or unrestricted external media upload is enabled.

## Deployment status

The GitHub import may have completed while the Cloudflare build or Worker did not. Inspect the existing `argus-prime-forensics` Worker in **Cloudflare Dashboard → Workers & Pages**. Do not create a duplicate project and never overwrite `nicholas-ai-switchboard` or `nicholas-ai-os`.

If Cloudflare GitHub Builds is attached, the next commit to this standalone repo's `main` should trigger its normal build. **Do not claim this happened until you see the Cloudflare Build and deployment result.**

The Worker returns `/health` with `enabled:false` until its distinct encrypted secret is configured. A complete acceptance test requires `/health` enabled true, unauthenticated `POST /analyze` returning 401, and an authenticated synthetic PNG returning a correct SHA-256 and ExifTool output. It also requires Workers Paid/Containers to be enabled.

For a secure deployment and test sequence, see [the Cloudflare deployment runbook](CLOUDFLARE_DEPLOYMENT.md).

The original source of truth remains [Nicholas-AI-OS](https://github.com/2strongtrainers-art/Nicholas-AI-OS/tree/main/services/osint-forensics). This copy is a distinct repository and needs explicit updates when the source changes.

## Release verification

Cloudflare's Git integration builds this standalone repository from `main`.
A successful GitHub CI run or `wrangler deploy --dry-run` is not proof of a live
Cloudflare Container. Validate the actual Cloudflare build, then check the new
Worker's `GET /health`, authentication boundary and a synthetic SHA-256
round-trip before enabling real evidence. Do not transmit secrets in GitHub
Actions or commit history.
