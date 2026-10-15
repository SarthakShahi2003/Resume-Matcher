import os
import json
import pymupdf
from pydantic import BaseModel
from dotenv import load_dotenv
from groq import Groq


# Defines the information we want to extract from the resume.
class ResumeDetails(BaseModel):
    name: str
    skills: list[str]
    experience: list[str]
    education: list[str]


# Defines the requirements we want to extract from HR's description.
class JobRequirements(BaseModel):
    required_skills: list[str]
    required_experience_years: float | None = None
    required_education: list[str]


# Opens the PDF resume.
doc = pymupdf.open("DataBricks Resume.pdf")

# Extracts text from every page and combines it into one string.
resume_text = "\n".join(
    page.get_text() for page in doc
)

# Closes the PDF after extracting its text.
doc.close()


# Loads the API key from the .env file.
load_dotenv()

# Reads the Groq API key from environment variables.
my_api_key = os.getenv("GROQ_API_KEY")

# Stops the program if the API key is missing.
if not my_api_key:
    raise ValueError("Groq API key is missing!")

# Creates the Groq client to communicate with the AI model.
client = Groq(api_key=my_api_key)
model = "openai/gpt-oss-20b"


# RESUME EXTRACTION

# Converts the Pydantic model into a schema for the AI to follow.
schema = ResumeDetails.model_json_schema()

# Tells the AI what information to extract from the resume.
system_prompt = f"""
Extract the candidate's name, skills, experience, and education
from the resume.

Return the extracted information as a valid JSON object.

Rules:
- Use only information present in the resume.
- Do not invent missing details.
- Return an empty list if skills, experience, or education are missing.
- Return an empty string if the name is missing.

Follow this JSON schema:
{schema}
"""

# Sends the instructions and resume text to the AI.
messages = [
    {
        "role": "system",
        "content": system_prompt
    },
    {
        "role": "user",
        "content": resume_text
    }
]

# Requests the AI to return its answer in JSON format.
response = client.chat.completions.create(
    model=model,
    messages=messages,
    response_format={"type": "json_object"}
)

# Extracts the JSON text from the AI's response.
answer = response.choices[0].message.content

# Converts the JSON text into a Python dictionary.
data = json.loads(answer)

# Validates the extracted data and creates a ResumeDetails object.
resume_details = ResumeDetails.model_validate(data)

# Displays the extracted resume information.
print("\n--- Extracted Resume Details ---")
print(resume_details.model_dump_json(indent=2))


# HR JOB REQUIREMENTS EXTRACTION

# Sample job description provided by HR.
job_description = """
Job Title: AI Engineer

Required Skills:
Python, SQL, Machine Learning, LangChain, RAG

Required Experience: 2 years

Education: Bachelor's degree in Computer Science
"""

# Converts the JobRequirements model into a schema for the AI to follow.
job_schema = JobRequirements.model_json_schema()

# Tells the AI to extract the requirements mentioned in the job description.
job_system_prompt = f"""
Extract the required skills, experience, and education
from the job description.

Return the information as a valid JSON object.

Rules:
- Extract only requirements mentioned in the description.
- Do not invent missing requirements.
- If experience is not specified, return null.
- If skills or education are not specified, return an empty list.

Follow this JSON schema:
{job_schema}
"""

# Sends the instructions and job description to the AI.
job_messages = [
    {
        "role": "system",
        "content": job_system_prompt
    },
    {
        "role": "user",
        "content": job_description
    }
]

# Requests the extracted job requirements in JSON format.
job_response = client.chat.completions.create(
    model=model,
    messages=job_messages,
    response_format={"type": "json_object"}
)

# Extracts the JSON text returned by the AI.
job_answer = job_response.choices[0].message.content

# Converts the JSON text into a Python dictionary.
job_data = json.loads(job_answer)

# Validates the extracted data and creates a JobRequirements object.
job_requirements = JobRequirements.model_validate(job_data)

