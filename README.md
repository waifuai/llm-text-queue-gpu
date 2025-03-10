# llm-text-queue-gpu

This project provides a simple and efficient text generation service using a pre-trained language model (distilgpt2) and a Redis queue for handling requests. It consists of a Flask-based API that exposes endpoints for generating text and checking service health. The service is designed to be run on a GPU for faster inference.

## Table of Contents

- [Features](#features)
- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Configuration](#configuration)
- [Running the Service](#running-the-service)
- [API Documentation](#api-documentation)
- [Testing](#testing)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)
- [License](#license)

## Features

*   Text generation using the distilgpt2 language model.
*   Request queuing using Redis for handling concurrent requests.
*   Flask API for easy interaction.
*   GPU support for faster inference.
*   Health check endpoint for monitoring service status.

## Architecture

The project consists of three main components:

*   **`respond.py`**: This is the core text generation service. It loads the distilgpt2 model and handles requests to generate text. It exposes a `/generate` endpoint for POST requests containing the prompt.
*   **`queue.py`**: This service acts as a gateway for incoming requests. It receives prompts, queues them using Redis, and then forwards them to `respond.py` for processing. It also exposes a `/generate` endpoint.
*   **`worker.py`**: This is a Redis worker that listens for jobs on the queue and executes them. This allows `respond.py` to handle multiple requests concurrently without blocking.

The flow of a request is as follows:

1.  A client sends a POST request to `/generate` on `queue.py`.
2.  `queue.py` adds the request to a Redis queue.
3.  `worker.py` picks up the request from the queue.
4.  `worker.py` calls the `call_predict_response` function in `queue.py`, which sends a POST request to `/generate` on `respond.py`.
5.  `respond.py` generates the text and returns it to `queue.py` via the worker.
6.  `queue.py` returns the generated text to the client.

## Prerequisites

*   Python 3.7+
*   pip
*   Redis server (installed and running)
*   A CUDA-enabled GPU (optional, but recommended for performance)

You can find instructions for installing Redis on your system on the official Redis website: [https://redis.io/docs/getting-started/](https://redis.io/docs/getting-started/)

## Installation

1.  Clone the repository:

    ```bash
    git clone <YOUR_REPOSITORY_URL>
    cd llm-text-queue-gpu
    ```

2.  Create and activate a virtual environment:

    ```bash
    python3 -m venv venv
    source venv/bin/activate  # On Linux/macOS
    venv\Scripts\activate  # On Windows
    ```

3.  Install dependencies:

    ```bash
    pip install --user -r requirements.txt
    ```

    Alternatively, you can run the `setup.sh` script, which performs steps 2 and 3:

    ```bash
    ./src/setup.sh
    ```

## Configuration

1.  Copy the `.env.example` file to `.env`:

    ```bash
    cp .env.example .env
    ```

2.  Modify the `.env` file to set the appropriate values for your environment. The following environment variables are available:

    *   `MODEL_PATH`: The path to the pre-trained language model (default: `distilgpt2`).
    *   `GPU_SERVICE_URL`: The URL of the GPU service (default: `http://localhost:5001`).
    *   `REDIS_URL`: The URL of the Redis server (default: `redis://localhost:6379`).
    *   `QUEUE_PORT`: The port for the queue service (default: `5000`).
    *   `RESPOND_PORT`: The port for the response service (default: `5001`).
    *   `MAX_NEW_TOKENS`: The maximum number of new tokens to generate (default: 20).

    **Note:** The `.env` file should not be committed to the repository for security reasons.

## Running the Service

Start the services using the `scripts/start-services.sh` script:

```bash
./scripts/start-services.sh
```

This script will start the Redis server (if not already running), the worker, the queue service, and the response service. The services will run in the background.

## API Documentation

### Health Check

```http
GET /health
```

This endpoint checks the health of the services and returns a 200 OK if all services are running.

### Generate Text

```http
POST /generate
Content-Type: application/json

{
  "prompt": "Your input text here"
}
```

This endpoint generates text based on the provided prompt. The prompt should be a string.

**Example Request:**

```http
POST /generate
Content-Type: application/json

{
  "prompt": "Hello, how are you?"
}
```

**Example Response:**

```json
{
  "response": "I am doing well, thank you for asking."
}
```

## Testing

The project includes API tests in `tests/test_api.py`, queue tests in `tests/test_queue.py`, and respond tests in `tests/test_respond.py`. You can run all tests using `pytest`:

```bash
pytest tests
```

## Troubleshooting

*   **Error: "Missing 'prompt' parameter"**: Make sure you are sending a JSON payload with a `prompt` field in your POST request to `/generate`.
*   **Error: "Error calling GPU service"**: Ensure that the `respond.py` service is running and accessible at the configured `GPU_SERVICE_URL`.
*   **Error: "Service unavailable"**: Check if Redis server, worker, queue and respond services are running.
*   **If encountering CUDA errors:** Ensure your CUDA drivers and toolkit are correctly installed and compatible with your PyTorch version.

## Contributing

1.  Fork the repository.
2.  Create a new branch for your feature or bug fix.
3.  Commit your changes.
4.  Push your branch to your forked repository.
5.  Create a pull request.

Please adhere to the project's code of conduct.

## License

This project is licensed under the MIT-0 License - see the [LICENSE](LICENSE) file for details.
