# Document Translation Demo

## Overview

The Document Translation Demo showcases the HP ZGX Nano AI Station's ability to perform real-time multilingual translation using Meta's NLLB-200 (No Language Left Behind) model. This demonstration highlights how enterprise-grade translation capabilities can run entirely on local hardware without requiring cloud services or external API dependencies.

The demo provides a web-based interface where users can translate text between English and 27 different languages, demonstrating the ZGX Nano's capacity for natural language processing workloads.

## Model Information

This demo uses the NLLB-200 distilled model from Meta:

- **Model**: facebook/nllb-200-distilled-600M
- **Parameters**: ~600 million (distilled version for efficient inference)
- **Capabilities**: Supports translation between 200 languages with high quality
- **Memory Requirements**: Approximately 2-4GB VRAM when loaded in float16 precision

## Supported Languages

The demo supports bidirectional translation between English and the following 27 languages:

| Language | Language | Language |
|----------|----------|----------|
| Arabic | French | Polish |
| Chinese | German | Portuguese |
| Croatian | Hungarian | Romanian |
| Czech | Indonesian | Russian |
| Danish | Italian | Swedish |
| Dutch | Japanese | Thai |
| Finnish | Korean | Turkish |
| | Malay | Ukrainian |
| | Norwegian | Vietnamese |
| | Norwegian Bokmal | |

## System Requirements

- HP ZGX Nano AI Station (or compatible NVIDIA GPU system)
- Ubuntu 22.04 or later
- Python 3.10 or later
- NVIDIA GPU with at least 4GB VRAM (8GB+ recommended)
- CUDA toolkit and drivers installed
- Network connectivity for initial model download

## Directory Structure

```
document-translation-demo/
├── backend/
│   ├── main.py              # FastAPI backend server
│   └── requirements.txt     # Python dependencies
├── frontend/
│   ├── index.html           # Web interface
│   └── hp_logo.png          # HP branding asset
├── document-translate-env/  # Python virtual environment
├── install.sh               # Installation script
├── start_demo_remote.sh     # Demo startup script
├── download_models.sh       # Model download script (S3)
└── README.md                # This documentation
```

## Installation

### Step 1: Create the Virtual Environment

Before running the installation script, create the required Python virtual environment:

```bash
python3 -m venv document-translate-env
```

### Step 2: Run the Installation Script

```bash
chmod +x install.sh
./install.sh
```

The installation script will activate the virtual environment and install all required Python dependencies.

### Step 3: Model Download Options

The NLLB-200 model can be obtained in two ways:

**Option A: Automatic Download from Hugging Face (Recommended)**

The model will download automatically from Hugging Face when you click "Load Models" in the web interface. This requires internet connectivity and will cache the model locally for future use.

**Option B: Pre-download from S3 (Air-gapped environments)**

If you have pre-staged the model in an S3 bucket, use the download script:

```bash
chmod +x download_models.sh
./download_models.sh
```

This downloads the model to the Hugging Face cache directory at `~/.cache/huggingface/hub/models--facebook--nllb-200-distilled-600M/`.

## Configuration

### SALES TEAM CALLOUT: Server IP Address Configuration

The demo requires configuration of the server IP address to allow access from other machines on the network.

**File to modify**: `frontend/index.html`

**Line to update** (Line 360):
```javascript
const API_URL = 'http://192.168.10.117:8000';
```

**Action required**: Replace `192.168.10.117` with the actual IP address of the HP ZGX Nano running the demo.

**How to find the server IP address**:
```bash
hostname -I | awk '{print $1}'
```

**Note**: The `start_demo_remote.sh` script automatically updates this value when launched. However, if you need to run the frontend separately or the automatic detection fails, manual configuration may be required.

### Port Configuration

The demo uses the following network ports:

| Service | Port | Purpose |
|---------|------|---------|
| Backend API | 8000 | FastAPI server handling translation requests |
| Frontend | 8080 | Web interface served via Python HTTP server |

Ensure these ports are not blocked by firewalls and are available on the system.

## Running the Demo

### Starting the Demo

```bash
chmod +x start_demo_remote.sh
./start_demo_remote.sh
```

The startup script performs the following actions:

