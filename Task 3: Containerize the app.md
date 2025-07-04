#### Step 1: Create a Dockerfile in the project folder.
Open powershell and create a Dockerfile.
```powershell
New-Item Dockerfile -ItemType File
```
#### Step 2: Open the Dockerfile in Visual studio code and add the below script to the file and save it.
```dockerfile
# Use official Python slim image
FROM python:3.10-slim

# Set working directory
WORKDIR /app

# Copy current folder contents into the container at /app
COPY . /app

# Install system dependencies (if any)
RUN apt-get update && apt-get install -y \
    build-essential \
    libpoppler-cpp-dev \
    pkg-config \
    python3-dev \
    && rm -rf /var/lib/apt/lists/*

# Install Python dependencies
RUN pip install --no-cache-dir --upgrade pip
RUN pip install --no-cache-dir fastapi uvicorn python-docx pymupdf nltk scikit-learn sentence-transformers
RUN pip install fastapi uvicorn python-docx pymupdf nltk \
    scikit-learn sentence-transformers python-multipart

# Download NLTK stopwords (optional, can also be done in code)
RUN python -c "import nltk; nltk.download('stopwords')"

# Expose port 8000 for the FastAPI app
EXPOSE 8000

# Command to run the FastAPI app with uvicorn
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```
#### Step 3: Build the docker image
```bash
docker build -t main-app .
```
#### Step 4: Run the container locally
```bash
docker run -p 8000:8000 main-app
```
#### Step 5: Visit http://localhost:8000/docs in your browser.
#### Push the docker image to Docker hub
You can now push your image with:
```bash
docker tag main-app sameeravurity/main-app:latest
docker push sameeravurity/main-app:latest
```


