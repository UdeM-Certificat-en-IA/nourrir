# NourrIR Flask Project

NourrIR is a minimal Flask-based web application showcasing static pages and an AI-powered chat assistant ("NuRiH Ami") that proxies messages to an Ollama LLM server.

## Table of Contents

- [Features](#features)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Configuration](#configuration)
- [Running the App](#running-the-app)
  - [Locally with Flask](#locally-with-flask)
  - [With Docker](#with-docker)
  - [With docker-compose](#with-docker-compose)
- [Usage](#usage)
- [Project Structure](#project-structure)
- [Troubleshooting](#troubleshooting)
- [Deployment](#deployment)
  - [Render.com Deployment](#rendercom-deployment)
  - [Netlify Deployment (via Docker)](#netlify-deployment-via-docker)
- [License](#license)

## Features

- Static pages:
  - Home (`/`)
  - Integration Policy (`/politique`)
  - HR Contact (`/contact`)
- Floating AI chat widget ("NuRiH Ami") available on all pages
- Proxy endpoint (`/nurih-ami`) to forward user messages to an Ollama LLM server
- Dockerized application with Dockerfile and docker-compose support

## Prerequisites

- Python 3.11 or higher
- pip
- (Optional) Docker & docker-compose
- Access to an Ollama server for the chat backend

## Installation

1. Clone the repository:

   ```bash
   git clone <repository-url>
   cd nourrir_flask
   ```

2. (Optional) Create and activate a virtual environment:

   ```bash
   python3 -m venv venv
   source venv/bin/activate
   ```

3. Install Python dependencies:

   ```bash
   pip install --no-cache-dir -r requirements.txt
   ```

## Configuration

Configure the Ollama API endpoints via environment variables:

- `OLLAMA_CHAT_URL`: URL for the chat completion endpoint. For OpenAI-compatible API, this is typically:
  ```
  https://ollama.artemis-ai.ca/v1/chat/completions
  ```
  Fallback default (legacy Ollama API):
  ```
  http://192.168.2.10:11434/api/chat
  ```
- `OLLAMA_MODELS_URL`: URL for the models listing endpoint. For OpenAI-compatible API:
  ```
  https://ollama.artemis-ai.ca/v1/models
  ```
  Fallback default (legacy):
  ```
  http://192.168.2.10:11434/api/models
  ```
- `OLLAMA_MODEL`: The primary model identifier to use (e.g. `mistral:latest`). Default: `mistral:latest`.

Example:

```bash
export OLLAMA_CHAT_URL=https://ollama.artemis-ai.ca/v1/chat/completions
export OLLAMA_MODELS_URL=https://ollama.artemis-ai.ca/v1/models
export OLLAMA_MODEL=mistral:latest
```

## Running the App

### Locally with Flask

```bash
export FLASK_APP=app.py
export FLASK_ENV=development
export OLLAMA_CHAT_URL=https://ollama.artemis-ai.ca/v1/chat/completions
export OLLAMA_MODELS_URL=https://ollama.artemis-ai.ca/v1/models
export OLLAMA_MODEL=mistral:latest
flask run --host=0.0.0.0 --port=8080
```

Open your browser at [http://localhost:8080](http://localhost:8080).

### With Docker

docker build -t nourrir-flask .
docker run -d -p 8282:8080 \
docker build -t nourrir-flask .
docker run -d -p 8282:8080 \
  -e OLLAMA_CHAT_URL=https://ollama.artemis-ai.ca/v1/chat/completions \
  -e OLLAMA_MODELS_URL=https://ollama.artemis-ai.ca/v1/models \
  -e OLLAMA_MODEL=mistral:latest \
  --name nourrir-flask nourrir-flask
```

Browse to [http://localhost:8282](http://localhost:8282).

### With docker-compose

```bash
docker-compose up --build
```

By default, the web service is exposed on port `8282`.

## Usage

- Navigate to the static pages via the top navigation bar.
- Click the chat icon (💬) in the bottom right to open the NuRiH Ami assistant.
- Type your questions or prompts; the message will be forwarded to the Ollama model and the response displayed in the chat widget.

## Project Structure

```
.
├── app.py
├── Dockerfile
├── docker-compose.yml
├── requirements.txt
├── static/
│   └── assets/...
└── templates/
    ├── base.html
    ├── index.html
    ├── politique.html
    └── contact.html
```

## Troubleshooting

- **Cannot connect to Ollama**: Verify `OLLAMA_CHAT_URL` (or legacy `OLLAMA_URL`) and that the Ollama server is reachable from your network.
- **Port conflicts**: Ensure ports `8080` (Flask) or `8282` (Docker) are available.
- **Asset loading issues**: Check the `/assets/<filename>` route and that files exist under `static/assets/`.

## Deployment

This application is configured for deployment on Render and Netlify.

### Render.com Deployment

1.  **Sign up or Log in** to [Render.com](https://render.com/).
2.  **Create a New Web Service**:
    *   Connect your Git repository where this project is hosted.
    *   Render will automatically detect the `render.yaml` file. Review the settings it populates.
    *   Alternatively, you can manually set up the service:
        *   **Environment**: Docker
        *   **Repository**: Your Git repo URL
        *   **Branch**: Your desired deployment branch (e.g., `main`)
        *   **Dockerfile Path**: `./Dockerfile` (if not automatically detected)
        *   **Instance Type**: Choose an appropriate plan (e.g., Free, Standard). The `render.yaml` defaults to `free`.
3.  **Environment Variables**:
    *   Navigate to your service's "Environment" settings on Render.
    *   Add the following essential environment variables:
        *   `OLLAMA_CHAT_URL`: The URL for your Ollama chat completions API endpoint.
        *   `OLLAMA_MODELS_URL`: The URL for your Ollama models listing API endpoint.
        *   `OLLAMA_MODEL` (Optional): The default Ollama model you wish to use (e.g., `mistral:latest`). If not set, the application's default will be used.
        *   `PORT`: Should be `8080` (as defined in `Dockerfile` and `render.yaml`). Render usually sets this automatically based on Docker EXPOSE or `render.yaml`.
        *   `PYTHON_VERSION`: Should be `3.11` (Render might infer this from the Docker base image).
    *   Ensure these variables are saved. `OLLAMA_CHAT_URL` and `OLLAMA_MODELS_URL` are marked with `sync: false` in `render.yaml`, meaning they *must* be set in the dashboard.
4.  **Deploy**:
    *   Trigger a manual deploy or rely on auto-deploys if configured.
    *   Monitor the deployment logs for any issues.
5.  **Access Your Application**:
    *   Once deployed, Render will provide you with a URL (e.g., `your-app-name.onrender.com`).

### Netlify Deployment (via Docker)

Netlify can deploy services from a Docker container.

1.  **Sign up or Log in** to [Netlify.com](https://netlify.com/).
2.  **Create a New Site**:
    *   Import an existing project.
    *   Connect to your Git provider and select your repository.
3.  **Build Settings**:
    *   Netlify should detect the `netlify.toml` file. This file tells Netlify to use the `Dockerfile` for deployment.
    *   Ensure the correct repository and branch are selected.
    *   Netlify will build the Docker image from your `Dockerfile` and deploy it. The `EXPOSE 8080` instruction in your `Dockerfile` tells Netlify which port your application is listening on.
4.  **Environment Variables**:
    *   Go to **Site settings > Build & deploy > Environment**.
    *   Add the following essential environment variables:
        *   `OLLAMA_CHAT_URL`: The URL for your Ollama chat completions API endpoint.
        *   `OLLAMA_MODELS_URL`: The URL for your Ollama models listing API endpoint.
        *   `OLLAMA_MODEL` (Optional): The default Ollama model you wish to use (e.g., `mistral:latest`).
    *   **Note**: Unlike some platforms, you don't typically need to set `PORT` as an environment variable in Netlify for Docker deployments if `EXPOSE` is used correctly in the Dockerfile. Netlify handles port mapping.
5.  **Deploy Site**:
    *   Trigger a deploy.
    *   Monitor the deploy logs in the Netlify dashboard.
6.  **Access Your Application**:
    *   Once deployed, Netlify will provide you with a URL (e.g., `your-site-name.netlify.app`).

#### Troubleshooting Netlify Docker Deployment

If your Netlify deployment results in a 404 error or the deploy logs indicate that Netlify is attempting a language-specific build (e.g., installing Python dependencies directly) instead of using your Dockerfile:

1.  **Check Netlify UI Build Settings**:
    *   Go to your site in the Netlify dashboard.
    *   Navigate to **Site configuration** (or **Site settings**) > **Build & deploy**.
    *   Under "Build settings," ensure that your site is configured to **Deploy with Docker** (or a similar option indicating Docker image deployment). If it's set to a specific language (like "Python") or "None" with build commands, Netlify might ignore the Dockerfile.
    *   The presence of files like `requirements.txt` (for Python), `package.json` (for Node.js), etc., can cause Netlify's build system to auto-select a language-specific build process if Docker is not explicitly selected as the deployment method in the UI.

2.  **`netlify.toml` Configuration**:
    *   The `netlify.toml` file in this repository is configured to be minimal for Docker deployments, relying on Netlify's Dockerfile detection once Docker is enabled in the UI.
    *   Ensure your `netlify.toml` doesn't contain conflicting build commands that might override Docker deployment settings.

3.  **Review Deploy Logs**:
    *   Carefully examine the deploy logs in the Netlify dashboard.
    *   Look for lines indicating whether Netlify is trying to build a Docker image (e.g., `docker build ...`) or if it's running commands for a specific language runtime (e.g., `pip install ...`, `npm install ...`).
    *   If it's not building with Docker, the UI settings are the most likely cause.

4.  **Dockerfile Instructions**:
    *   Ensure your `Dockerfile` correctly `EXPOSE`s the port your application listens on (e.g., `EXPOSE 8080`). Netlify uses this to route requests to your container.
    *   Ensure your `Dockerfile`'s `CMD` or `ENTRYPOINT` instruction correctly starts your application server (e.g., Gunicorn).

If issues persist, consult the official [Netlify documentation on Docker deployments](https://docs.netlify.com/configure-builds/docker-deploys/) for the most current guidance.

**Important Considerations for Both Platforms**:
*   **Ollama Service**: This application requires access to an Ollama instance. Ensure the `OLLAMA_CHAT_URL` and `OLLAMA_MODELS_URL` environment variables point to a running and accessible Ollama service.
*   **Resource Allocation**: Depending on the traffic and resource needs of the Ollama models, you might need to choose appropriate service plans on Render or Netlify (if their container service has different tiers) to ensure smooth operation.
*   **Logging**: Check the application logs on Render or Netlify to diagnose any issues post-deployment. The application is configured to log to standard output.

## License

This project is provided for educational and demonstration purposes.