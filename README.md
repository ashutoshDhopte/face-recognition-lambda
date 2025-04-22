# Face Recognition as a Service (PaaS via ECR)

## Course: CSE 546 Cloud Computing – Spring 2025
## Author: Ashutosh Dhopte (ASU ID: 1233725170)

⸻

## Project Overview

I developed a serverless face recognition pipeline on AWS using PaaS services. My application streams video frames from IoT clients (e.g., security cameras), detects faces in each frame, and recognizes them against a pretrained dataset—all without managing servers.

### Key components I built:
	•	Face Detection Lambda: Receives base64‑encoded frames via a Function URL, uses MTCNN to locate faces, and publishes cropped face images to an SQS request queue.
	•	Face Recognition Lambda: Triggered by the request queue, loads a ResNet‑based model (InceptionResnetV1 pretrained on VGGFace2), computes embeddings, compares against stored embeddings from resnetV1_video_weights.pt, and sends identification results to an SQS response queue.
	•	Docker Container: Packaged both functions in a single Docker image stored in Amazon ECR, ensuring consistent dependencies (PyTorch, facenet-pytorch, Pillow, OpenCV, boto3).
	•	Messaging Layer: Configured two standard SQS queues (<ASU_ID>-req-queue and <ASU_ID>-resp-queue) for decoupled, scalable message passing.

⸻

### Architecture Diagram

Client → [Face-Detection Lambda] → SQS Request Queue → [Face-Recognition Lambda] → SQS Response Queue → Client



⸻

### Implemented Features
#### Serverless Face Detection
    •	Efficient cold-start handling by caching the MTCNN model in the Lambda execution environment.
    •	Preprocessing: Decoded incoming base64 frames, converted to PIL images, and ran face detection with confidence scores.
#### Real-Time Face Recognition
    •	Loaded a pretrained InceptionResnetV1 model under torch.no_grad() to speed up inference.
    •	Embedded comparison: Computed cosine similarity between detected face embeddings and known embeddings.
    •	Robust message handling via SQS triggers and retries to guarantee at-least-once processing.
#### Containerized Deployment
    •	Consolidated both Lambdas into one Dockerfile, reducing image size by using python:3.x-slim and installing only necessary packages.
    •	Automated push to ECR for seamless CI/CD on AWS.
#### Scalability & Reliability
    •	Stateless Lambdas and SQS ensure horizontal scalability to handle bursts of incoming camera streams.
    •	Designed for idempotent processing and safe handling of in-flight messages to prevent recursive triggers.

⸻

### Technologies & Services
	•	AWS Lambda (Custom container runtime)
	•	Amazon ECR (Docker image registry)
	•	Amazon SQS (Request & response queues)
	•	PyTorch & facenet-pytorch (MTCNN & InceptionResnetV1)
	•	Docker (Containerization)
	•	Pillow, OpenCV, boto3, requests

⸻

### How to Invoke
	1.	Send a POST to the Face-Detection Function URL with JSON body:

    {
      "content": "<base64-encoded-frame>",
      "request_id": "uuid-1234",
      "filename": "frame1.png"
    }


	2.	Poll the Response Queue <ASU_ID>-resp-queue for messages:

    {
      "request_id": "uuid-1234",
      "result": "John Doe"
    }
