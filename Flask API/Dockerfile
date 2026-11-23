# Use an official Python runtime as a parent image
FROM python:3.13-slim

# Set the working directory in the container
WORKDIR /usr/src/app

# Install dependencies first so this layer is cached between code changes
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy the rest of the app
COPY . .

# Render injects PORT at runtime; default to 5000 for local docker runs
ENV PORT=5000
EXPOSE 5000

# Feature extraction fetches the page and runs WHOIS, so allow long requests
CMD ["sh", "-c", "exec gunicorn app:app --bind 0.0.0.0:$PORT --workers 2 --timeout 120"]
