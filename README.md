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



# 18-10-24
**API connection for query suggestor**
**main file:**
from fastapi import FastAPI
from models import QueryModel
from typing import List
import ollama

app = FastAPI()

def generate_similar_queries(input_question: str) -> List[str]:
        prompt = f"generate 5 different versions of below input question: '{input_question}'."
        response = ollama.chat(model='llama3', messages=[
            {"role": "system", "content": "You are an AI Assistant for suggesting similar queries."},
            {"role": "user", "content": prompt} 
        ])
        suggested_queries = response['message']['content'].replace('*', '').strip().split('\n')
        suggested_queries_list = [line.split('. ', 1)[1] for line in suggested_queries if line.strip().startswith(('1.', '2.', '3.', '4.', '5.'))]
        return suggested_queries_list

@app.post("/suggest-queries/")
def suggest_queries(query_model: QueryModel):
    query = query_model.query
    suggested_queries = generate_similar_queries(query)
    return {"suggested_queries": suggested_queries,"asked_query":query}   

**model**
from pydantic import BaseModel

class QueryModel(BaseModel):
    query: str

**__init__.py**
from .model import QueryModel



# 17-10-24
**how to connect local env to public accesible**

import requests
url="http://192.168.0.216:8000/suggest-queries/"

payload = {'query': 'who are you?'}  # This is the correct dictionary format
headers = {'Content-Type': 'application/json'}  # If the API expects JSON
response = requests.post(url, json=payload, headers=headers)

print(response.status_code)
print(response.text)

