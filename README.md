# ResuMatch
ResuMatch is an AI-powered resume screening and job description matching API.
Built using FastAPI, NLP, and cloud-native DevOps tools, it analyzes resumes and job descriptions to generate a smart match score that helps recruiters or job seekers understand fit instantly.

This lightweight, scalable tool is ideal for startups, placement cells, or developers looking to integrate intelligent resume screening into their workflow.

## Features:
* Upload and parse resumes (PDF, DOCX, TXT)

* Accept job descriptions as input

* Use NLP to extract and analyze key data

* Calculate similarity using TF-IDF + cosine similarity

* Return a smart match score (e.g., 78%)

* Containerized with Docker, CI/CD via GitHub Actions

* Deployed to the cloud using free-tier infrastructure

## Summary: Free Tool Stack
| Task         |    Tool|
|--------------| --------|
|Resume Parsing| PyMuPDF |
|NLP|scikit-learn, spaCy, HuggingFace|
|API|FastAPI|
|Docker|Docker + DockerHub|
|CI/CD|GitHub Actions|
|Hosting|Render / Railway (Free-tier PaaS)|
|DB|Render Postgres (256MB free) or SQLite|

## Project Goal (MVP):
#### Build an API that: 
* Accepts a resume (PDF)
* Accepts a job description (text)
* Extracts text, compares the two
* Returns a similarity score (e.g., 78% match)
* Is containerized, automated via CI/CD, and deployed to the cloud for free


