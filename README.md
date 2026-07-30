# 🎬 AI Video Assistant

> **End-to-end meeting intelligence** — paste a YouTube URL or drop a local video/audio file and get a full transcript, structured summary, action items, key decisions, open questions, and an interactive Q&A chatbot — all powered by local Whisper ASR and Mistral AI.

---

## ✨ Features

| Feature | Details |
|---|---|
| 🔗 **YouTube & Local File Support** | Accepts any YouTube URL or local `.mp4`, `.wav`, `.mp3`, `.m4a` file |
| 🎙️ **Multilingual Transcription** | OpenAI Whisper with English + Hinglish support |
| 📋 **AI Summary** | LangChain LCEL map-reduce chain — chunks transcript → partial summaries → final bullet-point summary |
| 🏷️ **Auto Title Generation** | LLM-generated professional meeting title (max 8 words) |
| ✅ **Action Item Extraction** | Task, owner, deadline from the transcript |
| 🔑 **Key Decisions** | Numbered list of decisions made in the meeting |
| ❓ **Open Questions** | Unresolved topics flagged for follow-up |
| 💬 **RAG Chat Interface** | Ask anything about the transcript — powered by ChromaDB + `all-MiniLM-L6-v2` embeddings |

---

## 🏗️ Architecture

```
Input (YouTube URL / Local File)
        │
        ▼
┌─────────────────────┐
│   Audio Processor   │  yt-dlp download → pydub → WAV → 10-min chunks
└─────────────────────┘
        │
        ▼
┌─────────────────────┐
│     Transcriber     │  OpenAI Whisper (small) — per chunk → join
└─────────────────────┘
        │
        ├──────────────────────────────────────────────┐
        ▼                                              ▼
┌─────────────────┐                        ┌──────────────────────┐
│   Summarizer    │  LangChain LCEL        │    Vector Store      │
│   (map-reduce)  │  mistral-small-latest  │  ChromaDB            │
│   3000-tok chunk│  → bullet summary      │  all-MiniLM-L6-v2    │
└─────────────────┘                        │  500-tok chunks      │
        │                                  │  top-k=4 retrieval   │
        ▼                                  └──────────────────────┘
┌─────────────────┐                                   │
│   Extractor     │  3 parallel chains:               ▼
│  - Action Items │  Action items              ┌──────────────┐
│  - Decisions    │  Key decisions             │  RAG Chain   │
│  - Questions    │  Open questions            │  LCEL + LLM  │
└─────────────────┘                            └──────────────┘
        │                                              │
        └──────────────────┬───────────────────────────┘
                           ▼
                  ┌─────────────────┐
                  │  Streamlit UI   │
                  │  Dark theme     │
                  │  Live pipeline  │
                  │  status + chat  │
                  └─────────────────┘
```

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| **ASR / Transcription** | [OpenAI Whisper](https://github.com/openai/whisper) (`small` model) |
| **Audio Processing** | yt-dlp, pydub, FFmpeg |
| **LLM** | Mistral AI (`mistral-small-latest`) via LangChain |
| **Orchestration** | LangChain LCEL — `RunnablePassthrough`, `RunnableLambda` |
| **Summarization** | Map-reduce chain — 3,000-token chunks |
| **Embeddings** | `sentence-transformers/all-MiniLM-L6-v2` (runs on CPU) |
| **Vector Store** | ChromaDB — 500-token chunks, top-k=4 similarity search |
| **UI** | Streamlit — dark theme, live pipeline status, chat interface |
| **LLM API** | Mistral AI |

---

## ⚡ Quick Start

### Prerequisites
- Python 3.10+
- [FFmpeg](https://ffmpeg.org/download.html) installed and on `PATH`
- Mistral AI API key — get one free at [console.mistral.ai](https://console.mistral.ai)

### 1. Clone & Install

```bash
git clone https://github.com/sayedmizan1/video-assistant.git
cd video-assistant

python -m venv venv
venv\Scripts\activate          # Windows
# source venv/bin/activate     # macOS / Linux

pip install -r requirements.txt
```

### 2. Set Environment Variables

Create a `.env` file in the project root:

```env
MISTRAL_API_KEY=your_mistral_api_key_here
WHISPER_MODEL=small
```

> **Available Whisper models:** `tiny`, `base`, `small`, `medium`, `large`
> `small` is the recommended balance of speed and accuracy.

### 3. Run

```bash
streamlit run app.py
```

Open [http://localhost:8501](http://localhost:8501) in your browser.

---

## 🎯 Usage

1. Paste a **YouTube URL** or enter a **local file path** (`.mp4`, `.wav`, `.mp3`) in the sidebar
2. Select the **language** (`english` or `hinglish`)
3. Click **⚡ Analyse**
4. Watch the live pipeline status in the sidebar:
   - 🔊 Audio Processing
   - 📝 Transcription
   - 🏷️ Title Generation
   - 📋 Summarisation
   - 🔍 Extraction
   - 🧠 RAG Engine
5. When complete — read the summary, view extracted insights, and **chat with your meeting**

---

## 📁 Project Structure

```
video-assistant/
├── app.py                    # Streamlit UI
├── main.py                   # CLI pipeline runner
├── core/
│   ├── transcriber.py        # Whisper ASR — load model, transcribe chunks
│   ├── summarize.py          # LangChain map-reduce summarization + title gen
│   ├── extractor.py          # Action items / decisions / questions chains
│   ├── rag_engine.py         # LCEL RAG chain — retriever + LLM
│   └── vector_store.py       # ChromaDB build / load / retriever
├── utils/
│   └── audio_processor.py    # yt-dlp download, pydub conversion, chunking
├── requirements.txt
└── .env                      # API keys (not committed)
```

---

## 🔧 Pipeline Details

### Audio Processing
- YouTube URLs → `yt-dlp` downloads best audio → FFmpeg converts to `.wav` (16kHz mono)
- Local files → `pydub` converts to 16kHz mono `.wav`
- Audio split into **10-minute chunks** for Whisper memory efficiency

### Summarization (Map-Reduce)
- Transcript split into **3,000-token chunks** with 200-token overlap
- Each chunk summarized independently (map step)
- Partial summaries combined into a final professional bullet-point summary (reduce step)

### RAG Pipeline
- Transcript split into **500-token chunks** with 50-token overlap
- Chunks embedded with `all-MiniLM-L6-v2` (runs on CPU, no GPU needed)
- Stored in **ChromaDB** local vector database
- Retrieval: **top-k=4** cosine similarity search
- Context injected into `mistral-small-latest` with a strict "answer only from context" system prompt

---

## 📋 Requirements

See [`requirements.txt`](requirements.txt). Key dependencies:

```
openai-whisper        # Local ASR
langchain             # LLM orchestration
langchain-mistralai   # Mistral AI integration
langchain-chroma      # ChromaDB vector store
sentence-transformers # CPU embeddings
streamlit             # Web UI
yt-dlp                # YouTube audio download
pydub                 # Audio conversion & chunking
```

---

## 📄 License

MIT License — feel free to use and build on this project.

---

<p align="center">Built by <a href="https://github.com/sayedmizan1">Sayed Mizan Hussain</a></p>
