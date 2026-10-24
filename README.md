# AI Video Assistant

A Streamlit app that turns meeting or video audio into a transcript, summary, and follow up notes. You can provide a YouTube URL or a local audio/video file, then ask questions about its contents in a chat.

## Features

- Transcribe audio from YouTube videos or local files.
- Use Whisper for English transcription and Sarvam for Hinglish transcription translated to English.
- Generate a title and summary with Mistral AI.
- Extract action items, key decisions, and open questions.
- Chat with the transcript using retrieval augmented generation (RAG).

## Requirements

- Windows, macOS, or Linux
- Python 3.10 or newer
- FFmpeg installed and available on your `PATH`
- API keys for Mistral, and for Sarvam if you want Hinglish transcription
- `uv` for the setup commands below (or use equivalent `pip` commands)

Whisper and the Hugging Face embedding model may download model files the first time they are used. English transcription runs locally and can be slow on a CPU.

## Setup on Windows

Open PowerShell in the project folder and run:

```powershell
cd "C:\Users\smaur\OneDrive\Desktop\VIDEO AGENT"
uv venv
.\.venv\Scripts\Activate.ps1
uv pip install -r Requirements.txt
```

If PowerShell blocks activation, you can run the app directly with the virtual environment's Python instead, or allow scripts for only the current PowerShell session:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy RemoteSigned
.\.venv\Scripts\Activate.ps1
```

FFmpeg must be installed separately from Python packages. After installing it, open a new terminal and check that `ffmpeg -version` works.

## Configure API keys

Create or edit a `.env` file in the project root. Add your own keys; do not share or commit this file:

```env
MISTRAL_API_KEY=your_mistral_api_key
SARVAM_API_KEY=your_sarvam_api_key
WHISPER_MODEL=small
SARVAM_STT_MODEL=saaras:v2.5
```

`MISTRAL_API_KEY` is used for summaries, extracted notes, and RAG chat. `SARVAM_API_KEY` is needed only for Hinglish transcription. The `.gitignore` excludes `.env` and generated downloads. Never paste real API keys into source code or publish them on GitHub. If a key has been exposed, revoke it and create a replacement.

## Run the app

From the project folder, run this in the terminal:

```powershell
streamlit run app.py
```

Streamlit will display a local URL in the terminal and usually open it in your browser. In the app, enter a YouTube URL or choose a local media file, select the transcription language, and start the analysis.

To run the command line version instead, use:

```powershell
python main.py
```

## Project layout

```text
VIDEO AGENT/
├── app.py                  # Streamlit user interface
├── main.py                 # Command line entry point
├── Requirements.txt        # Python dependencies
├── core/
│   ├── extractor.py        # Action items, decisions, and questions
│   ├── rag_engine.py       # Transcript question answering
│   ├── summarizer.py       # Titles and summaries
│   ├── transcriber.py      # Whisper and Sarvam transcription
│   └── vector_store.py     # Embeddings and vector database
├── utils/
│   └── audio_processor.py # Downloads, converts, and chunks audio
├── downloads/              # Downloaded or processed media
├── vector_db/              # Local RAG database, created as needed
└── .env                    # Local API keys; do not commit
```

## Troubleshooting

- **FFmpeg is not recognized:** install FFmpeg, add its `bin` folder to `PATH`, and restart the terminal.
- **Mistral returns `401`:** check that `MISTRAL_API_KEY` is valid and active in `.env`.
- **Mistral returns `429` / rate limit exceeded:** the API workspace has reached a usage or rate limit. Check its usage and limits, then retry later or reduce requests.
- **Sarvam returns `400` or `403`:** check the Sarvam key and model name in `.env`. The model used by this project defaults to `saaras:v2.5`.
- **Whisper says a model is unavailable:** use a supported model name such as `small`, `small.en`, `medium`, or `base` for `WHISPER_MODEL`. The selected model may need to download on first use.
- **Missing `langchain_chroma` or `langchain_text_splitters`:** install the missing packages in the active virtual environment with `uv pip install langchain-chroma langchain-text-splitters`.
- **Whisper reports that FP16 is unsupported on CPU:** this is a normal warning; it falls back to FP32.

