# Which AI Model Fits Your Mac? 8 GB to 32 GB, Tested

Companion code for the Ring Zero video **[Which AI Model Fits Your Mac?](VIDEO_LINK)**.

Three small scripts that answer, for **your** Mac:

1. **How much memory can AI actually use here?** (Not the number on the box. About two-thirds of it.)
2. **Which models fit?** Every model from the video, checked against your Mac.
3. **How fast does it run, and is it fully on the graphics chip?** A repeatable speed test you can share.

No Python, no installs beyond Ollama. Plain shell scripts you can read in a minute.

## What you need

- A Mac with Apple Silicon (M1 or newer) and **macOS 14 Sonoma or newer**. Intel Macs work, but Ollama runs on the CPU only, so it's slow.
- **Ollama**, installed and open. New to it? Follow the setup video: [How to Run a Local LLM on Your Mac with Ollama](OCT6_VIDEO_LINK) or download it from [ollama.com/download](https://ollama.com/download).

## Quick start

Open **Terminal** and paste these one at a time:

```bash
git clone https://github.com/sreejithsr441/which-model-fits-mac.git
cd ring-zero-examples/which-model-fits-mac
./fit.sh
```

Then download the pick for your Mac and test it:

```bash
./pull-tier.sh
./verify.sh
```

That's it. The rest of this page explains each step and what the results mean.

## 1. Your Mac's real AI memory budget  *(video chapter 1:14)*

```bash
./fit.sh
```

Example on a 16 GB Mac:

```
Your Mac: Apple M2 · 16 GB memory
AI budget (graphics chip): 11.5 GB  ← estimate (about 2/3 of RAM up to 36 GB, 3/4 above)
```

**Why not 16?** macOS keeps part of the memory for itself. The graphics chip, the part that runs AI fast, gets about **two-thirds** on Macs up to 36 GB and about **three-quarters** on bigger ones. If Apple's developer tools are installed, `fit.sh` asks your Mac for the exact number instead of estimating.

> **Units:** sizes here match Ollama's, where 1 GB = 1,000,000,000 bytes. A Mac sold as "16 GB" has 17.2 GB in these units, so its budget shows as about 11.5 GB. (The video says "10.7", which is the same amount in Apple's binary gigabytes.)

