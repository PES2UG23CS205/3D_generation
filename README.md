# 3D Game Asset Generation API

This project provides a backend service for rapidly prototyping 3D game assets from text prompts. It uses OpenAI's **Shap-E** model, exposed through a high-performance, asynchronous **FastAPI** application. The service is designed to take a simple text prompt and return a game-ready 3D model, bridging the gap between creative ideas and tangible assets.

The entire application is self-contained within a single Google Colab notebook for easy execution and demonstration.

## Table of Contents
- [Key Features](#key-features)
- [Sample Output](#sample-output)
- [Demonstration Setup](#demonstration-setup)
  - [Prerequisites](#prerequisites)
  - [Execution Steps](#execution-steps)
- [API Reference](#api-reference)
  - [Endpoints](#endpoints)
  - [Example Workflow](#example-workflow)
- [Project Structure](#project-structure)

## Key Features
-   **Text-to-3D Generation**: Leverages the Shap-E model to create 3D meshes from natural language descriptions.
-   **Asynchronous Architecture**: The API is built asynchronously to handle long-running model inference (1-3 minutes) without blocking or causing timeouts—a critical design pattern for production-level ML services.
-   **Standardized Outputs**: The service generates both a `.glb` file, the modern standard for 3D web and game assets, and a rotating `.gif` for immediate visual previewing.
-   **Direct Colab Execution**: The project is packaged as a `.ipynb` file, allowing for one-click setup in Google Colab with free GPU access.

---

## Sample Output
The following asset was generated using the API.

#### Prompt: `a cute magical fox`
| Animated GIF Preview                                           | 3D Model (`.glb`)                                        |
| :-------------------------------------------------------------: | :----------------------------------------------------------: |
| ![Fox GIF Preview](./generated_assets/sample_cute_magical_fox.gif) | [Download .glb File](./generated_assets/sample_cute_magical_fox.glb) |

---

## Demonstration Setup
The project is designed to be executed directly in Google Colab.

### Prerequisites
- A Google Account (for Colab).
- A free [Ngrok](https://ngrok.com/) account to create a public URL tunnel to the API.

### Execution Steps
1.  **Open Notebook in Colab**: Navigate to the `3D_Model.ipynb` file in this repository. GitHub provides an "Open in Colab" button at the top of the file viewer. Click it.
2.  **Set Runtime Type**: Once the notebook is open in Colab, go to the menu and navigate to `Runtime` -> `Change runtime type` and select `T4 GPU` from the dropdown. This is required for model inference.
3.  **Add Ngrok Token**:
    -   Log in to your [ngrok dashboard](https://dashboard.ngrok.com/get-started/your-authtoken) and copy your authentication token.
    -   In the Colab code cell, find the line `NGROK_AUTH_TOKEN = "YOUR_NGROK_AUTHTOKEN_HERE"` and paste your token inside the quotes.
4.  **Run the Notebook**: Run the single large code cell. It will automatically install all dependencies, load the Shap-E model, and start the FastAPI server. Upon successful launch, it will print the public API URL provided by ngrok.

---

## API Reference
The server exposes a RESTful API to generate and retrieve 3D models.

### Endpoints

`GET /status`
-   **Description**: Checks the health and status of the API and the loaded model.
-   **Success Response (200 OK)**:
    ```json
    {
      "status": "online",
      "model_version": "openai/shap-e",
      "uptime_seconds": 358.98,
      "device": "cuda"
    }
    ```

`POST /generate3d`
-   **Description**: Submits a new job to generate a 3D model. This is a non-blocking, asynchronous call.
-   **Query Parameter**: `prompt` (string, required).
-   **Success Response (202 Accepted)**:
    ```json
    {
      "message": "3D model generation job has been queued.",
      "job_id": "012d768d-8745-4a2e-aeb4-44457e8bffe3",
      "status_url": "/job/012d768d-8745-4a2e-aeb4-44457e8bffe3"
    }
    ```

`GET /job/{job_id}`
-   **Description**: Polls for the status of a specific generation job.
-   **Path Parameter**: `job_id` (string, required).
-   **Success Response (200 OK) - Completed**:
    ```json
    {
      "id": "012d768d-8745-4a2e-aeb4-44457e8bffe3",
      "prompt": "a cute magical fox",
      "status": "completed",
      "result": {
        "glb_url": "/assets/012d768d-8745-4a2e-aeb4-44457e8bffe3.glb",
        "gif_url": "/assets/012d768d-8745-4a2e-aeb4-44457e8bffe3.gif",
        ...
      }
    }
    ```

### Example Workflow
You can use a tool like `curl` to interact with the live API.
```bash
# 1. Start a generation job for "a sci-fi space helmet"
curl -X POST "https://<your-ngrok-url>/generate3d?prompt=a%20sci-fi%20space%20helmet"

# Note the job_id from the response.

# 2. Check the job status after ~90 seconds
curl -X GET "https://<your-ngrok-url>/job/<your-job-id>"

# 3. Once complete, view the GIF preview in a browser
# e.g., https://<your-ngrok-url>/assets/<your-job-id>.gif
```

---

## Project Structure
```
.
├── 3D_Model.ipynb    # The main Colab notebook with all code and logic.
├── requirements.txt          # Python dependencies (for local setup reference).
├── generated_assets/         # Directory for sample outputs.
│   └── sample_cute_magical_fox.gif
│   └── sample_cute_magical_fox.glb
├── .gitignore                # Specifies files and directories to be ignored by Git.
└── README.md                 # This project documentation file.
```
