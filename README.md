# Script Writer — AI-assisted Script Writer Pipeline

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

The notebook installs and/or expects these Python packages (used in the included cells):

```bash
pip install -U langgraph langchain_community langchain_core tavily-python langchain_nvidia_ai_endpoints
pip install -U "google-genai>=1.16.0"
```
## Setup
1) Google API key
Colab: the notebook fetches GOOGLE_API_KEY using google.colab.userdata. If you run in Colab, set the key using the Colab UI snippet or replace that line to read from environment variables.

Locally: export your key before running:

```bash

export GOOGLE_API_KEY="your-key-here"
Then in code, create the client:
```
```python

from google import genai
import os
client = genai.Client(api_key=os.environ["GOOGLE_API_KEY"])
```
##Usage
Provide an initial logline (string). Example used in the notebook:

```python

initial_state = {"logline": "A young genius prodigy interrupts a series of meteor strikes; he fights them with his friends and saves the city."}
Invoke the graph:
```
```python

# assuming `graph` is constructed as in the notebook
output = graph.invoke(initial_state)
print(output.get("first_draft"))
```
If enabled, the pipeline will run evaluation and optimization cycles. If a revised_draft scores below threshold, the pipeline will assign first_draft = revised_draft and continue review.

How to customize
Edit prompts: All prompt text is defined using PromptTemplate instances in the notebook. Update the templates to change tone, length, or constraints.

Change threshold: Edit the numeric threshold inside scoring() to make acceptance easier or stricter (default 75.0).

Add / reorder nodes: The graph uses StateGraph (langgraph.graph.StateGraph) — add or remove nodes using g.add_node(...) and change edges as needed.

Switch models: Replace gemini-2.5-flash with other model identifiers supported by your account (be mindful of token limits and costs).

Important implementation notes
The notebook contains multiple copies/versions of similarly named helper functions — this is normal during iterative development, but consider cleaning duplicates before turning the code into a reusable module.

The evaluator attempts to parse JSON scorecards from the model output. Because model text can be noisy, parsing is defensive: if it cannot extract the JSON scores, the code falls back gracefully.

The scoring() function extracts the first numeric value in the model reply and constrains it to [0, 100]. It then compares to the configured threshold and, if below threshold, assigns the revised_draft back to first_draft so the optimizer may re-run.

The convert_script_format node uses a separate prompt that expects a JSON array representing scenes and outputs a conventional screenplay layout. Keep the JSON format stable if you modify the scene generator.

## Next steps / ideas
Add a simple web UI (Streamlit or Gradio) to input loglines and show iterative drafts.

Export final screenplay to PDF or Fountain format.

Add fine-grained unit tests for prompt outputs (e.g., JSON extraction) and defensive checks for missing fields.

Integrate TTS (Gemini TTS preview) for a quick audio read-through of scenes.
