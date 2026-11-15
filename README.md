# Jev Ultrafast — Browser Flight Agent

A browser-use agent that finds flights using [Jev](https://typesafe.ai), TypeSafe AI's System One decision model. This repository is Ankit Kalluraya's adaptation of the open-source [Browser Use Jev Ultrafast project](https://github.com/browser-use/jev-ultrafast).

## Attribution and my contribution

The core browser agent, original demonstrations and performance measurements were implemented upstream by [Gregor Žunič (gregpr07)](https://github.com/gregpr07) and the Browser Use project. The upstream MIT licence and copyright notice are retained in [LICENSE](LICENSE).

My contributions are the README and Windows setup documentation, repository ignore rules, and explicit UTF-8 file reads in `browser.py` and `demo.py` for Windows compatibility. These are adaptations of the upstream implementation, not original authorship of its core agent or benchmark results. See [my changes](https://github.com/kallurayaankit/jev-flight-agent/commit/6205ffa13fd043732ff2c1536b5870a933e21a26).

## What it does

Give it a natural language task like "Find one-way flights from Zurich to London on September 20, 2026" and it drives a real Chrome browser to complete the search. Jev picks the next action at each step. A small language model handles only text generation (like typing "Zurich" into a field).

## Why Jev?

Traditional browser agents screenshot the page and ask a large LLM "what should I do next?" — slow and expensive. Jev flips this:

- **Jev picks, code acts.** Jev returns structured decisions (which element, what operation) with calibrated confidence. It never touches the browser.
- **A tiny LLM only writes text.** When Jev decides `TYPE_TEXT`, a small model generates the string. Jev's output tokens are free.
- **DOM state, not screenshots.** The agent reads the page's interactive elements as an indexed table. No visual parsing.

The upstream demonstration reports approximately 7 seconds for one flight-search task. Its [performance notes](docs/performance.md) describe the measurement boundaries and limited test scope. This is not a new benchmark by me or a guarantee for other searches; runtime and cost vary with models, pricing and page behaviour.

## Tech stack

- **Jev** (TypeSafe AI) — decision layer
- **Playwright** — browser automation
- **browser-harness** — CDP connection to Chrome
- **Python 3.12**

## Getting started

### Prerequisites

- Python 3.12
- Google Chrome (136+)
- A TypeSafe API key from [console.typesafe.ai](https://console.typesafe.ai)
- An OpenRouter API key from [openrouter.ai](https://openrouter.ai)

### Install

```bash
git clone https://github.com/YOUR_USERNAME/jev-ultrafast.git
cd jev-ultrafast
uv sync --python 3.12
```

### Configure

Copy `.env.example` to `.env` and add your keys:

```
TYPESAFE_API_KEY=ts_your_key_here
TEXT_MODEL_API_KEY=sk-or-your_key_here
```

### Run

1. Launch Chrome with remote debugging enabled:
   ```bash
   # macOS/Linux
   google-chrome --remote-debugging-port=9222 --user-data-dir=/tmp/chrome-debug

   # Windows
   start "" "C:\Program Files\Google\Chrome\Application\chrome.exe" --remote-debugging-port=9222 --user-data-dir="%USERPROFILE%\ChromeDebugProfile"
   ```

2. Get the WebSocket URL:
   ```bash
   curl http://127.0.0.1:9222/json/version
   ```

3. Set the environment variable (use the URL from step 2):
   ```bash
   # macOS/Linux
   export BU_CDP_WS=ws://127.0.0.1:9222/devtools/browser/YOUR-UUID

   # Windows
   set BU_CDP_WS=ws://127.0.0.1:9222/devtools/browser/YOUR-UUID
   ```

4. Start the server:
   ```bash
   uv run jev
   ```

5. Open `http://127.0.0.1:8766` and click **Start demo**.

## Windows notes

The project required several Windows-specific fixes:

- **Encoding:** Add `encoding="utf-8"` to `read_text()` calls, or set `PYTHONUTF8=1`.
- **Chrome 136+:** Requires `--user-data-dir` and the `chrome://inspect/#remote-debugging` toggle.
- **WebSocket URL expires:** Get a fresh URL with `curl http://127.0.0.1:9222/json/version` each session.

## Safety

The agent **stops at the results page**. It does not select or book flights. Never let it proceed to payment or booking without explicit human confirmation.

## License

MIT