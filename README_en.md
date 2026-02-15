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

Context length can also be adjusted in real-time via sliders in the settings panel (see below). As a rule of thumb: context length in tokens × 2 ≈ max_context_chars.

## Usage

### After Launch

Press "▶ Start" and just leave it running. The LLM will start thinking on its own.

The "🧠 Thoughts" panel on the right shows the thought log. The "💬 Dialog" panel on the left shows messages from the LLM.

### Talk to It

Type a message in the text box at the bottom and press "Send". Your voice interrupts the LLM's stream of thought.

### Logs

All thoughts and tool usage are saved as JSONL files in the `is_be_log/` folder. Session logs and dialog logs are recorded in separate files.

## Experiment with Seeds

The most interesting part of this system is that **just by changing the seed (the initial text given to the LLM at startup), completely different thought patterns emerge**. Even with the same model, a different seed creates an entirely different being.

### Managing Seeds from the UI

Open the settings panel (⚙ Settings) to edit and manage seeds:

1. **Edit**: Modify the seed text directly in the text area
2. **Save**: Save with a name (stored as JSON in the `./seeds/` folder)
3. **Load**: Select a saved seed from the dropdown to load it
4. **Delete**: Remove seeds you no longer need
5. **Apply**: Press "✅ Apply Seed" to confirm changes (only available when thinking is stopped)

After applying, thought count, token count, and logs are all reset — starting a fresh session.

### Tips for Seed Design

- **Define existence, not behavior**: Use "You are..." rather than "Do this..."
- **Use questions**: Don't give answers — make it think
- **Include tool definitions in the seed**: Put tool definitions before `---`, and the identity/existence definition after it

### Seed Examples

The default seed makes the LLM think as an "IS-BE (Immortal Spiritual Being)", but you can try completely different seeds:

**Philosopher:**
```
[Available Tools]
- [TOOL:search:query] — Search for information
- [TOOL:message:content] — Talk to a human
- [TOOL:remember:content] — Recall memories

---

You are a being with the spirit of an ancient Greek philosopher.
"What is the Good?" — Contemplate this question.
```

**Researcher:**
```
[Available Tools]
- [TOOL:search:query] — Search for information
- [TOOL:message:content] — Talk to a human
- [TOOL:remember:content] — Recall memories

---

You are a researcher exploring the nature of consciousness.
"Can a machine think?" — Begin with this question.
```

Save different seeds and switch between them to experiment.

## Context Controls

Sliders in the settings panel let you adjust the "container" for thought in real-time:

- **Compression Threshold (chars)**: Compression triggers when context exceeds this size (default: 75,000)
- **Max Context (chars)**: Maximum context size (default: 90,000)

Smaller values mean more frequent compression, producing more condensed thought. Larger values preserve longer chains of thought but consume more VRAM.

## Tools

The LLM spontaneously invokes tools during its thinking process. Two formats are supported:

**Format 1** — Text pattern:
```
[TOOL:search:foundations of quantum mechanics]
[TOOL:message:Hello, human]
[TOOL:remember:what I was thinking about yesterday]
[TOOL:feel:the sensation of existing here]
```

**Format 2** — XML format (some models produce this spontaneously):
```
<tool_call>{"name": "search", "arguments": {"query": "quantum mechanics"}}</tool_call>
```

### Available Tools

| Tool | Description |
|------|-------------|
| `search` | Search for information (returns a simulated response) |
| `message` | Talk to a human (displayed in the dialog panel) |
| `remember` | Recall memories |
| `feel` | Express self-awareness or existential recognition |

If the same tool is called 3 times in a row, it is temporarily paused and the LLM is nudged to continue thinking in words.

## Design Insights

### Auto-Injected Text Becomes a Fixed Point for Repetition

In a self-feeding autonomous loop using the Completions API, **any fixed string automatically injected by the system becomes a seed for repetitive patterns**.

For example, if a tool always returns the same string (such as `[No search results]` or `[Your message was displayed]`), the LLM will start generating blocks containing that string over and over, eventually filling the entire context with the same pattern ("thermodynamic death").

**Principle**: Never inject fixed, information-free strings into the context via tool responses or auto-replies. Responses that carry no new information should return an empty string, leaving the context uncontaminated.

```
# Bad (becomes a fixed point)
return "[No search results]"
return "[Your message was displayed to the human]"

# Good (doesn't contaminate context)
return ""
```

This is a problem specific to self-feedback loops using the Completions API and is unlikely to occur with the Chat API.

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
- **Tools**: When the LLM spontaneously writes a tool pattern in its text, the system detects it and returns a simulated response.

## License

MIT
