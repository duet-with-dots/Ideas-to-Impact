# 🤖 Simple Agents: Coze Artifacts

Welcome to **Simple Agents**! This repository is a curated collection of specialized prompts and sample artifacts designed for creating powerful, focused AI agents on platforms like [Coze](https://www.coze.com/).

Whether you're looking to automate recruitment pipelines or stay updated on the latest tech trends, these agent prompts are ready to be deployed and customized.

## 📂 Project Structure

The repository is organized into the following key areas:

```text
simple-agents/
└── coze-artifacts/
    ├── prompts/           # 🧠 The core system & user prompts for each AI agent
    └── sample-resumes/    # 📄 Example data for testing the recruitment agents
```

## 🚀 Available Agents

### 1. 📄 Resume Extractor (`resume-extractor.md`)
An AI-powered Resume Analyzer designed to parse candidate resumes efficiently.
* **What it does:** Extracts core hard and soft skills, technologies, and methodologies.
* **Key Feature:** Assigns a percentage or 1-10 scale weight to each skill based on recency, duration of use, and depth of expertise.
* **Output:** A scannable summary report categorized logically (e.g., Languages, Frameworks, Cloud/Infrastructure).

### 2. 🎯 Skill Matcher (`skill-matcher.md`)
A precise recruitment assistant that compares a candidate's resume against a specific Job Description (JD).
* **What it does:** Calculates an overall match score (0-100%) based on qualifications, technical skills, and experience.
* **Key Feature:** Provides a detailed breakdown including matching skills, missing skills (gaps), key strengths, and actionable improvement suggestions.
* **Default Target Role:** Machine Learning Engineer (can be customized via the prompt).

### 3. 📈 Tech Trend Assistant (`tech-trend-assistant.md`)
A specialized news briefing agent tailored for technical founders, developers, and AI practitioners.
* **What it does:** Identifies the top 5 biggest shifts happening in Generative AI and Enterprise Software for the current month.
* **Key Feature:** Uses web search to provide specific, data-grounded trends, explaining *why* it's happening, the business implications, and citing real sources.

## 🧪 Testing with Sample Data

To help you get started immediately, we have included sample documents in the `coze-artifacts/sample-resumes/` directory:
- `Alex Turing_resume.docx`
- `Jordan Patel_resume.docx`

You can feed these files into your Coze agents using the **Resume Extractor** or **Skill Matcher** prompts to see how they perform in real-time.

## 🛠️ How to Use

1. **Copy a Prompt:** Navigate to `coze-artifacts/prompts/` and open your desired `.md` file.
2. **Configure your Agent:** Paste the `System Prompt` into the personality/instructions section of your AI agent platform (like Coze).
3. **Set up the User Input:** Use the provided `User Prompt` structure to handle incoming files or user queries.
4. **Test & Iterate:** Use the provided sample resumes to test the agent's output and tweak the prompt instructions as needed to fit your specific use case.

---
*Built to simplify complex workflows with focused AI agents.*