| Your Mac | AI budget (Ollama's GB) | Comfortable model size |
|---|---|---|
| 8 GB | ≈ 5.7 GB | up to ≈ 4 GB |
| 16 GB | ≈ 11.5 GB | up to ≈ 10 GB |
| 24 GB | ≈ 17.2 GB | up to ≈ 15.5 GB |
| 32 GB | ≈ 22.9 GB | up to ≈ 21 GB |

"Comfortable" leaves 1.5 GB for the conversation's notepad (the *context*). Change it with `HEADROOM_GB=2 ./fit.sh`.

## 2. The picks for each Mac  *(chapters 2:22 – 5:37)*

`./fit.sh` checks all of these against your Mac. Sizes are Ollama's download sizes (Oct 2026).

| Mac | Pick | Also good | Trap (sounds like it fits) |
|---|---|---|---|
| 8 GB | `qwen3.5:4b` · 3.3 GB | `qwen3.5:2b-q4_K_M` · 1.9 GB, `gemma4:e2b-it-qat` · 4.3 GB | `gemma4:e2b-mlx` · 7.5 GB |
| 16 GB | `qwen3.5:9b` · 6.6 GB | `gemma4:12b-it-qat` · 7.2 GB | `gpt-oss:20b` · 14 GB |
| 24 GB | `gpt-oss:20b` · 14 GB | `gemma4:12b-it-q4_K_M` · 8.0 GB | `qwen3.6:27b-q4_K_M` · 17 GB |
| 32 GB | `qwen3.6:27b-q4_K_M` · 17 GB | `gemma4:26b-a4b-it-qat` · 16 GB (faster) | `qwen3.6:35b-a3b-q4_K_M` · 24 GB |

Download your tier's pick:

```bash
./pull-tier.sh          # picks your tier automatically
./pull-tier.sh 16       # or choose a tier
./pull-tier.sh 16 --alt # the "also good" model instead
```

It won't download a model that's too big for your Mac (add `--force` if you really want to).

## 3. Check any model in 10 seconds  *(chapter 6:20)*

New models come out every week. To check one:

1. Open its page on [ollama.com/library](https://ollama.com/library), click **Tags**, and note the size.
2. Run:
   ```bash
   ./fit.sh --size 9.4
   ```
   or, if you've already downloaded it:
   ```bash
   ./fit.sh some-model:tag
   ```
3. Run it, then check where it's running:
   ```bash
   ollama ps
   ```
   **100% GPU** means the whole model is on the graphics chip. If you see **CPU** (for example `48%/52% CPU/GPU`), part of it didn't fit and it will be slow.

Curious what a different Mac would get? `FIT_RAM_GB=24 ./fit.sh`

## 4. Measure the speed

```bash
./verify.sh                # your tier's pick
./verify.sh qwen3.5:9b     # any downloaded model
./verify.sh qwen3.5:9b --runs 5
```

It sends the same prompt each time (thinking turned off where the model allows), reports the **median tokens per second** of 3 runs, and reads `ollama ps` to show whether the model is 100% on the graphics chip. A *token* is a word piece; people read about 4–5 words a second.

At the end it prints one line to paste in the video's comments, like this (example numbers):

```
Apple M2 · 16 GB · qwen3.5:9b · 18.2 tok/s · 100% GPU
```

Every run is also saved to `results.csv` in this folder.

**Why your speed differs from the video:** memory size decides *whether* a model fits; your chip's memory bandwidth decides *how fast* it runs. An M4 Pro will be much faster than an M1 with the same model.

## 5. Troubleshooting  *(chapter 6:56)*

| You see | What it means | Fix |
|---|---|---|
| `ollama ps` shows `CPU` or a `CPU/GPU` split | The model is bigger than your AI budget | Pick a smaller model, or the `-qat` version (smaller). `./fit.sh` lists what fits |
| `model requires more system memory` | Not enough free memory right now | Close heavy apps (a browser with lots of tabs is the usual one) and try again |
| Fast at first, slower after a long chat | The conversation notepad (context) grew | Start a fresh chat |
| `Can't reach Ollama at http://127.0.0.1:11434` | Ollama isn't running | Open the Ollama app, or run `ollama serve` in another Terminal window |
| `permission denied: ./fit.sh` | The download lost the "run" permission | `chmod +x *.sh` |
| A thinking model takes ages to answer | It's "thinking" before it writes | In `ollama run`, type `/set nothink`. `verify.sh` turns thinking off for you |

Ollama's own log, if something else breaks: `~/.ollama/logs/server.log`

## For the curious: simulate a smaller Mac

The video tested all four sizes on one Mac by lowering the graphics chip's memory limit:

```bash
./simulate-tier.sh 16     # behave like a 16 GB Mac (asks for your password)
# quit and reopen Ollama, then ./fit.sh and ./verify.sh <model>
./simulate-tier.sh reset  # back to normal (restarting the Mac also resets it)
```

It only ever **lowers** the limit. It shows whether a model *fits* on a smaller Mac; speeds are still your own chip's. You'll find guides that *raise* this limit to squeeze in bigger models. On an 8 or 16 GB Mac, don't: macOS needs that room, and Apple doesn't support it.

## Files

| File | What it does |
|---|---|
| `fit.sh` | Your AI budget, and which models fit |
| `pull-tier.sh` | Downloads the pick for your Mac |
| `verify.sh` | Speed test + "is it 100% on the GPU?" check |
| `simulate-tier.sh` | Pretend to be a smaller Mac (optional) |
| `models.tsv` | The models from the video and their sizes. Add your own lines |
| `lib.sh` | Shared helpers |
| `TESTED.md` | The exact setup and results from the video |

## Sources

- Ollama model sizes: [ollama.com/library](https://ollama.com/library) (tag pages, checked Oct 2026)
- Ollama context defaults: [docs.ollama.com/context-length](https://docs.ollama.com/context-length)
- `ollama ps` CPU/GPU column: [docs.ollama.com/faq](https://docs.ollama.com/faq)
- gpt-oss "only requires 16GB": [openai.com/index/introducing-gpt-oss](https://openai.com/index/introducing-gpt-oss/)
- A 16 GB Mac's GPU limit (10922.67 MB): [llama-cpp-python #687](https://github.com/abetlen/llama-cpp-python/issues/687)

Model sizes and tags change quickly. If a tag disappears, check its Ollama page and update `models.tsv`.
