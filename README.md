# 🎬 AI Video Assistant

Turn any meeting, lecture, or YouTube video into a transcript, a summary, structured takeaways, and a chat interface you can ask questions to.

Paste a YouTube URL (or a local audio/video file path), and the assistant will:

- 🔊 Download and chunk the audio
- 📝 Transcribe it (English via local Whisper, Hinglish via Sarvam)
- 🏷️ Generate a title
- 📋 Write a summary
- 🔍 Extract **action items**, **key decisions**, and **open questions**
- 💬 Let you **chat with the transcript** using a RAG pipeline

It ships with both a **Streamlit web UI** and a **command-line interface**.

---

## ✨ Features

| Feature | Details |
|---|---|
| Input sources | YouTube URLs or local audio/video files |
| Languages | `english` (Whisper) and `hinglish` (Sarvam) |
| Summarisation | LLM-generated summary and title (Mistral via LangChain) |
| Extraction | Action items, key decisions, open questions |
| RAG chat | ChromaDB vector store + HuggingFace embeddings |
| UI | Dark-themed Streamlit app with live pipeline status |
| CLI | Run the full pipeline and chat from the terminal |

---

## 🧱 Pipeline

```
YouTube URL / file
        │
        ▼
 Audio processing  (yt-dlp, pydub, FFmpeg → chunks)
        │
        ▼
 Transcription     (Whisper for English / Sarvam for Hinglish)
        │
        ├──► Title generation
        ├──► Summary
        ├──► Action items · Key decisions · Open questions
        │
        ▼
 RAG engine        (ChromaDB + HuggingFace embeddings + Mistral)
        │
        ▼
 Chat with your meeting
```

---

## 📁 Project Structure

```
.
├── app.py                 # Streamlit web UI
├── main.py                # CLI entry point + run_pipeline()
├── test.py                # Quick script to test the pipeline without RAG
├── Requirements.txt       # Python dependencies
├── .gitignore             # Ignores .env
├── .env                   # API keys (you create this, never commit it)
├── core/
│   ├── transcriber.py     # transcribe_all()
│   ├── summarizer.py      # summarize(), generate_title()
│   ├── extractor.py       # extract_action_items(), extract_key_decisions(), extract_questions()
│   └── rag_engine.py      # build_rag_chain(), ask_question()
└── utils/
    └── audio_processor.py # process_input()
```

---

## ⚙️ Prerequisites

- **Python 3.10+**
- **FFmpeg** installed and available on your `PATH`
  - macOS: `brew install ffmpeg`
  - Ubuntu/Debian: `sudo apt install ffmpeg`
  - Windows: `choco install ffmpeg` or download from [ffmpeg.org](https://ffmpeg.org/download.html)
- A **Mistral API key** ([console.mistral.ai](https://console.mistral.ai))
- A **Sarvam API key** (only if you use the `hinglish` option)

---

## 🚀 Installation

```bash
# 1. Clone the repository
git clone <your-repo-url>
cd <your-repo-folder>

# 2. Create and activate a virtual environment
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate

# 3. Install dependencies
pip install -r Requirements.txt
```

> The first run will download the Whisper model and the HuggingFace embedding model, so it may take a few minutes.

---

## 🔑 Configuration

Create a `.env` file in the project root:

```env
MISTRAL_API_KEY=your_mistral_key_here
SARVAM_API_KEY=your_sarvam_key_here   # only needed for Hinglish
```

> Check the `core/` modules for the exact variable names they read. The `.env` file is already listed in `.gitignore`, so your keys won't be committed.

---

## ▶️ Usage

### Web app (Streamlit)

```bash
streamlit run app.py
```

Then in the sidebar:

1. Paste a YouTube URL or a local file path
2. Choose a language (`english` or `hinglish`)
3. Click **⚡ Analyse**

Once the pipeline finishes, you'll see the title, summary, transcript, action items, key decisions, and open questions, plus a chat box to ask questions about the content.

### Command line

```bash
python main.py
```

You'll be prompted for:

```
Enter YouTube URL or local file path:
Language (english/hinglish):
```

After the results are printed, you can chat with the transcript. Type `exit`, `quit`, or `q` to leave.

### Quick pipeline test

`test.py` runs transcription, summarisation, and extraction on a hard-coded YouTube URL (no RAG, no chat). Edit the `source` and `language` variables at the top of the file, then run:

```bash
python test.py
```

### Use it as a library

```python
from main import run_pipeline

result = run_pipeline("https://www.youtube.com/watch?v=...", language="english")

print(result["title"])
print(result["summary"])
print(result["action_items"])
```

`run_pipeline()` returns a dictionary with these keys:

| Key | Description |
|---|---|
| `title` | Generated title |
| `transcript` | Full transcript text |
| `summary` | Summary of the content |
| `action_items` | Extracted action items |
| `key_decisions` | Extracted key decisions |
| `open_questions` | Extracted open questions |
| `rag_chain` | Chain to pass to `ask_question(rag_chain, question)` |

---

## 🛠️ Tech Stack

- **Audio:** yt-dlp, pydub, ffmpeg-python
- **Speech-to-text:** OpenAI Whisper (local), Sarvam (Hinglish)
- **LLM orchestration:** LangChain (LCEL) + Mistral
- **RAG:** ChromaDB, sentence-transformers, langchain-huggingface
- **UI:** Streamlit
- **Utilities:** python-dotenv, tqdm, numpy, requests

---

## 🩺 Troubleshooting

- **`ffmpeg not found`**: Install FFmpeg and make sure it's on your `PATH`.
- **API key errors**: Confirm `.env` exists in the project root and that `load_dotenv()` runs before any `core/` imports (as in `main.py` and `test.py`).
- **Slow transcription**: Whisper runs locally. A GPU (CUDA) speeds it up considerably, and long videos take longer.
- **YouTube download fails**: Update yt-dlp with `pip install -U yt-dlp`.

---

## 🗺️ Roadmap

The dependency list already includes PDF/TXT export libraries (`reportlab`, `fpdf2`), so exporting results is a natural next feature.

---

## 📄 License

Add your license here (e.g. MIT).