1. Detects the server IP address automatically
2. Terminates any existing processes on ports 8000 and 8080
3. Activates the Python virtual environment
4. Starts the FastAPI backend server
5. Updates the frontend configuration with the detected IP address
6. Starts the frontend web server
7. Displays access URLs for the demo

### Accessing the Demo

Once the demo is running, access the web interface from any browser on the network:

```
http://[SERVER_IP]:8080
```

Replace `[SERVER_IP]` with the IP address displayed by the startup script.

## Demo Workflow

### Step 1: Load the Translation Model

1. Open the web interface in your browser
2. Click the "Load Models" button in the status bar
3. Wait for the model to load (this may take 1-2 minutes on first run)
4. The status indicator will turn green when models are ready

### Step 2: Select a Language Pair

1. Use the "Select Language Pair" dropdown menu
2. Choose from English to another language, or from another language to English
3. Language pairs are organized into two groups for easy navigation

### Step 3: Enter or Select Text

1. Type or paste text into the "Source Text" area
2. Alternatively, click one of the "Sample" buttons to load pre-defined example text
3. There is no strict character limit, but shorter texts translate faster

### Step 4: Translate

1. Click the "Translate Text" button
2. Wait for the translation to complete (typically 1-5 seconds depending on text length)
3. The translated text appears in the "Translation" panel on the right


## Troubleshooting

### Models fail to load

**Symptoms**: Error message when clicking "Load Models" or timeout during loading.

**Possible causes and solutions**:

- **Insufficient GPU memory**: Ensure no other applications are using the GPU. Run `nvidia-smi` to check GPU memory usage.
- **CUDA not available**: Verify CUDA is installed correctly with `python3 -c "import torch; print(torch.cuda.is_available())"`.
- **Network issues**: If downloading from Hugging Face, ensure internet connectivity. Check if you need to configure proxy settings.

### Cannot access web interface from another machine

**Symptoms**: Browser shows "connection refused" or page does not load.

**Possible causes and solutions**:

- **Wrong IP address**: Verify the IP address in `frontend/index.html` matches the server's actual IP.
- **Firewall blocking ports**: Ensure ports 8000 and 8080 are open. On Ubuntu: `sudo ufw allow 8000` and `sudo ufw allow 8080`.
- **Services not running**: Check that both backend and frontend processes are active using `lsof -i:8000` and `lsof -i:8080`.

### Translation returns error

**Symptoms**: Error message appears instead of translated text.

**Possible causes and solutions**:

- **Models not loaded**: Ensure you clicked "Load Models" and waited for the green status indicator.
- **Unsupported language pair**: Verify the selected language pair is in the supported list.
- **Text too long**: Try with shorter text to rule out memory issues.

### Startup script fails

**Symptoms**: Error messages during `./start_demo_remote.sh` execution.

**Possible causes and solutions**:

- **Virtual environment not found**: Ensure `document-translate-env` directory exists in the demo folder. Re-run installation if needed.
- **Port already in use**: Another application may be using ports 8000 or 8080. The script attempts to kill existing processes, but manual intervention may be needed: `sudo lsof -ti:8000 | xargs kill -9`.

### Slow translation performance

**Symptoms**: Translations take longer than expected (more than 10 seconds for short text).

**Possible causes and solutions**:

- **CPU fallback**: The model may be running on CPU instead of GPU. Check GPU utilization with `nvidia-smi` during translation.
- **Thermal throttling**: Ensure the ZGX Nano has adequate cooling and is not throttling.
- **First inference**: The first translation after loading may be slower due to CUDA kernel compilation. Subsequent translations should be faster.

## Stopping the Demo

Press `Ctrl+C` in the terminal where `start_demo_remote.sh` is running. The script will automatically:

1. Terminate the backend API server
2. Terminate the frontend web server
3. Restore the original IP configuration in `index.html`

## Technical Notes

- The model runs in float16 precision to reduce memory usage while maintaining translation quality
- The backend uses FastAPI with CORS enabled to allow cross-origin requests from the frontend
- Translation uses beam search with 5 beams for improved output quality
- Maximum input length is 512 tokens; longer texts are automatically truncated
