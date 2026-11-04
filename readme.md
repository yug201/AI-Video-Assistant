# 🎬 AI Video Assistant (Meeting Intelligence)

Hey there! 👋 This is an AI-powered meeting and video assistant that I built to save time watching long videos, lectures, and meetings. It automatically transcribes audio, summarizes the entire discussion, extracts action items and key decisions, and lets you ask questions directly to your video through an interactive RAG (Retrieval-Augmented Generation) chat.

---

## 💡 What This Project Does

Instead of sitting through 1-2 hour long videos or meetings, you just paste a YouTube link (or give a local audio/video file) and select the language. The tool will:
1. Extract and process the audio automatically.
2. Transcribe spoken speech to text (supports English and Hindi/Hinglish).
3. Generate a clean title and a structured bulleted summary.
4. Extract actionable items (with tasks, assignees, and deadlines).
5. Extract key decisions and unresolved questions.
6. Let you chat with the video using a local RAG pipeline to answer any specific question based only on what was discussed.

---

## 🛠️ What I Did & Architecture

Here is how I designed and built this system from scratch:

### 1. Audio Processing & Chunking (`utils/audio_processor.py`)
- Used `yt-dlp` to download audio streams directly from YouTube URLs.
- Used `pydub` and `FFmpeg` to standardize audio (converting files into mono 16kHz WAV format).
- Added audio chunking (10-minute slices) to handle long video files without running into memory or API size constraints.

### 2. Speech-to-Text & Translation (`core/transcriber.py`)
- **English**: Ran OpenAI's **Whisper** model locally for fast and accurate offline speech recognition.
- **Hinglish / Hindi**: Integrated **Sarvam AI**'s speech-to-text translation API. Because Sarvam's synchronous API has a 30-second audio limit, I implemented custom chunk slicing (25-second micro-chunks) that sends pieces sequentially and merges them cleanly into an English transcript.

### 3. Summarization & Title Generation (`core/summarizer.py`)
- Built an LLM chain with **Mistral AI** (`mistral-small-latest`) via **LangChain LCEL**.
- Implemented a **Map-Reduce** pattern: large transcripts are split using `RecursiveCharacterTextSplitter` (chunk size 3000, overlap 200), each piece is summarized individually, and then combined into a cohesive final executive summary.
- Added automatic short title generation based on the conversation context.

### 4. Structured Information Extraction (`core/extractor.py`)
- Created specialized prompt chains to pull out structured meeting takeaways:
  - **Action Items**: Task description, responsible owner, and deadline.
  - **Key Decisions**: Crucial choices agreed upon during the discussion.
  - **Open Questions**: Unresolved topics needing follow-up.

### 5. RAG Engine & Meeting Chat (`core/vector_store.py` & `core/rag_engine.py`)
- Used `sentence-transformers` (`all-MiniLM-L6-v2`) to generate vector embeddings on CPU.
- Stored chunked embeddings locally in a persistent **ChromaDB** vector database.
- Built a LangChain LCEL RAG pipeline with strict context grounding (if the information isn't in the video, it clearly says so rather than hallucinating).

### 6. Interactive User Interface (`app.py` & `main.py`)
- Designed a modern, dark-themed **Streamlit** dashboard with live pipeline step trackers, custom badges, expandable transcript viewer, and a chat interface.
- Also provided a clean terminal CLI (`main.py`) for quick command-line usage.

---

## ⚙️ Tech Stack Used

- **Language:** Python 3.10+
- **Audio Processing:** `yt-dlp`, `pydub`, `ffmpeg`
- **Transcription:** OpenAI Whisper (Local) & Sarvam AI API (Hinglish/Hindi translation)
- **LLM & Orchestration:** Mistral AI (`mistral-small-latest`), LangChain (LCEL)
- **Vector DB & Embeddings:** ChromaDB, HuggingFace (`all-MiniLM-L6-v2`)
- **Frontend UI:** Streamlit (Custom CSS dark theme)

---

## 🚀 How to Run the Project

### 1. Clone or Download the Repository
https://github.com/yug201/AI-Video-Assistant

### 2. Set Up Virtual Environment (Recommended)
```bash
python -m venv venv
# On Windows:
venv\Scripts\activate
# On Linux/macOS:
source venv/bin/activate
```

### 3. Install Dependencies
Make sure you have [FFmpeg](https://ffmpeg.org/download.html) installed on your system, then install the required Python packages:
```bash
pip install -r Requirements.txt
```

### 4. Configure Environment Variables
Create a `.env` file in the project root with your API keys:
```env
MISTRAL_API_KEY=your_mistral_api_key_here
SARVAM_API_KEY=your_sarvam_api_key_here     # Required only if transcribing Hinglish/Hindi
WHISPER_MODEL=small                        # Options: tiny, base, small, medium, large
```

### 5. Launch the Application

#### Option A: Run the Streamlit Web App (Recommended)
```bash
streamlit run app.py
```
Open `http://localhost:8501` in your browser.

#### Option B: Run in Terminal (CLI Mode)
```bash
python main.py
```

---

## 📁 Project Structure

```text
├── app.py                  # Streamlit web UI with modern dark theme
├── main.py                 # CLI interface for running the pipeline in terminal
├── test.py                 # Quick testing script
├── Requirements.txt        # Python package dependencies
├── .env                    # Environment variables (API keys)
├── utils/
│   └── audio_processor.py  # YouTube downloader, audio conversion (16kHz mono), and chunking
└── core/
    ├── transcriber.py      # Whisper (English) and Sarvam AI (Hinglish) transcription
    ├── summarizer.py       # Map-Reduce summarization and title generator
    ├── extractor.py        # Action items, key decisions, and questions extractor
    ├── vector_store.py     # ChromaDB and HuggingFace embeddings
    └── rag_engine.py       # LangChain LCEL RAG chain for video Q&A
```

---

## 📝 Notes & Tips
- For long videos, the first run might take a couple of minutes to process audio and generate embeddings.
- Whisper runs locally on your CPU or GPU. If you have an NVIDIA GPU with CUDA enabled, Whisper will automatically run significantly faster.