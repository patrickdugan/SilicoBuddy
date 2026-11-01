# SilicoBuddy

Token-economized Auto-research handoffs between Codex, ChatGPT and Silico, via a bounded Playwright Firefox session.

This standalone package is based on [PuppetMaster's SilicoBuddy implementation](https://github.com/patrickdugan/PuppetMaster/tree/main/scripts/silico-buddy). The code is intentionally plan-first. Every live browser send requires explicit `--approve-send`, and any Silico experiment or paid compute requires separate approval in Silico.

## Quick start

Requires Node.js 20+.

```powershell
git clone https://github.com/patrickdugan/SilicoBuddy.git
cd SilicoBuddy
npm install
npx playwright install firefox
npm test
npm run buddy -- --task "Can 1-bit quantization preserve held-out reasoning transfer?"
```

The default dry run writes packets and JSON receipts to `runs/` without making browser or model requests.

## Save a model call: Codex-first

Run a cheap local inspection and have Codex write a small experiment design to `codex-plan.txt`. Then:

```powershell
npm run buddy -- --task "Test Qwen 0.8B versus the held-out baseline" --plan-file .\codex-plan.txt
```

With `--plan-file`, live mode can send the packet directly to Silico without calling ChatGPT first. Omit it to let ChatGPT prepare the experiment packet. A third, optional ChatGPT debrief is available with `--debrief-send`.

## Live Firefox handoff

Open an existing Silico thread in a dedicated Firefox profile. Sign in to both services inside the launched browser. Inspect the current Silico editor and assistant-message CSS selectors, then run:

```powershell
npm run buddy -- --task-file .\question.txt `
  --live --approve-send `
  --chatgpt-url "https://chatgpt.com/" `
  --silico-url "https://<your-lab-host>.silico.ai/<thread>" `
  --silico-composer "<verified-css-selector>" `
  --silico-reply "<verified-css-selector>" `
  --max-ru 0.2 --max-gpu-hours 0
```

The permitted Silico hosts in the prototype are `silico.ai` and `silico.run` (including subdomains). Check your actual Silico service URL before using live mode; the current domain/DOM integration has not been authenticated or verified. The script uses a separate Firefox profile; it cannot attach to tabs in your existing everyday Firefox window.

## Experiment economics

- Prompt limit: 5,000 characters per exchange by default.
- Model response capture: first/last 6,500 characters by default.
- One request to Silico with a prewritten Codex plan; otherwise ChatGPT + Silico; optional third critique.
- `runs/*/summary.json` includes hashes and rough `chars / 4` token estimates. Actual RU and GPU billing need separate provider-side measurement.
- `--max-ru` and `--max-gpu-hours` express **constraints to the model**. They do not impose billing controls.
- Silico's automatic acceptance and experiment execution should remain disabled until explicit manual approval.

See [detailed setup](docs/SILICO_BUDDY.md). Run `npm run buddy -- --help` for the complete CLI.

## Provenance and validation

The initial implementation was copied from the PuppetMaster main branch, with standalone packaging and documentation. Offline tests exercise budget validation, prompt construction, redaction, and URL allowlisting. Live Firefox selectors and Silico billing remain to be checked against an authenticated session.
