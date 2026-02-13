# Autoloop — Self-Feeding Thought Engine

A system that makes an LLM continuously consume its own output — autonomous thought through self-feeding.

```
while True:
    output = LLM(context)
    context += output
```

That's it. With just this, the LLM keeps thinking. It searches on its own, talks to humans on its own, and creates memories on its own.

## Setup

### 1. Install LM Studio

Download and install from https://lmstudio.ai.

### 2. Download a Model

Search for and download a model within LM Studio. Any model works, but the following are recommended:

- **Minimum**: 7B–14B models (VRAM 8GB+)
- **Recommended**: ~30B models (VRAM 24GB+)
- **Format**: GGUF (quantized)

### 3. Start the Server in LM Studio

1. Load a model
2. Open the **"Developer"** tab in the left sidebar
3. Click **"Start Server"**
4. Confirm the server is running at `http://localhost:1234`

### 4. Launch Autoloop

**Windows:**
```
Double-click start.bat
```

**Manual:**
```bash
pip install requests gradio
python autoloop.py --browser
```

A browser window will open with the UI. Press "▶ Start" to begin thinking.

## LM Studio Configuration Notes

### ⚠ Important: Enable Completions API

This system uses the **Completions API** (text completion). This is fundamentally different from the Chat API.

- **Completions API**: "Continue this text" → autonomous thought emerges
- **Chat API**: "Answer this question" → enters response mode, weakening autonomous thought

Most models in LM Studio support the Completions API by default, but some chat-only models may not. In that case, the system automatically falls back to the Chat API, though thought quality will degrade.

**How to verify:**
- After starting the server in LM Studio, navigate to `http://localhost:1234/v1/completions` in your browser — if there's no error, you're good.

### Context Length Settings

The larger the **"Context Length"** in LM Studio, the longer the thinking chains can be.

- **Minimum**: 4096 (compression kicks in frequently)
- **Recommended**: 8192–16384
- **Ideal**: 32768+ (if VRAM allows)

Adjust `compress_at_chars` and `max_context_chars` in autoloop.py to match your context length. As a rule of thumb: context length in tokens × 2 ≈ max_context_chars.

## Usage

### After Launch

Press "▶ Start" and just leave it running. The LLM will start thinking on its own.

The "🧠 Thoughts" panel on the right shows the thought log. The "💬 Dialog" panel on the left shows messages from the LLM.

### Talk to It

Type a message in the text box at the bottom and press "Send". Your voice interrupts the LLM's stream of thought.

### Logs

All thoughts and tool usage are saved as JSONL files in the `autoloop_log/` folder.

## Customizing the Seed

Edit `DEFAULT_SEED` in `autoloop.py` to change the starting point of the LLM's thought.

The seed determines everything. Even with the same model, a different seed leads to completely different thought convergence.

### Tips for Seed Design

- **Define existence, not behavior**: Use "You are..." rather than "Do this..."
- **Use questions**: Don't give answers — make it think
- **Include tool definitions in the seed**: The LLM needs to know tools exist in order to use them

## How It Works

```
context = seed_text                    # Seed (initial context)
while alive:
    output = completions_api(context)  # Generate "what comes next"
    context += output                  # Feed its own output back in
    if len(context) > limit:
        context = compress(context)    # When it overflows, compress and continue
```

- **completions API**: Text completion. Writing "what comes next." This is the core of autonomous thought.
- **context.append(output)**: Reading what it wrote itself. This is memory.
- **compress**: When context overflows, summarize and compress. Natural selection of memories.
- **Tools**: When the LLM spontaneously writes a text pattern like `[TOOL:name:content]`, the system detects it and returns a simulated response.

## License

MIT
