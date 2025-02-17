# COT 5930 - Project 2

**Name:** Ibrahim Luke Barhoumeh Z23652726
**Date:** February 17, 2025  
**Professor:** Andrade
**Cloud Run URL** https://imaging-app-856239458079.us-east1.run.app


## Overview

This project builds upon Project 1 by extending our knowledge of **Cloud Run**, **Python**, and **Flask** to incorporate **Gemini AI** (Google’s multimodal LLM API) for generating short descriptions of uploaded images. The app:

1. **Accepts image uploads** from users via a Flask web form.
2. **Calls the Gemini model** with the image to obtain a concise caption/description.
3. **Stores** both the image and a corresponding JSON file (containing the generated caption) in a **Google Cloud Storage (GCS)** bucket.
4. **Displays** the images and generated descriptions via a simple web interface.

## Architecture & Flow

1. **User Upload**  
   - The user visits the **Cloud Run** app and uploads a JPEG.  
   - The Flask endpoint `/upload` saves the file locally before sending it to Gemini AI.

2. **Gemini AI**  
   - Using the Generative Language API (`generateContent`), the app sends a **base64-encoded image** plus a short prompt.  
   - Gemini AI returns a textual description.

3. **Storage**  
   - The image is uploaded to a private GCS bucket under `files/<filename>`.
   - A corresponding JSON file `<filename>.json` is also created, containing:
     ```json
     {
       "title": "<ORIGINAL_FILENAME>",
       "description": "<AI-GENERATED TEXT>"
     }
     ```
   - Both are stored in GCS so the app can list and retrieve them later.

4. **Display**  
   - Users can click on filenames in the index page (`/`) to see a detail page that renders the image and shows the generated description.

## Key Features

- **Uses a Private GCS Bucket**  
  The bucket is **not** publicly accessible; the app controls uploads/downloads via the Cloud Run service account credentials.
- **No Direct Bucket URLs**  
  All image access goes through the Flask `/files/<filename>` endpoint, preventing direct public links to GCS.
- **Secret-Free Code**  
  The **Gemini API key** is passed via an environment variable in Cloud Run (`GENAI_API_KEY`), so it’s **not** hardcoded into `main.py`.
- **Deployed on Cloud Run**  
  The app runs in a Docker container built from a lightweight `python:3.10-slim` base image. A public URL is provided for easy access without authentication.

## Pros & Cons

### Pros
- **Serverless & Scalable**: Uses Cloud Run’s autoscaling; can handle variable traffic without manual server management.  
- **Secure & Organized**: Bucket remains private; the application mediates access.  
- **Modular Architecture**: Separates AI calls (Gemini API) from storage (GCS) and web front end (Flask).  
- **Easy Maintenance**: Code is straightforward Python + Flask; easy to modify or extend.

### Cons
- **API Key Usage**: Even though it’s in an env var, we still rely on a key. More robust would be IAM-based calls (if/when Vertex AI or Gemini allows it).  
- **Cost Considerations**: Storing images and calling the AI for large volumes might get expensive for millions of users.  
- **Single-Region**: Currently, Gemini only runs in `us-central1`, while our bucket/app is in `us-east1`. This introduces some cross-region traffic.

## Where to Improve for Millions of Users
- **Caching**: We could add a caching layer if the same image is uploaded multiple times (less likely).  
- **Parallel Processing**: If we anticipate high concurrency, move AI calls into a task queue or asynchronous architecture.  
- **Load Testing & Observability**: Integrate Cloud Monitoring/Logging in more detail, plus potential error-handling for large images or timeouts.  
- **Multi-Region or CDN**: Place buckets or replicate them across multiple regions for lower latency and resilience.

## Deployment Instructions

1. **Prerequisites**:
   - A GCP project with the **Generative AI API** enabled.
   - A private GCS bucket named `cot5930-image-p1`.
   - Cloud Run enabled on the same project.
2. **Clone the Repository**:
   ```bash
   git clone https://github.com/lukebarhoumeh/COT5930-Project2.git
   cd COT5930-Project2
