# 社内情報特化型生成AI検索アプリ - AI Coding Assistant Guide

## Architecture Overview

This is a **Streamlit-based RAG (Retrieval-Augmented Generation) application** for internal document search and inquiry. The app features two distinct modes:
- **社内文書検索** (Document Search): Returns file locations for relevant documents
- **社内問い合わせ** (Internal Inquiry): Generates answers based on internal documents with source citations

### Core Components

- **`main.py`**: Entry point with Streamlit UI orchestration and error handling
- **`components.py`**: UI display functions, organized by mode-specific rendering
- **`utils.py`**: Business logic for LLM interactions and response processing
- **`initialize.py`**: Startup initialization including RAG setup and session management
- **`constants.py`**: Centralized configuration and prompt templates

### Data Architecture

Documents are organized under `data/` with recursive loading:
```
data/
├── MTG議事録/        # Meeting minutes by department
├── サービスについて/   # Service information  
├── 会社について/       # Company information
├── 顧客について/       # Customer information
└── 社員について/       # Employee information (CSV format)
```

## Development Patterns

### Session State Management
All dynamic state is managed through `st.session_state`:
```python
# Critical session state keys in initialize.py
st.session_state.messages      # Display conversation log
st.session_state.chat_history  # LLM conversation memory
st.session_state.retriever     # RAG retriever object
st.session_state.mode         # Current operating mode
```

### Error Handling Convention
Consistent error handling pattern throughout `main.py`:
```python
try:
    # Operation
except Exception as e:
    logger.error(f"{ct.ERROR_MESSAGE}\n{e}")
    st.error(utils.build_error_message(ct.ERROR_MESSAGE), icon=ct.ERROR_ICON)
    st.stop()
```

### Mode-Specific Processing
The app switches behavior based on `st.session_state.mode`:
- **社内文書検索**: Returns empty string `""` for matches, shows file paths with icons
- **社内問い合わせ**: Returns detailed answers with markdown formatting + source citations

### Japanese Text Handling
Critical Windows compatibility in `initialize.py`:
```python
def adjust_string(s):
    if sys.platform.startswith("win"):
        s = unicodedata.normalize('NFC', s)
        s = s.encode("cp932", "ignore").decode("cp932")
```

## Configuration & Dependencies

### Environment Setup
- Requires `.env` file with OpenAI API key
- Use `requirements_windows.txt` for Windows, `requirements_mac.txt` for Mac
- Key dependencies: `streamlit`, `langchain`, `openai`, `chromadb`

### Prompt Templates
Prompts are centralized in `constants.py` with mode-specific variations:
- `SYSTEM_PROMPT_DOC_SEARCH`: Returns `""` or `"該当資料なし"`
- `SYSTEM_PROMPT_INQUIRY`: Generates detailed markdown responses

### File Type Support
Defined in `constants.py`:
```python
SUPPORTED_EXTENSIONS = {
    ".pdf": PyMuPDFLoader,
    ".docx": Docx2txtLoader, 
    ".csv": lambda path: CSVLoader(path, encoding="utf-8")
}
```

## Key Development Workflows

### Adding New Document Types
1. Add loader to `SUPPORTED_EXTENSIONS` in `constants.py`
2. Ensure proper encoding for Japanese text
3. Test with `adjust_string()` for Windows compatibility

### Modifying LLM Behavior
1. Update prompt templates in `constants.py`
2. Adjust response processing in `components.py` display functions
3. Update response matching constants (`NO_DOC_MATCH_ANSWER`, `INQUIRY_NO_MATCH_ANSWER`)

### Debugging RAG Issues
- Check `initialize_retriever()` in `initialize.py` for document loading
- Verify chunk splitting parameters (`chunk_size=500, chunk_overlap=50`)
- Monitor logs in `./logs/application.log` with session ID tracking

### Running the Application
```bash
streamlit run main.py
```

## Critical Implementation Notes

- **File icons**: Use `utils.get_source_icon()` to determine appropriate icons for file types vs URLs
- **Conversation flow**: User messages and AI responses are stored separately in `st.session_state.messages` for display and `st.session_state.chat_history` for LLM context
- **Vector store**: Chroma DB is rebuilt on each app restart from documents in `data/` directory
- **Japanese support**: All text processing includes Unicode normalization and Windows encoding compatibility