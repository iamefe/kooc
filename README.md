# kooc — Kaggle GPU Agent Launcher

```
   ██╗  ██╗ ██████╗  ██████╗  ██████╗
   ██║ ██╔╝██╔═══██╗██╔═══██╗██╔════╝
   █████╔╝ ██║   ██║██║   ██║██║
   ██╔═██╗ ██║   ██║██║   ██║██║
   ██║  ██╗╚██████╔╝╚██████╔╝╚██████╗
   ╚═╝  ╚═╝ ╚═════╝  ╚═════╝  ╚═════╝
   Kaggle · Ollama · OpenCode / Claude 

   ────────────────────────────────────────

   Kaggle URL:
   ▶ _
```

A colorful interactive terminal launcher that connects to a remote [Ollama](https://ollama.com) instance running on a Kaggle GPU, lets you pick a model with arrow keys, and launches your chosen AI coding assistant.

---

## How it works

```
Kaggle GPU (T4×2)
  └─ Ollama server
       └─ Cloudflare Tunnel → public URL
                              ↕
                           kooc script
                          ↓
                    opencode / claude TUI
```

1. You enter (or confirm) your Kaggle tunnel URL
2. `kooc` runs `ollama list` against the remote instance to fetch available models
3. You pick a model from an interactive `fzf` picker
4. OpenCode or Claude Code launches with that model over `OLLAMA_HOST`

---

## Requirements

| Tool | Purpose | Install |
|------|---------|---------|
| `bash` | Shell | Pre-installed on macOS/Linux |
| [`ollama`](https://ollama.com) | CLI to query remote Ollama | `brew install ollama` |
| [`fzf`](https://github.com/junegunn/fzf) | Interactive model picker | `brew install fzf` |
| [`opencode`](https://github.com/opencode-ai/opencode) | AI coding assistant TUI | `brew install opencode` |
| A running Kaggle session | Remote GPU inference | See [Kaggle setup](#kaggle-setup) |

---

## Install

```bash
git clone https://github.com/iamefe/kooc ~/kooc

# Symlink to somewhere on your PATH
ln -s ~/kooc/kooc ~/.local/bin/kooc
chmod +x ~/.local/bin/kooc
```

Or copy it directly:

```bash
cp ~/kooc/kooc /usr/local/bin/kooc
chmod +x /usr/local/bin/kooc
```

---

## Usage

```bash
kooc
```

You'll see an interactive prompt:

```
   ██╗  ██╗ ██████╗  ██████╗  ██████╗
   ██║ ██╔╝██╔═══██╗██╔═══██╗██╔════╝
   █████╔╝ ██║   ██║██║   ██║██║
   ██╔═██╗ ██║   ██║██║   ██║██║
   ██║  ██╗╚██████╔╝╚██████╔╝╚██████╗
   ╚═╝  ╚═╝ ╚═════╝  ╚═════╝  ╚═════╝
   Kaggle · Ollama · Claude / OpenCode

   Kaggle URL:
   ▶ _
```

Paste your tunnel URL (Cloudflare, ngrok, etc.) and hit **Enter**. Then pick a model from the `fzf` list and your agent launches.

---

## Kaggle setup

Create a new Kaggle Notebook with the following cells to get Ollama running and expose it via a tunnel:

### Cell 1 — Install Ollama

```bash
!curl -fsSL https://ollama.com/install.sh | sh
```

### Cell 2 — Start Ollama with GPU support

```python
import subprocess, os

# Start Ollama server in the background
os.system("""
export OLLAMA_GPU=nvidia
export OLLAMA_MAX_LOADED_MODELS=1
export OLLAMA_KEEP_ALIVE=-1
ollama serve &
""")

print("Ollama server started.")
```

### Cell 3 — Pull a model

```bash
!ollama pull qwen2.5-coder:32b
```

Pick the largest model your GPU can handle. `qwen2.5-coder:32b` works on T4×2; for smaller resources, try `qwen2.5-coder:7b`.

### Cell 4 — Expose via tunnel

**Option A — Cloudflare (recommended):**
```bash
!curl -fsSL https://install.tailscale.com/os/cloudflare | sh
!tailscale up --advertise-tags=tag:ollama
# Check the output for the Tailscale IP, or use ngrok below
```

**Option B — ngrok:**
```bash
!wget https://bin.equinox.io/c/bNyj1mQVY4c/ngrok-v3-stable-linux-amd64.tgz
!tar xzf ngrok-v3-stable-linux-amd64.tgz
!./ngrok config add-authtoken YOUR_TOKEN
!./ngrok http 11434 &
```
Then grab the forwarding URL from the ngrok output.

### Cell 5 — Print your tunnel URL

```python
import os, socket

# For ngrok, the URL will be in the ngrok output above (e.g., https://abc123.ngrok-free.app)
# For local access:
print(f"Local: http://{socket.gethostbyname(socket.gethostname())}:11434")
```

Copy the URL and paste it into `kooc` when prompted. Your URL gets saved automatically so you don't have to re-enter it.

---

## Troubleshooting

**"Could not reach Ollama"**
- Your Kaggle session may have timed out or the tunnel URL has changed
- Restart the Kaggle notebook and update your saved URL via the delete picker

**"No models found"**
- The tunnel is reachable but Ollama has no models — run Cell 3 in the notebook to pull one

**`fzf: command not found`**
- Install fzf: `brew install fzf`

**`opencode: command not found`**
- Install OpenCode: `brew install opencode` or see [opencode.ai](https://opencode.ai)

**`ollama: command not found`**
- Install Ollama CLI: `brew install ollama` (the server doesn't need to run locally — just the CLI)

---

## License

MIT
