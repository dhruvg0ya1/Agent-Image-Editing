#  Moleculyst: Agentic Image Editing

An agentic image editing pipeline inspired by the [Agent Banana paper](https://arxiv.org/abs/2602.09084) (arXiv:2602.09084). Implements **Image Layer Decomposition (ILD)** for high-fidelity, localized edits with seamless blending.

[![Python](https://img.shields.io/badge/python-3.10%2B-blue?style=flat-square&logo=python&logoColor=white)](https://www.python.org/)
[![Gemini](https://img.shields.io/badge/Gemini-2.5%20Flash-4285F4?style=flat-square&logo=googlegemini&logoColor=white)](https://ai.google.dev/)
[![Florence-2](https://img.shields.io/badge/Florence--2-grounding-FFD21E?style=flat-square&logo=huggingface&logoColor=black)](https://huggingface.co/microsoft/Florence-2-large)
[![Docker](https://img.shields.io/badge/docker-ready-2496ED?style=flat-square&logo=docker&logoColor=white)](./Dockerfile)
[![License](https://img.shields.io/badge/license-MIT-green?style=flat-square)](./LICENSE)

**[Live demo on Hugging Face Spaces](https://huggingface.co/spaces/vansh7266/Agent_Crop_M)**

## ✨ Features

- **Ground-First Local Inpainting** — Locates the target object *before* editing, crops a local patch, and sends it to Gemini for context-aware editing
- **Laplacian Pyramid Blending** — Multi-band blending (Burt & Adelson, 1983) seamlessly fuses edited patches back into the original
- **LLM Grounding Advisor** — Gemini 2.5 Flash reasons about spatial context and disambiguates targets (e.g., "glasses" → drinking glasses vs eyewear)
- **Interactive BBox Editor** — Draw custom bounding boxes on the original image to fine-tune the edit region
- **Custom Reconstruction Instructions** — Type specific instructions (e.g., "fill with table texture") for each recompose
- **Iterative Editing Loop** — Each output becomes the input for the next round — refine endlessly
- **Agentic Timeline UI** — Full transparency: reasoning, grounding phrases, spatial guidance, quality scores

## 🏗️ Architecture

```
User Instruction
    │
    ▼
┌──────────────────┐
│   LLM Planner    │ ── Parse instruction → plan steps
└────────┬─────────┘
         │
    ┌────▼────────────────────────────────────┐
    │           For each step:                │
    │                                         │
    │  1. GROUND (Florence-2 + LLM Advisor)   │
    │     └─ Find target on original image    │
    │                                         │
    │  2. CROP LOCAL PATCH                    │
    │     └─ bbox + 50% padding from original │
    │                                         │
    │  3. EDIT LOCALLY (Gemini)               │
    │     └─ Model sees surrounding context   │
    │     └─ Acts as inpainter                │
    │                                         │
    │  4. BLEND BACK (Laplacian Pyramid)      │
    │     └─ Multi-band frequency blending    │
    │     └─ Low-freq: wide color smoothing   │
    │     └─ High-freq: crisp edges           │
    └─────────────────────────────────────────┘
         │
         ▼
    Final Image (original pixels preserved outside edit region)
```

## 🚀 Quick Start

### Prerequisites

- Python 3.10+
- [Gemini API key](https://aistudio.google.com/)

### Setup

```bash
# Clone and enter the project
git clone https://github.com/dhruvg0ya1/Agent-Image-Editing.git
cd Agent-Image-Editing

# Create virtual environment
python -m venv .venv
source .venv/bin/activate

# Install dependencies
pip install -e .

# Configure API key
echo "GEMINI_API_KEY=your-key-here" > .env
```

### Run

```bash
python -m agent_banana.server --host 127.0.0.1 --port 8011
```

Open **http://127.0.0.1:8011** in your browser.

## 🎯 Usage

1. **Upload an image** and type an instruction (e.g., "remove the glasses from the table")
2. **Review** the agentic timeline: LLM reasoning → grounding → local edit → composition
3. **Adjust the bounding box** by drawing on the original image
4. **Type custom instructions** in the text field (e.g., "fill the area with wooden texture")
5. **Click Re-compose** — a new editor appears on the result for further refinement
6. **Iterate** until satisfied

## 📁 Project Structure

```
src/agent_banana/
├── server.py                 # Web UI + API endpoints
├── pipeline.py               # ILD pipeline: ground → crop → edit → blend
├── vision.py                 # Laplacian pyramid blending + image utilities
├── nano_banana.py            # Gemini API client
├── llm_grounding_advisor.py  # LLM spatial reasoning advisor
├── vlm_localizer.py          # Florence-2 grounding
├── targeting.py              # Target classification + bbox refinement
├── planning.py               # RL-based edit planner
├── react_executor.py         # ReAct loop: think -> act -> observe
├── tool_registry.py          # Tool schemas the agent may call
├── quality.py                # Quality evaluation judge
├── vlm_critic.py             # VLM-based quality scoring
├── seam_detector.py          # Blend-seam detection on recomposed output
├── models.py                 # Data models (BoundingBox, StepResult, etc.)
├── memory.py                 # Context folding + session storage
├── cli.py                    # Command-line entry point
└── config.py                 # Environment configuration

tests/
└── test_agent_banana.py      # Unit tests

examples/sessions/            # Recorded agent runs: reasoning, edits, folded context
```

## Docker

```bash
docker build -t agent-banana .
docker run -p 7860:7860 -e GEMINI_API_KEY=your-key-here agent-banana
```

## Tests

```bash
pip install -e ".[dev]"
pytest -q
```

## Example Sessions

`examples/sessions/*.json` are real recorded runs. Each captures the full agent
trace for one instruction: the parsed edit plan, the grounding phrases, the
bounding boxes chosen, per-step quality scores, and the folded context carried
into the next turn. Useful for seeing how the ReAct loop behaves without
running the model yourself.

## 🔧 Configuration

| Environment Variable | Description | Default |
|---------------------|-------------|---------|
| `GEMINI_API_KEY` | Google Gemini API key | *required* |
| `AGENT_BANANA_IMAGE_MODEL` | Image editing model | `gemini-2.5-flash-preview-04-17` |
| `AGENT_BANANA_ADVISOR_MODEL` | Grounding advisor model | `gemini-2.5-flash-preview-04-17` |

## 📄 Key Concepts from the Paper

### Image Layer Decomposition (ILD)
Instead of editing the full image (which causes color drift and detail loss), ILD:
- Crops the target region with context padding
- Edits only the local patch (model naturally matches surrounding pixels)
- Blends back using Gaussian/Laplacian pyramids

### Context Folding
Compresses interaction history into structured memory across three levels:
- **Asset Level**: Lightweight image state nodes
- **Execution Level**: Transient tool context for error recovery
- **Planning Level**: Persistent memory of verified edit paths

## 📜 License

MIT

## Acknowledgments

- [Agent Banana Paper](https://arxiv.org/abs/2602.09084) — Ye et al., 2026
- [Florence-2](https://huggingface.co/microsoft/Florence-2-large) — Microsoft
- [Gemini API](https://ai.google.dev/) — Google DeepMind
