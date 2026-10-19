# AI Resume Matcher 🤖

An AI-powered application that analyzes a candidate's resume against a job description and calculates a matching percentage based on skills, work experience, and educational qualifications.

The project uses a Large Language Model (LLM) through the Groq API to extract structured information from resumes and job descriptions, then applies a Python-based matching algorithm to generate a detailed evaluation report.

## 🚀 Project Overview

Recruiters often spend significant time reviewing resumes to identify candidates who meet specific job requirements. This project aims to simplify the process by automating resume analysis and job requirement matching.

The application extracts relevant information from a PDF resume, analyzes a job description, compares the candidate's profile against the required qualifications, and generates an overall matching score.

## ✨ Features

* **PDF Resume Parsing:** Extracts text from PDF resumes using PyMuPDF.
* **AI-Powered Resume Analysis:** Uses an LLM to extract the candidate's name, skills, experience, and education.
* **Job Description Analysis:** Extracts required skills, experience, and educational qualifications from a job description.
* **Structured Data Validation:** Uses Pydantic models to validate extracted information.
* **Skill Matching:** Identifies matched and missing skills using case-insensitive text matching.
* **Experience Matching:** Estimates relevant work experience and compares it with job requirements.
* **Education Matching:** Compares the candidate's educational qualifications with the required qualifications.
* **Weighted Scoring:** Calculates an overall matching percentage using configurable weights.
* **Detailed Matching Report:** Displays individual scores, matched skills, missing skills, and the overall result.

## 🛠️ Tech Stack

| Technology         | Purpose                                       |
| ------------------ | --------------------------------------------- |
| Python             | Core application logic and matching algorithm |
| Groq API           | Access to the language model                  |
| OpenAI GPT-OSS 20B | Model used for information extraction         |
| PyMuPDF            | PDF text extraction                           |
| Pydantic           | Data modeling and validation                  |
| python-dotenv      | Environment variable management               |
| JSON               | Structured data exchange                      |

## ⚙️ How It Works

The application follows these steps:

**1. Resume Parsing**

PyMuPDF extracts text from every page of the candidate's PDF resume and combines it into a single text string.

**2. Resume Information Extraction**

The extracted text is sent to the language model through the Groq API. The model identifies the candidate's name, skills, experience, and education and returns the information in JSON format.

**3. Job Requirement Extraction**

The job description is sent to the model in a separate API request. The model extracts the required skills, years of experience, and educational qualifications.

**4. Data Validation**

Pydantic models validate the structure of the extracted information before it is passed to the matching engine.

**5. Skill Matching**

The application normalizes skill names and compares the candidate's skills with the required skills. It identifies matching skills and skills that were not found.

**6. Experience Matching**

The language model estimates the candidate's relevant experience from the available resume information. Python then calculates an experience score by comparing the estimate with the job's experience requirement.

**7. Education Matching**

The application performs basic text-based matching between the candidate's educational qualifications and the job requirements.

**8. Overall Score Calculation**

The individual scores are combined using a weighted average to generate an overall matching percentage.

**9. Report Generation**

The application prints a report containing the overall match, individual scores, matched skills, missing skills, and available experience and education results.

## 📊 Matching Algorithm

The application evaluates candidates across three categories.

| Category   |   Weight |
| ---------- | -------: |
| Skills     |      60% |
| Experience |      25% |
| Education  |      15% |
| **Total**  | **100%** |

### Skill Matching

The skill score is calculated as:

`Skill Score = (Matched Skills / Required Skills) × 100`

For example, if a job requires five skills and three are found in the resume, the skill matching score is 60%.

### Experience Matching

The experience score is calculated as:

`Experience Score = min(Candidate Experience / Required Experience, 1) × 100`

The score is capped at 100%. If the available information is insufficient to calculate a score, the category is excluded from the overall calculation.

### Education Matching

Education is evaluated using basic text matching. A matching qualification receives a score of 100%, while an unmatched qualification receives 0%. If no education requirement is provided, the score is treated as unavailable.

### Overall Matching Score

The overall score is a weighted average of the available category scores. If a category cannot be evaluated, its weight is excluded and the remaining weights are adjusted proportionally.

**Note:** These weights are configurable design choices. The matching percentage represents the result of the implemented scoring rules, not the probability of being hired.

## 📁 Project Structure

```text
resume_matcher/
│
├── main.py                 # Main application and matching engine
├── DataBricks Resume.pdf   # Sample PDF resume for testing
├── .env                    # Groq API key (excluded from GitHub)
├── .gitignore              # Files excluded from version control
├── pyproject.toml          # Project configuration and dependencies
├── uv.lock                 # Locked dependency versions
└── README.md               # Project documentation
```

## 💻 Installation and Setup

### Prerequisites

* Python 3.12 or a compatible Python version
* [uv](https://docs.astral.sh/uv/) package manager
* A Groq API key

### 1. Clone the Repository

```bash
git clone https://github.com/YOUR_USERNAME/resume-matcher.git
cd resume-matcher
```

Replace `YOUR_USERNAME` and `resume-matcher` with your GitHub username and repository name.

### 2. Install Dependencies

```bash
uv sync
```

### 3. Configure the API Key

Create a `.env` file in the project root and add:

```env
GROQ_API_KEY=your_groq_api_key
```

Replace the placeholder with your actual Groq API key.

**Security:** Never upload your `.env` file or expose your API key in a public repository.

### 4. Add a Resume

Place a PDF resume in the project directory.

The current implementation expects the filename:

```text
DataBricks Resume.pdf
```

If you use a different filename, update the corresponding path in `main.py`.

### 5. Run the Application

```bash
uv run python main.py
```

The application will extract resume information, analyze the job requirements, calculate matching scores, and display the final report in the terminal.

The current version uses a job description defined directly in `main.py`.

## ⚠️ Current Limitations

* The application currently supports PDF resumes with extractable text.
* The resume filename and job description are hardcoded.
* Skill matching relies on exact normalized text rather than semantic similarity.
* Education matching uses basic text comparison.
* Relevant work experience is estimated by an LLM and may be inaccurate.
* Extracted information is validated structurally but is not independently verified against every resume claim.
* The application currently provides a terminal-based interface.

## 🔮 Future Improvements

* [ ] Build a Streamlit web interface.
* [ ] Allow users to upload resumes and paste job descriptions.
* [ ] Add support for Word documents (DOCX).
* [ ] Improve skill matching using aliases and semantic similarity.
* [ ] Improve education matching and qualification normalization.
* [ ] Calculate experience using structured employment dates.
* [ ] Provide evidence for extracted skills and qualifications.
* [ ] Generate personalized resume improvement suggestions.
* [ ] Add unit tests and improved error handling.
* [ ] Support evaluation of multiple candidates.

## 🎯 Learning Outcomes

Through this project, I explored:

* Integrating LLMs into Python applications using APIs.
* Prompt engineering for structured information extraction.
* PDF processing with PyMuPDF.
* Working with JSON and Python data structures.
* Data validation using Pydantic.
* Managing API credentials with environment variables.
* Implementing rule-based matching algorithms.
* Calculating weighted scores and handling unavailable data.
* Combining LLM-based text interpretation with deterministic Python logic.

## 🛡️ Responsible Use

AI-generated resume analysis may contain errors or omissions. Matching results should be reviewed by a human and should not be used as the sole basis for employment decisions.

Only process resumes you are authorized to use, and avoid sharing confidential candidate information with external services without appropriate authorization.

## 📄 License

No license has been specified yet. Add an appropriate open-source license if you intend to permit others to use, modify, and distribute this project.

---

**Developed using Python, Groq API, Pydantic, and PyMuPDF.**
