# Use the official Python base image with a slim version of Debian
# The slim image is smaller and already includes essential dependencies.
FROM python:3.11-slim

# Set the working directory inside the container
WORKDIR /app

# Install system dependencies, including FFmpeg
# apt-get update fetches the latest package lists
# apt-get install -y ffmpeg installs FFmpeg without prompting
RUN apt-get update && apt-get install -y ffmpeg

# Copy the requirements file and install Python dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy your Python script into the container
COPY main.py .

# Expose any necessary ports, if applicable.
# ZLMediaKit is on port 8554 and your WebSocket is on 7860, but this container
# only needs to connect out, so no port needs to be exposed for this script.

# Set the command to run your Python script when the container starts
CMD ["python3", "main.py"]
