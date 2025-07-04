## Task 2: JD Text processing and Scoring using TF-IDF + Cosine similarity
* Preprocess JD
* Compare resume vs JD using:
  * TfidfVectorizer + cosine_similarity (from sklearn)
  * Sentence-transformers for better NLP (free Hugging Face model)
  * Return a match score (e.g., 82%)
#### TF-IDF Vectorization
TfidfVectorizer stands for:
 * TF = Term Frequency → how often a word appears in a document
 * IDF = Inverse Document Frequency → how rare that word is across documents

It converts both the resume and JD into vectors (lists of numbers), where:

  - Common words like "the", "and", "is" are down-weighted

  - Important keywords like “Terraform”, “DevOps”, “Azure” are weighted higher

So instead of comparing plain words, you're comparing the relative importance of words in the documents.

#### Cosine Similarity
Once TF-IDF has turned both texts into numerical vectors, we use cosine_similarity() to calculate how close those two vectors are:

- Cosine similarity = angle between two vectors

- Ranges from 0 (completely different) to 1 (identical)

- Multiply by 100 to get a percentage match score.

Example:

Let’s say:
  
  * Resume: "Experienced in Terraform, Azure, and Kubernetes."
  
  * JD: "Looking for Terraform and Azure DevOps engineer."

TF-IDF creates numerical vectors that capture word importance like:
  
  * Resume_Vector = [0.5, 0.7, 0.9, 0, 0, 0]
  
  * JD_Vector     = [0.6, 0.8, 0, 0.3, 0.1, 0]

Then cosine_similarity() computes how similar those vectors are.

Result: 82.35% match score

#### Step 1: Preprocess JD
This is done using the clean_text() function (lowercase, remove punctuation, stopwords)
#### Step 2:
Compare resume vs JD using: TfidfVectorizer + cosine_similarity (from sklearn)
Implemented in /match_tfidf endpoint
```python
vectorizer = TfidfVectorizer()
tfidf_matrix = vectorizer.fit_transform([resume_text, jd_text])
cosine_similarity(tfidf_matrix[0:1], tfidf_matrix[1:2])
```
#### Step 3: Use sentence-transformers for better NLP
Implemented in /match_semantic endpoint
```python
model = SentenceTransformer('all-MiniLM-L6-v2')
embeddings = model.encode([resume_text, jd_text], convert_to_tensor=True)
score = util.pytorch_cos_sim(embeddings[0], embeddings[1])
```
#### Step 4: Return a match score (e.g., 82%)
Each endpoint returns a formatted score like "82.35%"
#### Step 5: Edit the API
Install required libraries
```bash
pip install scikit-learn
pip install scikit-learn sentence-transformers
```
Edit the main.py file by adding the below script to it.
```python
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.metrics.pairwise import cosine_similarity
from sentence_transformers import SentenceTransformer, util

# Load sentence transformer model once
sentence_model = SentenceTransformer('all-MiniLM-L6-v2')

@app.post("/match_tfidf")
async def match_resume_tfidf(job_description: str = Form(...)):
    global uploaded_resume_text

    if not uploaded_resume_text.strip():
        raise HTTPException(status_code=400, detail="No resume uploaded yet.")

    cleaned_jd = clean_text(job_description)
    corpus = [uploaded_resume_text, cleaned_jd]

    vectorizer = TfidfVectorizer()
    tfidf_matrix = vectorizer.fit_transform(corpus)

    score = cosine_similarity(tfidf_matrix[0:1], tfidf_matrix[1:2])[0][0] * 100

    return {
        "similarity_score": f"{score:.2f}%",
        "method": "TF-IDF + Cosine Similarity"
    }

@app.post("/match_semantic")
async def match_resume_semantic(job_description: str = Form(...)):
    global uploaded_resume_text

    if not uploaded_resume_text.strip():
        raise HTTPException(status_code=400, detail="No resume uploaded yet.")

    embeddings = sentence_model.encode(
        [uploaded_resume_text, job_description], convert_to_tensor=True
    )
    score = util.pytorch_cos_sim(embeddings[0], embeddings[1]).item() * 100

    return {
        "semantic_similarity_score": f"{score:.2f}%",
        "method": "Sentence Transformers (MiniLM-L6-v2)"
    }
```
Reload the API again using the command
```bash
uvicorn resume_matcher:app --reload
```
Open your browser at http://127.0.0.1:8000/docs and check the API for resume and JD match.

