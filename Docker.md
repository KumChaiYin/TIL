# Docker

Background: Deploy my 3D project on HuggingFace

## Dockerfile
A text-based configuration file that defines the step-by-step instructions to assemble a reproducible container image, bundling the application code with its system-level dependencies.

``` Dockerfile
# 1. SPECIFY LANGUAGE (The Base Image)
# "slim" is a smaller version of linux without unnecessary tools.
FROM python:3.10-slim

# 2. SYSTEM DEPENDENCIES (The "sudo apt-get" part)
# We add --no-install-recommends to keep the image small.
RUN apt-get update && apt-get install -y \
    libxkbcommon-x11-0 \
    libgl1 \
    && rm -rf /var/lib/apt/lists/*

# 3. SET WORKING DIRECTORY
WORKDIR /app

# 4. INSTALL PYTHON LIBRARIES
# Copy just the requirements first to use Docker caching (speeds up re-builds).
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# 5. COPY YOUR CODE
COPY . .

# 6. SWITCH USER (Safety)
# By default, Docker runs as 'root'. This is dangerous if hacked.
# We create a user named 'appuser' and switch to it.
RUN useradd -m -u 1000 appuser
USER appuser

# 7. COMMAND TO RUN
CMD ["python", "main.py"]
```

## The DevOps Side
Hugging Face acts as the "DevOps" provider. They read this YAML, see `sdk: docker`, take your Dockerfile, build it, find a server, and run it. Else, we need to manage our own server.
```YAML
---
title: My Blender App
emoji: 🎨
colorFrom: blue
colorTo: red
sdk: docker        # <--- This tells HF to look for your Dockerfile
app_port: 7860     # <--- Crucial! The port your app must listen on
pinned: false
---
```