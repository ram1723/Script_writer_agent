# Script Writer — AI-assisted Script Generation Pipeline

**One-line:** A modular LangGraph + Google GenAI pipeline that turns a single logline into a formatted screenplay draft, with critique-and-optimize cycles.

---

## Overview

This notebook / project implements a multi-step pipeline that produces a screenplay-style first draft from a short logline. It uses Google GenAI (Gemini) as the language model backend and LangGraph's StateGraph pattern to compose prompt-driven nodes. Each node performs a focused part of the writing workflow (title, synopsis, character bios, scene breakdown, draft assembly, critique and optimization, scoring, and optional formatting to screenplay layout).

The intent is to provide a reproducible, modular workspace to iterate and improve generated drafts automatically, and to make it easy to adapt prompts, node topology, or models.

---

## Key features

- Modular StateGraph pipeline with clearly separated responsibilities (parsing, title, synopsis, scene breakdown, draft assembly, critique, optimizer, scoring).
- Uses `google-genai` (Gemini) as the model (the notebook uses `gemini-2.5-flash` by default).
- Automatic critique → optimization loop: low scoring drafts are re-assigned and re-processed until they pass a configurable threshold.
- Optional formatting node that converts the JSON-style draft into a standard screenplay layout.
- Ready to run in Google Colab (the notebook uses `google.colab.userdata` to fetch the API key) or locally by setting `GOOGLE_API_KEY`.

---

## High-level architecture / nodes

The graph in the notebook contains (but is not limited to) these nodes:

- `logline_parser` — parses / normalizes the input logline into structured pieces used downstream.
- `title_generator` — generates a short, attention-grabbing title from the parsed logline.
- `synopsis_generator` — writes a 2–3 paragraph synopsis (beginning, middle, end).
- `character_bio_generator` — creates short bios for the primary characters.
- `scene_breakdown` — expands a draft skeleton into scene-level action + dialogue JSON elements.
- `location_planner` — (optional) suggests locations/sets for scenes for production planning.
- `draft_assembler` — builds a draft skeleton (scene order, beats) usable by the first-draft generator.
- `first_draft_generator` — uses the skeleton to produce a full draft (still in JSON or structured format).
- `evaluator` (critique) — asks the model to analyze the draft and return a JSON critique and scorecard.
- `optimizer` — applies the critique to produce a `revised_draft` (a new draft iteration).
- `scoring` — parses numeric score and decides acceptance vs. further review. By default, a threshold of `75.0` is used.
- `convert_script_format` — converts the JSON draft into a conventional screenplay layout (scene headings, capitalized character names, centered dialogue, etc.).

---

## Dependencies

The notebook installs and/or expects these Python packages:

```bash
pip install -U langgraph langchain_community langchain_core tavily-python langchain_nvidia_ai_endpoints
pip install -U "google-genai>=1.16.0"
The notebook also imports google.genai and uses:

python
Copy
Edit
client = genai.Client(api_key=...)
Setup
1) Google API key
Colab: uses google.colab.userdata to fetch GOOGLE_API_KEY. Set via the Colab UI or replace with os.environ code.

Local:

bash
Copy
Edit
export GOOGLE_API_KEY="your-key-here"
python
Copy
Edit
from google import genai
import os
client = genai.Client(api_key=os.environ["GOOGLE_API_KEY"])
2) Install dependencies
Run the pip commands above. A virtual environment is recommended.

3) Model & cost
Uses gemini-2.5-flash. Multiple iterations or large prompts may incur cost.

Usage (quick)
Provide an initial logline:

python
Copy
Edit
initial_state = {
    "logline": "A young genius prodigy interrupts a series of meteor strikes; he fights them with his friends and saves the city."
}
Invoke the graph:

python
Copy
Edit
output = graph.invoke(initial_state)
print(output.get("first_draft"))
Low-scoring drafts are iteratively optimized until they meet the threshold.

How to customize
Prompts: Edit PromptTemplate text to change tone, constraints, or length.

Threshold: Change inside scoring() (default 75.0).

Nodes: Add/remove in StateGraph via add_node() and change edges.

Models: Swap gemini-2.5-flash for others supported by your account.

Implementation notes
Multiple helper function versions exist — consider cleanup for production.

The evaluator defensively parses JSON scorecards; falls back if parsing fails.

scoring() extracts numeric value (0–100) and compares to threshold.

convert_script_format expects JSON scenes array.

Running outside Colab
Replace google.colab.userdata with environment variable reading.
Ensure network access and correct authentication.

Limitations & safety
LLM outputs may hallucinate, produce unsafe content, or have bias.

Add human review before production.

Avoid generating scripts too close to copyrighted works.

Next steps
Web UI (Streamlit/Gradio) for input and iteration.

Export to PDF/Fountain.

Unit tests for prompt outputs and JSON stability.

TTS read-through with Gemini TTS.
