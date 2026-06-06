# BWOC Handbook

Bilingual (EN canonical / TH parity), role-indexed handbook for the **BWOC framework** — a backend-neutral framework for incarnating and orchestrating AI coding agents, driven by one CLI: `bwoc`.

📖 **Read it online:** https://bemindlabs.github.io/bwoc-handbook/
🧩 **Framework repo:** https://github.com/bemindlabs/BWOC-Framework

## Contents

The handbook pages live under [`docs/`](docs/):

| Role | Page |
|---|---|
| 🧑‍💻 End user | [`docs/end-user/`](docs/end-user/HANDBOOK.en.md) |
| 🛠️ Developer | [`docs/developer/`](docs/developer/HANDBOOK.en.md) |
| 🤖 Agent author / operator | [`docs/agents/`](docs/agents/HANDBOOK.en.md) |
| ✍️ Persona · Mindsets · Skills | [`docs/slots/`](docs/slots/HANDBOOK.en.md) |
| 🔌 Backends (drive/configure via each CLI) | [`docs/backends/`](docs/backends/HANDBOOK.en.md) |
| 🔎 AI search / retrieval | [`docs/ai-search/`](docs/ai-search/HANDBOOK.en.md) · [`docs/llms.txt`](docs/llms.txt) |
| 🕷️ Crawler / indexer | [`docs/crawler/`](docs/crawler/HANDBOOK.en.md) · [`docs/robots.txt`](docs/robots.txt) · [`docs/sitemap.md`](docs/sitemap.md) |
| 🧩 Ecosystem | [`docs/ecosystem/`](docs/ecosystem/HANDBOOK.en.md) |
| 📖 Glossary | [`docs/glossary.en.md`](docs/glossary.en.md) |

Start at the handbook index: [`docs/README.md`](docs/README.md) (🇹🇭 [`docs/README.th.md`](docs/README.th.md)).

## Build the site locally

```bash
pip install -r requirements.txt
mkdocs serve     # preview at http://127.0.0.1:8000
```

Published automatically by [`.github/workflows/pages.yml`](.github/workflows/pages.yml) (MkDocs Material → GitHub Pages) on every push to `main`.

## License

MIT — see the [framework repository](https://github.com/bemindlabs/BWOC-Framework).
