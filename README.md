# EUC Project Setup Guide

## 25-09-2024

### Summary
1. Set up the DELL Laptop with the final project code.
2. Successfully tested voice input functionality.
3. Documented the process for setting up the project on any laptop.

### Prerequisites
1. **Download & Install:**
   - [VS Code](https://code.visualstudio.com/)
   - [Python 3.10](https://www.python.org/downloads/)
   - [GIT (64-bit)](https://git-scm.com/)
   - [HWinfo (Local U.S)](https://www.hwinfo.com/download/)
   - [Node.js 20 (LTS)](https://nodejs.org/)
   - [OLLAMA (Windows)](https://ollama.com/)
     - Verify installation by running: `ollama run llama3`.

2. **Restart the Laptop.**

### Project Setup
1. **Clone the Repository:**
   - Navigate to your preferred directory, e.g., `C:/projects/`.
   - Clone the repository:
     ```bash
     git clone <repository_url>
     ```

2. **Create `llm_models` Folder:**
   - Inside `projects`, create a folder named `llm_models`.

3. **Download Required Models:**
   - From HARSHA'S drive, download the following models into `llm_models`:
     - BGE Large
     - BGE Reranker Large
   - From the `EUC` folder in the drive, download:
     - speech5_tts
     - speech5_hifigan
     - base.pt
     - medium.en.pt

### Virtual Environment Setup
1. **Open the project in VS Code.**
2. Run the following commands:
   ```bash
   pip install venv
   python -m venv venv
   source venv/bin/activate  # For Linux/MacOS
   venv\Scripts\activate  # For Windows
   pip install -r requirements.txt
   ```

### Node.js Setup
1. **Open a terminal in VS Code.**
2. Navigate to the `euc-app-new` directory:
   ```bash
   cd euc-app-new
   npm i --force
   ```

### HWinfo Setup
1. **Configuration Steps:**
   - Disable everything except total power consumption and GPU memory usage.
   - Enable toggling hotkey in settings and set it as `Ctrl + Shift + .`.
   - Add HWinfo to startup.
   - Set logging time limit to 2 seconds.
   - Enable "write direct to disk" option.
   - Enable "auto start" in main settings.
   - Grant full control access to `C:/Program Files/HWinfo 64/` for the user.

### FFMPEG Setup
1. **Download & Install FFMPEG.**
2. Set up the path variable for FFMPEG.

---

# 18-10-2024

## API Connection for Query Suggestor

### `main.py`
```python
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
    return {"suggested_queries": suggested_queries, "asked_query": query}
```

### `model.py`
```python
from pydantic import BaseModel

class QueryModel(BaseModel):
    query: str
```

### `__init__.py`
```python
from .model import QueryModel
```

---

# 17-10-2024

## Local Environment Connection for Public Access

```python
import requests

url = "http://192.168.0.216:8000/suggest-queries/"
payload = {"query": "who are you?"}  # Correct dictionary format
headers = {'Content-Type': 'application/json'}

response = requests.post(url, json=payload, headers=headers)

print(response.status_code)
print(response.text)
```

---

# 12-11-2024

## Implemented Pytest

### Notes
1. Install dependencies:
   ```bash
   pip install pytest pytest-httpx pytest-asyncio
   ```
2. Structure tests in the `tests/` directory and name files starting with or ending in `test_`.

### Example Test Cases
```python
from fastapi.testclient import TestClient
from main import app  

client = TestClient(app)

def test_query_context_success():
    payload = {
        "query": "Example query",
        "query_class": "class_a"
    }
    response = client.post("/query-context", json=payload)
    assert response.status_code == 200

def test_query_context_missing_field():
    payload = {
        "query_class": "class_a"
    }
    response = client.post("/query-context", json=payload)
    assert response.status_code == 422

def test_query_context_invalid_data():
    payload = {
        "query": "Example query",
        "query_class": 12345 
    }
    response = client.post("/query-context", json=payload)
    assert response.status_code == 422

def test_query_context_empty_payload():
    response = client.post("/query-context", json={})
    assert response.status_code == 422
```

---

# 13-11-2024

## Implemented Try-Exception Block

```python
try:
    query = payload.query
    classified_response = classify_question(query).lower()
    if classified_response in ["org", "tools"]:
        response = query_rephraser_core(classified_response.lower(), query)
    else:
        response = query
    return QueryRephraserResponse(query=query, rephrased_query=response)
except Exception as e:
    raise HTTPException(status_code=500)
```

---

# 14-11-2024

## Pytest Integration Across All Microservices

---

# 28-11-2024

## Snyk Testing and Environment Commands

### Snyk Testing
- **Snyk Usage:**
  - Snyk is used for testing vulnerabilities.
  - Key commands:
    - `snyk test`: Tests for vulnerabilities.
    - `snyk monitor`: Monitors for vulnerabilities and provides continuous tracking.

### Environment Commands
- **`pip freeze`**:
  - Lists all packages in the current environment.
- **`pip freeze | grep openai`**:
  - Filters and lists packages related to OpenAI.

# 10-1-2025
## Webhook Implementation

  ![image](https://github.com/user-attachments/assets/d6578025-a1a2-4998-89d3-ede667a60209)
above is the structure for webhook 

.env file wil contain as below:-
  ![image](https://github.com/user-attachments/assets/231fc385-0fc5-46e1-a4c7-997797025c62)

don't forget to create a __init__.py file which could be black or filled with created function in specific file 

text_to_text.py will contain webhook calling  as below code snippet
   
from core.config import settings
from utils.webhook import call_webhook


    

webhook_url = settings.webhook_url
dry_run_folder_path = "/home/nvanjari/projects/euc_phase2/genai_services/Text-to-Text/data/ImgtoText_Small_Model_Outputs_20250107_102024"

def process_text_to_text(images_folder_path,output_folder_path):
    """
    Runs image_to_text_small , image_to_text_medium and image_to_text_large sequentially.
    Args:
        images_folder_path (str): Path to the folder containing images.
        output_folder_path (str): Path to the folder where outputs will be saved.
    Returns:
        dict: Results paths of created csv , which will be stored in output folder path.
        Include a custom exception that triggers if the process fails, displaying the message:
        "Insufficient memory detected [size]. The process will terminate."
    """
    try:
        if settings.dry_run:
            Small_Model_Metrics_CSV = dry_run_folder_path
            Small_Model_Plots = ""
        else:
            from services.ttt_small import process_text_to_text_small
            print("Running image_to_text_small...")
            Small_Model_Metrics_CSV, Small_Model_Plots = process_text_to_text_small(images_folder_path, output_folder_path)

        call_webhook(model_type="small", folder_path=Small_Model_Metrics_CSV)
        
    except Exception as e:
        error = "Insufficient memory detected [0-5GB]. The process will terminate."
        print(error)
        raise error    
  
    try:
        if settings.dry_run:
            Medium_Model_Metrics_CSV = dry_run_folder_path
            Medium_Model_Plots = ""
        else:
            from services.ttt_medium import process_text_to_text_medium
            print("Running image_to_text_medium...")
            Medium_Model_Metrics_CSV, Medium_Model_Plots = process_text_to_text_medium(images_folder_path, output_folder_path)

        call_webhook(model_type="medium", folder_path=Medium_Model_Metrics_CSV)
        

    except Exception as e:
        error = "Insufficient memory detected [5-16GB]. The process will terminate."
        print(error)
        raise error      

    try:
        if settings.dry_run:
            Large_Model_Metrics_CSV = dry_run_folder_path
            Large_Model_Plots = ""
        else:
            from services.ttt_large import process_text_to_text_large
            print("Running image_to_text_large...")
            Large_Model_Metrics_CSV, Large_Model_Plots = process_text_to_text_large(images_folder_path, output_folder_path)
            print("Completed image_to_text_large.")

        call_webhook(model_type="large", folder_path=Large_Model_Metrics_CSV)

    except Exception as e:
        error = "Insufficient memory detected [16-32GB]. The process will terminate."
        print(error)
        raise error     

    return  (Small_Model_Metrics_CSV, Medium_Model_Metrics_CSV,Large_Model_Metrics_CSV),(Small_Model_Plots,Medium_Model_Plots,Large_Model_Plots)

   ![image](https://github.com/user-attachments/assets/977286d6-e2ed-48d5-89b6-a84c45dea2eb)
   above is the example for above reference

   webhook.py is the main file where we will create and initialize webhook below it's shown how to create one
   ![image](https://github.com/user-attachments/assets/8f01f5dd-03ea-4990-bb68-6fb877cd86e5)
   ![image](https://github.com/user-attachments/assets/992e4248-9620-461f-8967-51cb1b91c33a)


# 15-01-2025

# Docker Tutorial from Scratch

## Step 1: What is Docker?
Docker is an open-source platform designed to:
1. Build applications using containers, which package all the dependencies (libraries, configurations, etc.) required for your application.
2. Run these containers consistently across different environments (your local machine, cloud, or production servers).

### Key Concepts:
- **Images**: A lightweight, stand-alone, and executable package containing everything needed to run a piece of software (code, runtime, libraries, and system tools). Think of it as a blueprint.
- **Containers**: Running instances of images. These are isolated environments where your application runs.
- **Docker Engine**: The core software that runs and manages containers.
- **Docker Hub**: A public repository where Docker images are stored.

---

## Step 2: Install Docker

### Installation
Follow these steps based on your operating system:

#### For **Windows**:
1. Download [Docker Desktop for Windows](https://www.docker.com/products/docker-desktop/).
2. Ensure **WSL 2 (Windows Subsystem for Linux 2)** is installed. Docker Desktop uses WSL 2 for its backend.
   - Follow [Microsoft's WSL 2 installation guide](https://learn.microsoft.com/en-us/windows/wsl/install).
3. Install Docker Desktop by running the installer.
4. During the installation, make sure you enable the **"Use WSL 2 instead of Hyper-V"** option.
5. Start Docker Desktop after installation.

#### For **macOS**:
1. Download [Docker Desktop for macOS](https://www.docker.com/products/docker-desktop/).
2. Run the `.dmg` file and follow the installation steps.
3. Grant Docker the necessary permissions during installation.

#### For **Linux**:
1. Use your package manager to install Docker. For example:
   - On **Ubuntu**:
     ```bash
     sudo apt-get update
     sudo apt-get install docker.io
     ```
   - On **CentOS**:
     ```bash
     sudo yum install docker
     ```
2. Start the Docker service:
   ```bash
   sudo systemctl start docker
   ```
3. Enable Docker to start on boot:
   ```bash
   sudo systemctl enable docker
   ```

---

## Step 3: Verify Installation

1. Open your terminal or command prompt.
2. Run:
   ```bash
   docker --version
   ```
   You should see a version number like `Docker version 24.xx.xx`.

3. Test Docker by running its hello-world image:
   ```bash
   docker run hello-world
   ```
   This will:
   - Pull the `hello-world` image from Docker Hub.
   - Create and start a container using this image.
   - Display a message in your terminal if successful.

---

## Step 4: Docker Basics

### Core Docker Commands
1. **Pull an image from Docker Hub**:
   ```bash
   docker pull <image-name>
   ```
   Example:
   ```bash
   docker pull nginx
   ```
   This pulls the `nginx` image (a lightweight web server).

2. **List images on your system**:
   ```bash
   docker images
   ```

3. **Run a container from an image**:
   ```bash
   docker run <image-name>
   ```
   Example:
   ```bash
   docker run nginx
   ```
   This will start an `nginx` server in a container.

4. **List running containers**:
   ```bash
   docker ps
   ```

5. **Stop a running container**:
   ```bash
   docker stop <container-id>
   ```
   Replace `<container-id>` with the ID shown in `docker ps`.

6. **Remove a container**:
   ```bash
   docker rm <container-id>
   ```

7. **Remove an image**:
   ```bash
   docker rmi <image-id>
   ```

---

## Step 5: Your First Custom Image

Let’s create a simple Dockerfile to build a custom image. A Dockerfile is a text file with instructions to create an image.

### Steps:
1. Create a folder for your project:
   ```bash
   mkdir my-docker-app
   cd my-docker-app
   ```

2. Create a `Dockerfile` in this folder:
   ```bash
   touch Dockerfile
   ```

3. Add the following content to the `Dockerfile`:
   ```dockerfile
   # Use an official Python runtime as a parent image
   FROM python:3.9-slim

   # Set the working directory in the container
   WORKDIR /app

   # Copy the current directory contents into the container
   COPY . /app

   # Install any needed packages specified in requirements.txt
   RUN pip install --no-cache-dir -r requirements.txt

   # Make port 80 available to the world outside this container
   EXPOSE 80

   # Define the command to run the app
   CMD ["python", "app.py"]
   ```

4. Create a simple Python app (`app.py`) in the same folder:
   ```python
   from flask import Flask
   app = Flask(__name__)

   @app.route("/")
   def home():
       return "Hello, Docker!"

   if __name__ == "__main__":
       app.run(host="0.0.0.0", port=80)
   ```

5. Add a `requirements.txt` file with Flask:
   ```
   Flask==2.0.1
   ```

6. Build your Docker image:
   ```bash
   docker build -t my-python-app .
   ```

7. Run the container:
   ```bash
   docker run -p 4000:80 my-python-app
   ```
   - Visit `http://localhost:4000` in your browser. You’ll see “Hello, Docker!”.

---

## Step 6: Next Steps
Once you’re comfortable running containers:
1. Learn about **Docker Compose** for multi-container applications.
2. Explore **Docker Volumes** for persistent data storage.
3. Study **Docker Networking** for container communication.
4. Practice pushing images to Docker Hub or a private registry.

---


# Docker and Virtual Machines (VMs)

## Docker from Scratch

### **1. What is Docker?**
Docker is an open-source platform designed to:
1. Build applications using containers, which package all the dependencies (libraries, configurations, etc.) required for your application.
2. Run these containers consistently across different environments (your local machine, cloud, or production servers).

#### **Key Concepts:**
- **Images**: A lightweight, stand-alone, and executable package containing everything needed to run a piece of software (code, runtime, libraries, and system tools). Think of it as a blueprint.
- **Containers**: Running instances of images. These are isolated environments where your application runs.
- **Docker Engine**: The core software that runs and manages containers.
- **Docker Hub**: A public repository where Docker images are stored.

### **2. Installing Docker**

#### For **Windows**:
1. Download [Docker Desktop for Windows](https://www.docker.com/products/docker-desktop/).
2. Ensure **WSL 2 (Windows Subsystem for Linux 2)** is installed. Docker Desktop uses WSL 2 for its backend.
   - Follow [Microsoft's WSL 2 installation guide](https://learn.microsoft.com/en-us/windows/wsl/install).
3. Install Docker Desktop by running the installer.
4. During the installation, enable the **"Use WSL 2 instead of Hyper-V"** option.
5. Start Docker Desktop after installation.

#### For **macOS**:
1. Download [Docker Desktop for macOS](https://www.docker.com/products/docker-desktop/).
2. Run the `.dmg` file and follow the installation steps.
3. Grant Docker the necessary permissions during installation.

#### For **Linux**:
1. Use your package manager to install Docker. For example:
   - On **Ubuntu**:
     ```bash
     sudo apt-get update
     sudo apt-get install docker.io
     ```
   - On **CentOS**:
     ```bash
     sudo yum install docker
     ```
2. Start the Docker service:
   ```bash
   sudo systemctl start docker
   ```
3. Enable Docker to start on boot:
   ```bash
   sudo systemctl enable docker
   ```

### **3. Verify Installation**

1. Open your terminal or command prompt.
2. Run:
   ```bash
   docker --version
   ```
   You should see a version number like `Docker version 24.xx.xx`.

3. Test Docker by running its hello-world image:
   ```bash
   docker run hello-world
   ```
   This will:
   - Pull the `hello-world` image from Docker Hub.
   - Create and start a container using this image.
   - Display a message in your terminal if successful.

### **4. Core Docker Commands**

| Command                         | Description                                       |
|---------------------------------|---------------------------------------------------|
| `docker pull <image-name>`      | Pull an image from Docker Hub.                   |
| `docker images`                 | List images on your system.                      |
| `docker run <image-name>`       | Run a container from an image.                   |
| `docker ps`                     | List running containers.                         |
| `docker stop <container-id>`    | Stop a running container.                        |
| `docker rm <container-id>`      | Remove a container.                              |
| `docker rmi <image-id>`         | Remove an image.                                 |

### **5. Dockerfile Example**

Create a custom image with a Dockerfile:

#### Dockerfile:
```dockerfile
# Use an official Python runtime as a parent image
FROM python:3.9-slim

# Set the working directory in the container
WORKDIR /app

# Copy the current directory contents into the container
COPY . /app

# Install any needed packages specified in requirements.txt
RUN pip install --no-cache-dir -r requirements.txt

# Make port 80 available to the world outside this container
EXPOSE 80

# Define the command to run the app
CMD ["python", "app.py"]
```

#### Python Application (`app.py`):
```python
from flask import Flask
app = Flask(__name__)

@app.route("/")
def home():
    return "Hello, Docker!"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=80)
```

#### Build and Run:
1. Build the image:
   ```bash
   docker build -t my-python-app .
   ```
2. Run the container:
   ```bash
   docker run -p 4000:80 my-python-app
   ```
   Visit `http://localhost:4000` to see "Hello, Docker!".

---

## Docker vs Virtual Machines (VMs)

### **1. Architecture**

#### **Docker (Containers):**
- Containers share the **host operating system's kernel** and isolate applications at the **process level**.
- Lightweight and fast.

#### **Virtual Machines (VMs):**
- VMs virtualize hardware and run a **full guest operating system** on top of the host OS.
- Resource-heavy and slower startup.

### **2. Resource Usage**

| Feature             | Docker (Containers)       | Virtual Machines (VMs) |
|---------------------|---------------------------|-------------------------|
| **Size**            | MBs                       | GBs                     |
| **Startup Time**    | Seconds                   | Minutes                 |
| **Performance**     | Near-native               | Slower                  |

### **3. Isolation**

| Feature             | Docker                    | VMs                     |
|---------------------|---------------------------|-------------------------|
| **Level**           | Process-level isolation   | Full OS-level isolation |
| **Security**        | Moderate (shared kernel)  | High (separate OS)      |

### **4. Portability**

| Feature             | Docker                    | VMs                     |
|---------------------|---------------------------|-------------------------|
| **Portability**     | High                      | Moderate                |
| **Use Case**        | Microservices, CI/CD      | Legacy apps, multi-OS   |

---



