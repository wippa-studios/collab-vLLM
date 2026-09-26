# collab-vLLM

Run **Qwen3 on a free Colab GPU** and point [OpenCode](https://opencode.ai) at it
as an OpenAI-compatible provider — no local GPU, no API key, no account.

One notebook, `Qwen3-8B.ipynb`. It installs vLLM, serves the model, opens a
public tunnel to it, and prints the exact OpenCode provider block to paste into
your config.

> The file is named for the 8B model, but the default `MODEL_ID` in the notebook
> is `Qwen/Qwen3-1.7B`, which is the safer default on a Colab T4. Change line 1
> of Cell 2 to `Qwen/Qwen3-4B` or `Qwen/Qwen3-8B` for noticeably better coding
> quality — both still fit a T4.

## How it works

1. **Install** — `pip install vllm`, plus a `cloudflared` binary for the tunnel.
2. **Serve** — `vllm serve Qwen/Qwen3-1.7B` on port 8000, bound to `0.0.0.0`,
   with a locally generated API key, `--reasoning-parser qwen3` (so
   `<think>` content is split into a separate `reasoning_content` field rather
   than leaking into the answer), a 32k context window and
   `--gpu-memory-utilization 0.85`.
3. **Tunnel** — a `cloudflared` quick tunnel, no account or signup needed.
4. **Configure** — prints an `@ai-sdk/openai-compatible` provider block for
   `~/.config/opencode/opencode.json` (or a project-local `opencode.json`).

## Running it

Open `Qwen3-8B.ipynb` in Google Colab, choose **Runtime → Change runtime type →
T4 GPU**, and run the cells in order. The first run downloads the model weights,
which takes a few minutes.

The notebook prints your base URL and API key, and you paste the emitted config
into OpenCode. Then:

```bash
opencode --model colab-qwen3/Qwen/Qwen3-1.7B
```

## Things worth knowing

- **Keep the Colab tab open and running.** Closing it, or letting the runtime
  idle-disconnect, kills the server and invalidates the tunnel URL. There is no
  persistence here — this is a scratch GPU, not a deployment.
- **The endpoint is public.** A `cloudflared` quick tunnel is reachable by anyone
  who has the URL, and the notebook prints that URL into your terminal output.
  The API key is generated per-run with `secrets.token_urlsafe(24)` and is meant
  to be disposable: treat it as burned once the tab is closed, and never reuse
  it anywhere else. Do not point it at anything you care about.
- **Colab quotas apply.** Free GPU time is limited and Colab may terminate
  long-running sessions. This is fine for experiments, not for anything you
  depend on.
- If you hit GPU out-of-memory, lower `MAX_MODEL_LEN` or drop back to the 1.7B
  model before reaching for the 4B/8B ones.

## Status

This is a working notebook rather than a maintained package. There is no test
suite and no CI — the "test" is running it and seeing a completion come back
through OpenCode.

## License

MIT. See [LICENSE](LICENSE).
