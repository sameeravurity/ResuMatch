## Task 1: Install FastAPI and Uvicorn
* Installing FastAPI & Uvicorn
* Creating a minimal API with two endpoints:

  * POST /upload_resume
  
  * POST /match
### Step 1: Open your terminal and create a virtual environment
* Install latest version of Python in the local system based on the OS.
  Click the link to download Python https://www.python.org/downloads
* Create a directory for your project. In my case I created AIProject folder.
* Open Powershell or Command Prompt as administrator and cd to the AIProject diretory.
```bash
python --version
```
You should see your Python version
* Create a virtual environment
```bash
python -m venv venv
```
This creates a new venv folder.
* Activate the environment
```bash
.\venv\Scripts\activate   #for Windows
```
### Step 2: Create main.py file with the basic API and save it in the AIProject folder
Open visual studio code, edit the required code and save it in a local folder where the virtual environment is activated.
```python
from fastapi import FastAPI, UploadFile, File, HTTPException, Form
from docx import Document
import fitz  # PyMuPDF
import io
import re
import nltk
from nltk.corpus import stopwords

app = FastAPI()

# Download NLTK stopwords only once (avoid repeated downloads)
try:
    stop_words = set(stopwords.words('english'))
except LookupError:
    nltk.download('stopwords')
    stop_words = set(stopwords.words('english'))

uploaded_resume_text = ""

# Utility: Clean and normalize text
def clean_text(text: str) -> str:
    text = text.lower()
    text = re.sub(r'[^a-z\s]', '', text)  # remove symbols and numbers
    words = text.split()
    filtered = [word for word in words if word not in stop_words]
    return ' '.join(filtered)

@app.post("/upload_resume")
async def upload_resume(file: UploadFile = File(...)):
    global uploaded_resume_text

    contents = await file.read()
    file_extension = file.filename.lower().split('.')[-1]

    try:
        if file_extension == "pdf":
            with fitz.open(stream=contents, filetype="pdf") as doc:
                text = "".join([page.get_text() for page in doc])
        elif file_extension == "docx":
            doc = Document(io.BytesIO(contents))
            text = "\n".join([para.text for para in doc.paragraphs])
        elif file_extension == "txt":
            text = contents.decode("utf-8", errors="ignore")
        else:
            raise HTTPException(status_code=400, detail="Only PDF, DOCX, and TXT files are supported.")

        uploaded_resume_text = clean_text(text)

    except Exception as e:
        raise HTTPException(status_code=500, detail=f"Failed to parse and clean file: {e}")

    return {"filename": file.filename, "message": "Resume uploaded, cleaned, and parsed successfully."}

@app.post("/match")
async def match_resume(job_description: str = Form(...)):
    global uploaded_resume_text

    if not uploaded_resume_text.strip():
        raise HTTPException(status_code=400, detail="No resume uploaded yet.")

    cleaned_jd = clean_text(job_description)
    resume_words = set(uploaded_resume_text.split())
    jd_words = set(cleaned_jd.split())

    matched_keywords = resume_words & jd_words
    missing_keywords = jd_words - resume_words

    match_score = (len(matched_keywords) / len(jd_words)) * 100 if jd_words else 0

    return {
        "match_score": f"{match_score:.2f}%",
        "matched_keywords": sorted(list(matched_keywords)),
        "missing_keywords": sorted(list(missing_keywords))
    }

```
* Before running the API locally install PyMuPDF for extracting text from PDFs, nltk for text processing and cleaning and python-docx for extracting text from documents.
* ```bash
  pip install pymupdf nltk python-docx
  ```
  * Run the API locally
  ```bash
  uvicorn main:app --reload
  ```
Once you see Uvicorn running on http://127.0.0.1:8000, your API is live and ready to accept requests.
### Step 3: Open your browser at http://127.0.0.1:8000/docs

* This opens interactive Swagger UI where you can test:

  * POST /upload_resume: upload a PDF, DOCX, or TXT file (for now, it just reads text)

  * POST /match: enter some job description text, and it returns a simple match score
* Use POST /upload_resume to upload a sample resume file.

* Use POST /match and paste the Job Description text in the input form.

You should get a match score, matched keywords and missing keywords.
