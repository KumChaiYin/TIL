# Docker

Background: Deploy my 3D project on HuggingFace

## 1. Core Concepts: Images & Layers

**The Layer System**
- Definition: Every instruction in a Dockerfile that modifies the filesystem (e.g., RUN, COPY) creates a new Layer.
- Immutability (Key Concept): Layers are Read-Only. Once created, they cannot be changed.

Note: You cannot "undo" a previous layer. If you install a large file in Layer 1 and delete it in Layer 2, the file is hidden from the final view, but it still exists in Layer 1 and occupies disk space. (That's why we chain commands with && in one line).

**The Image vs. The Container**
- Image: A read-only template. It is essentially a collection of frozen layers.
- Container: A running instance of an image.
- Formula: Container = Image (Read-Only Layers) + Top Writable Layer.

Any changes you make while the app is running (writing logs, creating temp files) happen only in this top thin writable layer. The underlying image remains untouched.

**Docker Caching (Why order matters)**
- How it works: When building, Docker checks if a layer matches a previous build.
- Unchanged: Docker uses the cached version (Instant).
- Changed: Docker rebuilds that layer AND all layers after it.
- Optimization: This is why we copy requirements.txt before the source code. 

If you modify your code (main.py), only the layers after COPY . . are rebuilt. The heavy pip install layer is cached and skipped, saving minutes of build time.

## 2. Dockerfile
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

## 3. The DevOps Side
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