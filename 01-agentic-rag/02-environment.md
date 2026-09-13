---
video_url: "https://www.youtube.com/watch?v=3U4gBrmkZyM&list=PL3MmuxUbc_hLZFNgSad56pDBKK8KO0XIv"
---
# Environment (**)

Video: [Watch this lesson](https://www.youtube.com/watch?v=3U4gBrmkZyM&list=PL3MmuxUbc_hLZFNgSad56pDBKK8KO0XIv)

For this module, all you need is Python with Jupyter.

## Prerequisites

This modified lesson uses a Docker setup so that our environment can run almost totally offline. Once you install Docker Desktop successfully, you need the following:

- Python (3.14 or later)
- LocalAI image
- Qwen GGUF file

## Creating the project

We'll start from scratch with no cloning needed - you'll create the
project yourself, step by step, either locally or on GitHub Codespaces.

## Creating the project locally

First, install uv - it's a Python package manager, and I switched all my
projects to it because it's fast and convenient. Once I started using
it, I never wanted to go back.

On Mac or Linux:

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

On Windows:

```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

(You can also use `pip install uv` if you prefer.)

Create an empty folder for the project and initialize it:

```bash
mkdir llm-zoomcamp-2026-code
cd llm-zoomcamp-2026-code
uv init
```

This creates a `pyproject.toml` and a basic project structure.

## Creating the project on GitHub Codespaces

We suggest using Codespaces because everyone gets the same Ubuntu,
Python, and Docker. That makes it easier to help each other when
problems come up.

Setup:

- Create a new repo on GitHub. Name it whatever you want, for example
  `llm-zoomcamp-2026-code` or `introduction-to-rag`, and add a README.
- Open the repo, click the green `<> Code` button, switch to the
  Codespaces tab, and create a codespace.

You now have a remote environment running in Codespaces. By default it
opens an in-browser editor, but you can connect VS Code on your desktop
for a better experience. Click `Codespaces` in the bottom-left corner and
pick "Open in Visual Studio Code Desktop" from the dropdown.

Once VS Code opens, press `` ctrl+` `` to bring up the terminal and
initialize the project the same way as locally:

```bash
pip install uv
uv init
```

## Adding dependencies
Now add the dependencies we'll need:

```bash
uv add requests minsearch openai jupyter python-dotenv
```

This installs:

- `requests` - to fetch the FAQ dataset from the internet
- `minsearch` - a simple in-memory search engine for indexing and
  searching text
- `openai` - the OpenAI API client for calling the LLM
- `jupyter` - the notebook environment where we'll write and run code
- `python-dotenv` - to load API keys from a `.env` file

Copy the following codes to `docker-compose.yaml`:
```
services:
  local-ai:
    image: localai/localai:latest-cpu
    container_name: local-ai
    restart: always
    ports:
      - "8080:8080"
    environment:
      - MODELS_PATH=/models
      - THREADS=4
    volumes:
      - ./models:/models
      - ./data:/data
    healthcheck:
      test: [ "CMD", "curl", "-f", "http://localhost:8080/v1/models" ]
      interval: 15s
      timeout: 10s
      retries: 5

  llm-zoomcamp:
    build: . # This tells Docker to use the Dockerfile in the current folder
    container_name: llm-zoomcamp
    volumes:
      - .:/app # Mount your code into the container
    tty: true # Keeps the container open (like a terminal)
    stdin_open: true # Keeps the container open (like a terminal)
    ports:
      - "8888:8888" # Expose port 8000 for your application
```

Manually download a GUFF file of your chosen model and save it in your project folder's `models` directory. For this instance, we can download Qwen 2.5 from HuggingFace: [`Qwen/Qwen2.5-1.5B-Instruct-GGUF`](https://huggingface.co/Qwen/Qwen2.5-1.5B-Instruct-GGUF/tree/main) 

Then, create a file called `qwen-1.5b-instruct.yaml` and add the following code:
```
name: qwen-1.5b-instruct
parameters:
  model: qwen2.5-1.5b-instruct-q4_k_m.gguf
context_size: 4096
backend: llama-cpp

template:
  chat: |
    {{.Input}}
  chat_message: |
    <|im_start|>{{.RoleName}}
    {{.Content}}<|im_end|>
```
Save this file under the `models` director of your project folder. This file acts as your model config file. Basically, this YAML file tells LocalAI: *"Take this Qwen GUFF file, run it using the llama-cpp engine, give it a 4096-token memory, and use the Qwen formatting rules so it can chat properly."*

## Setting up API keys

We need an API key to talk to the LLM. If you're using OpenAI, you'll
need to deposit some money first. The minimum is $5 (as of June 2026).
This lesson costs well under 10 cents to run, so that $5 goes a long
way.

I also recommend creating a separate OpenAI project for the course.
Then you can open the usage page and see exactly how much you spent
here, apart from your other work.

The safest way to store the key is in a `.env` file that never gets
committed to git.

Create a `.env` file in your project folder and put your API key in
it (for this Docker-LocalAI setup, we don't need a password):

```bash
# .env file
OPENAI_API_BASE=http://local-ai:8080/v1
OPENAI_API_KEY=not-needed
```

Now add `.env` to `.gitignore` to make sure you never accidentally
commit your key:

```bash
.env
```


Never commit `.env` to git. Treat the API key like a password. If it
leaks, someone else can run up charges on your account.

## Starting Jupyter

Start Jupyter:

```bash
uv run jupyter notebook --ip=0.0.0.0 --no-browser --allow-root
```

Create a new notebook. Throughout the course, you'll copy code from
the section notes into notebook cells.

Check that the OpenAI client works:

```python
from dotenv import load_dotenv
load_dotenv() # This returns True

from openai import OpenAI
openai_client = OpenAI() # This returns blank
```


If you see an error, make sure the key in your `.env` file is
correct.

For Groq or other OpenAI-compatible providers, add the key to
`.env`:

```bash
GROQ_API_KEY=your_key_here
```

And configure the client:

```python
from openai import OpenAI
import os

openai_client = OpenAI(
    api_key=os.getenv("GROQ_API_KEY"),
    base_url="https://api.groq.com/openai/v1"
)
```

## (Optional) Auto-loading .env with dirdotenv

If you don't want to call `load_dotenv()` in every notebook, use
[dirdotenv](https://github.com/alexeygrigorev/dirdotenv).

It loads `.env` files automatically when you `cd` into a directory:

```bash
uv tool install dirdotenv
echo 'eval "$(dirdotenv hook bash)"' >> ~/.bashrc
```

Restart your terminal, and now whenever you enter the project
directory, the variables from `.env` are loaded automatically. No
`load_dotenv()` needed.
