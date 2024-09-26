# Documentation

# 25-9-2024

1. Set up the DELL Laptop with the final code.
2. Tried to run voice input on it .
3. Process to set-up the project on any laptop:-

**EUC Project Setup Guide**

### Prerequisites:
1. **Download & Install:**
   - [VS Code](https://code.visualstudio.com/)
   - [Python 3.10 (specifically)](https://www.python.org/downloads/)
   - [GIT (64-bit)](https://git-scm.com/)
   - [HWinfo (Local U.S)](https://www.hwinfo.com/download/)
   - [Node.js 20 (LTS)](https://nodejs.org/)
   - [OLLAMA (Windows)](https://ollama.com/)  
     - To verify installation: Run `ollama run llama3` in the terminal.

2. **Restart the Laptop.**

### Project Setup:
1. **Clone the Repository:**
   - Navigate to `C:/projects/` (or your preferred directory).
   - Clone the repo into a new folder:
     ```bash
     git clone <repository_url>
     ```

2. **Create `llm_models` Folder:**
   - Inside the `projects` folder, create a folder named `llm_models`.

3. **Download the Required Models:**
   - Inside `llm_models`, download the following from drive HARSHA'S DM:
     - BGE Large
     - BGE Reranker Large
     - From `EUC` folder in drive, download:
       - speech5_tts
       - speech5_hifigan
       - base.pt
       - medium.en.pt

### Virtual Environment Setup:
1. **Open the project in VS Code.**
2. In the terminal, run the following commands:
   ```bash
   pip install venv
   python -m venv venv
   venv/Scripts/activate  # Add this address to your current directory to activate the venv folder
   pip install -r requirements.txt
   ```

### Node.js Setup:
1. **Open a new terminal in VS Code.**
2. Navigate to the euc-app-new directory:
   ```bash
   cd euc-app-new
   npm i --force
   ```

### HWinfo Setup:
1. **Configure HWinfo:**
   - Disable everything except total power consumption and GPU memory usage.
   - Open settings, Enable toggling hot key then set up the hotkey as `Ctrl + Shift + .`
   - Add HWinfo to startup.
   - Set logging time limit 2 sec.
   - Enable write direct to disk option.
   - Open main setting , Enable the auto start option 
   - Give full control access to `C:/Program Files/HWinfo 64/` for the user.

### FFMPEG Setup:
1. **Download & Install FFMPEG.**
2. Set up the path variable for FFMPEG.