# Displays the extracted job requirements.
print("\n--- Extracted Job Requirements ---")
print(job_requirements.model_dump_json(indent=2))


# RESUME MATCHING ENGINE

# Converts skill names to lowercase and removes extra spaces for comparison.
candidate_skills = {
    skill.strip().casefold()
    for skill in resume_details.skills
}

required_skills = {
    skill.strip().casefold()
    for skill in job_requirements.required_skills
}

# Finds which required skills are present in the resume.
matched_skills = candidate_skills.intersection(required_skills)
missing_skills = required_skills - candidate_skills

# Calculates the percentage of required skills found in the resume.
if required_skills:
    skill_score = len(matched_skills) / len(required_skills) * 100
else:
    skill_score = None


# Asks the AI to estimate relevant experience using the resume evidence.
experience_prompt = f"""
Estimate the candidate's relevant work experience in years for this job.

Job requirements:
{job_description}

Resume experience:
{resume_details.experience}

Rules:
- Use only the provided resume evidence.
- Do not count unrelated experience as relevant.
- If the duration cannot be determined, return null.
- Return only a JSON object with the key "experience_years".
- The value must be a number or null.
"""

experience_response = client.chat.completions.create(
    model=model,
    messages=[
        {
            "role": "system",
            "content": experience_prompt
        },
        {
            "role": "user",
            "content": "Estimate the candidate's relevant experience."
        }
    ],
    response_format={"type": "json_object"}
)

# Reads the estimated experience; missing or unclear duration remains unknown.
experience_data = json.loads(
    experience_response.choices[0].message.content
)

candidate_experience = experience_data.get("experience_years")
required_experience = job_requirements.required_experience_years

# Calculates the experience score only when both values are available.
if (
    candidate_experience is not None
    and required_experience is not None
    and required_experience > 0
):
    experience_score = min(
        candidate_experience / required_experience, 1
    ) * 100
elif required_experience == 0:
    experience_score = 100
else:
    experience_score = None


# Compares education using simple text matching.
candidate_education = " ".join(
    item.casefold() for item in resume_details.education
)

if not job_requirements.required_education:
    education_score = None
else:
    # Checks whether any required qualification has a matching phrase.
    education_matches = any(
        requirement.casefold() in candidate_education
        or candidate_education in requirement.casefold()
        for requirement in job_requirements.required_education
        if candidate_education
    )

    education_score = 100 if education_matches else 0


# Combines available scores using their weights.
weights = {
    "skills": 0.60,
    "experience": 0.25,
    "education": 0.15
}

scores = {
    "skills": skill_score,
    "experience": experience_score,
    "education": education_score
}

# Excludes unknown scores and adjusts the weights proportionally.
available_weight = sum(
    weights[category]
    for category, score in scores.items()
    if score is not None
)

if available_weight:
    overall_score = sum(
        weights[category] * score
        for category, score in scores.items()
        if score is not None
    ) / available_weight
else:
    overall_score = None


# FINAL MATCHING REPORT

print("\n========== RESUME MATCHING REPORT ==========")
print(f"Candidate: {resume_details.name}")

if overall_score is not None:
    print(f"Overall Match: {overall_score:.2f}%")
else:
    print("Overall Match: Cannot be calculated")

if skill_score is not None:
    print(f"\nSkill Match: {skill_score:.2f}%")
    print(f"Matched Skills: {len(matched_skills)} out of {len(required_skills)}")
    print("Matched:", ", ".join(sorted(matched_skills)) or "None")
    print("Not found:", ", ".join(sorted(missing_skills)) or "None")
else:
    print("\nSkill Match: No required skills were provided")

if experience_score is not None:
    print(f"\nExperience Match: {experience_score:.2f}%")
    print(f"Required Experience: {required_experience} years")
    print(f"Estimated Relevant Experience: {candidate_experience} years")
else:
    print("\nExperience Match: Unable to verify")

if education_score is not None:
    print(f"\nEducation Match: {education_score:.2f}%")
else:
    print("\nEducation Match: Unable to verify")

print("============================================")
