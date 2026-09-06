# 🎓 Study Buddy AI

> An AI-powered quiz generation and evaluation application built with **Python, Streamlit, LangChain, Groq LLMs, Pydantic, Docker, Jenkins, Kubernetes, and ArgoCD**.

Study Buddy AI allows users to generate personalized quizzes from any topic using an LLM. Users can select the question type, difficulty level, and number of questions, attempt the generated quiz, instantly evaluate their answers, and download their results as a CSV file.

The project also demonstrates an end-to-end **LLMOps / DevOps deployment workflow**, including containerization, CI/CD automation, Docker image publishing, Kubernetes deployment, and ArgoCD-based synchronization.

---

## 🚀 Features

* 🤖 AI-generated quiz questions using Groq LLM
* 🧠 Powered by Llama 3.1 8B Instant
* 📝 Multiple-choice questions
* ✍️ Fill-in-the-blank questions
* 🎯 Three difficulty levels:

  * Easy
  * Medium
  * Hard
* 🔢 Generate 1–10 questions per quiz
* ✅ Automatic answer evaluation
* 📊 Quiz score calculation
* 💾 Export quiz results to CSV
* 🔄 Retry mechanism for failed LLM responses
* 🛡️ Structured LLM output validation using Pydantic
* 📝 Application logging
* ⚠️ Custom exception handling
* 🐳 Docker containerization
* 🔧 Jenkins CI/CD pipeline
* ☸️ Kubernetes deployment
* 🔁 ArgoCD application synchronization

---

# 🏗️ Architecture

```text
                    ┌──────────────────────┐
                    │       User           │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │     Streamlit UI     │
                    │    application.py    │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │    Quiz Manager      │
                    │     helpers.py       │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │ Question Generator   │
                    │ question_generator.py│
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │   Prompt Templates   │
                    │     templates.py     │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │    LangChain         │
                    │    ChatGroq          │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │   Groq LLM API       │
                    │ llama-3.1-8b-instant  │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │ Pydantic Validation  │
                    │   MCQ / Fill Blank   │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │   Quiz Evaluation    │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │     CSV Results      │
                    └──────────────────────┘


        Deployment Pipeline
        ───────────────────

 GitHub
    │
    ▼
 Jenkins
    │
    ▼
 Docker Build
    │
    ▼
 Docker Hub
    │
    ▼
 Kubernetes
    │
    ▼
 ArgoCD
    │
    ▼
 Running Application
```

---

# 🛠️ Tech Stack

| Technology               | Purpose                                      |
| ------------------------ | -------------------------------------------- |
| **Python**               | Application development                      |
| **Streamlit**            | Interactive web interface                    |
| **LangChain**            | LLM integration and prompt handling          |
| **Groq**                 | LLM inference                                |
| **Llama 3.1 8B Instant** | Question generation                          |
| **Pydantic**             | Structured output validation                 |
| **Pandas**               | Result processing and CSV generation         |
| **python-dotenv**        | Environment variable management              |
| **Docker**               | Application containerization                 |
| **Jenkins**              | CI/CD automation                             |
| **Docker Hub**           | Container image registry                     |
| **Kubernetes**           | Container orchestration                      |
| **ArgoCD**               | Continuous delivery / GitOps synchronization |

---

# 📁 Project Structure

```text
STUDY-BUDDY-AI/
│
├── application.py
├── Dockerfile
├── Jenkinsfile
├── requirements.txt
├── setup.py
├── .gitignore
│
├── manifests/
│   ├── deployment.yaml
│   └── service.yaml
│
└── src/
    │
    ├── common/
    │   ├── custom_exception.py
    │   └── logger.py
    │
    ├── config/
    │   └── settings.py
    │
    ├── generator/
    │   └── question_generator.py
    │
    ├── llm/
    │   └── groq_client.py
    │
    ├── models/
    │   └── question_schemas.py
    │
    ├── prompts/
    │   └── templates.py
    │
    └── utils/
        └── helpers.py
```

---

# ⚙️ How the Application Works

## 1. User selects quiz configuration

The Streamlit interface allows the user to select:

```text
Question Type
    ├── Multiple Choice
    └── Fill in the Blank

Topic
    └── Any topic provided by the user

Difficulty
    ├── Easy
    ├── Medium
    └── Hard

Number of Questions
    └── 1–10
```

Example:

```text
Question Type: Multiple Choice
Topic: Indian History
Difficulty: Medium
Questions: 5
```

---

# 🤖 2. Question Generation

The application creates a `QuestionGenerator` object.

```python
generator = QuestionGenerator()
```

The generator connects to the Groq LLM through LangChain.

```python
def get_groq_llm():
    return ChatGroq(
        api_key=settings.GROQ_API_KEY,
        model=settings.MODEL_NAME,
        temperature=settings.TEMPERATURE
    )
```

The configured model is:

```text
llama-3.1-8b-instant
```

---

# 🧠 3. Prompt Generation

For an MCQ, the application sends a structured prompt similar to:

```text
Generate a medium multiple-choice question about Indian History.

Return ONLY a JSON object with:

question
options
correct_answer
```

The model is instructed to return exactly four options.

Example expected response:

```json
{
  "question": "Who founded the Maurya Empire?",
  "options": [
    "Ashoka",
    "Chandragupta Maurya",
    "Bindusara",
    "Harshavardhana"
  ],
  "correct_answer": "Chandragupta Maurya"
}
```

---

# 🔍 4. Structured Output Validation

The generated LLM response is parsed using Pydantic.

```python
parser = PydanticOutputParser(
    pydantic_object=MCQQuestion
)

question = parser.parse(response.content)
```

The MCQ schema contains:

```python
class MCQQuestion(BaseModel):

    question: str

    options: List[str]

    correct_answer: str
```

The application also performs additional validation:

```python
if len(question.options) != 4:
    raise ValueError("Invalid MCQ Structure")

if question.correct_answer not in question.options:
    raise ValueError("Invalid MCQ Structure")
```

This prevents malformed LLM responses from being directly used by the application.

---

# 🔄 5. Retry Mechanism

LLM responses can occasionally fail to parse or produce invalid output.

To handle this, the project implements a retry mechanism:

```python
for attempt in range(settings.MAX_RETRIES):

    try:
        response = self.llm.invoke(
            prompt.format(
                topic=topic,
                difficulty=difficulty
            )
        )

        parsed = parser.parse(response.content)

        return parsed

    except Exception as e:

        if attempt == settings.MAX_RETRIES - 1:
            raise CustomException(
                f"Generation failed after {settings.MAX_RETRIES} attempts",
                e
            )
```

The configured maximum retry count is:

```python
MAX_RETRIES = 3
```

---

# 📝 6. Fill-in-the-Blank Questions

The application also supports fill-in-the-blank questions.

Example:

```json
{
  "question": "The capital of France is _____.",
  "answer": "Paris"
}
```

The corresponding Pydantic model is:

```python
class FillBlankQuestion(BaseModel):

    question: str

    answer: str
```

The application additionally checks that the generated question contains the blank marker:

```python
if "___" not in question.question:
    raise ValueError(
        "Fill in blanks should contain '___'"
    )
```

---

# 🎯 7. Quiz Evaluation

After generating the questions, users answer them directly through the Streamlit interface.

For MCQs:

```python
user_answer == correct_answer
```

For fill-in-the-blank questions, the comparison is case-insensitive:

```python
user_ans.strip().lower() == \
q['correct_answer'].strip().lower()
```

The application creates a result dictionary containing:

```python
{
    "question_number": 1,
    "question": "...",
    "question_type": "MCQ",
    "user_answer": "...",
    "correct_answer": "...",
    "is_correct": True
}
```

---

# 📊 8. Score Calculation

The final score is calculated using:

```python
correct_count = results_df["is_correct"].sum()

total_questions = len(results_df)

score_percentage = (
    correct_count / total_questions
) * 100
```

For example:

```text
Correct Answers: 4
Total Questions: 5

Score: 80%
```

---

# 💾 9. Export Results

Quiz results can be saved as CSV files.

Example:

```text
results/
└── quiz_results_20260906_143025.csv
```

The application generates a timestamped filename:

```python
timestamp = datetime.now().strftime(
    "%Y%m%d_%H%M%S"
)

unique_filename = (
    f"quiz_results_{timestamp}.csv"
)
```

This allows multiple quiz attempts to be saved without overwriting previous results.

---

# 🔐 Environment Configuration

The Groq API key is loaded from an environment variable.

Create a `.env` file:

```env
GROQ_API_KEY=your_groq_api_key_here
```

The application loads it using:

```python
from dotenv import load_dotenv

load_dotenv()
```

The key is then accessed through:

```python
os.getenv("GROQ_API_KEY")
```

### ⚠️ Important

Never commit your `.env` file or API keys to GitHub.

The project already includes:

```text
.env
```

inside `.gitignore`.

---

# 💻 Local Installation

## 1. Clone the repository

```bash
git clone https://github.com/YOUR_USERNAME/STUDY-BUDDY-AI.git
```

Move into the project:

```bash
cd STUDY-BUDDY-AI
```

---

## 2. Create a virtual environment

### Windows

```bash
python -m venv venv
```

Activate it:

```bash
source venv/Scripts/activate
```

Or in PowerShell:

```powershell
venv\Scripts\Activate.ps1
```

---

## 3. Install dependencies

```bash
pip install -r requirements.txt
```

---

## 4. Configure the API key

Create:

```text
.env
```

Add:

```env
GROQ_API_KEY=your_groq_api_key
```

---

## 5. Run the application

```bash
streamlit run application.py
```

The application will start on:

```text
http://localhost:8501
```

---

# 🐳 Running with Docker

The project includes a Dockerfile for containerized deployment.

## Build the image

```bash
docker build -t study-buddy-ai .
```

## Run the container

```bash
docker run -p 8501:8501 \
  -e GROQ_API_KEY="your_groq_api_key" \
  study-buddy-ai
```

Open:

```text
http://localhost:8501
```

---

# ☸️ Kubernetes Deployment

The project contains Kubernetes manifests under:

```text
manifests/
```

### Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
```

The deployment runs:

```text
2 replicas
```

This allows multiple application pods to run simultaneously.

The application container exposes:

```text
8501
```

### Service

The Kubernetes service exposes the Streamlit application through a `NodePort`.

```yaml
kind: Service

ports:
  - port: 80
    targetPort: 8501

type: NodePort
```

---

# 🔄 CI/CD Pipeline

The project includes a Jenkins pipeline that automates the deployment process.

The pipeline follows:

```text
GitHub
   ↓
Jenkins Checkout
   ↓
Build Docker Image
   ↓
Push Image to Docker Hub
   ↓
Update Kubernetes Deployment
   ↓
Commit Updated Manifest
   ↓
Install kubectl / ArgoCD
   ↓
Sync Application with ArgoCD
```

---

# 🔧 Jenkins Pipeline Stages

## 1. Checkout GitHub

Jenkins checks out the `main` branch.

```groovy
stage('Checkout Github') {
    steps {
        checkout scmGit(...)
    }
}
```

---

## 2. Build Docker Image

A new Docker image is created for every build.

```groovy
IMAGE_TAG = "v${BUILD_NUMBER}"
```

For example:

```text
v15
v16
v17
```

---

## 3. Push to Docker Hub

The generated image is pushed to Docker Hub:

```text
dataguru97/studybuddy
```

---

## 4. Update Kubernetes Manifest

Jenkins automatically updates the Docker image tag inside:

```text
manifests/deployment.yaml
```

For example:

```yaml
image: dataguru97/studybuddy:v15
```

can become:

```yaml
image: dataguru97/studybuddy:v16
```

---

## 5. Commit Updated Manifest

The updated Kubernetes manifest is committed back to GitHub.

This creates a Git-based record of the deployed image version.

---

## 6. ArgoCD Synchronization

Finally, ArgoCD synchronizes the Kubernetes application:

```bash
argocd app sync study
```

This allows the Kubernetes environment to deploy the updated application version.

---

# 🛡️ Error Handling

The project implements custom exception handling through:

```text
src/common/custom_exception.py
```

Example:

```python
raise CustomException(
    "MCQ generation failed",
    e
)
```

The custom exception captures:

* Error message
* Original exception
* File name
* Line number

Example format:

```text
MCQ generation failed |
Error: ... |
File: ... |
Line: ...
```

This makes debugging easier during development and deployment.

---

# 📝 Logging

The application uses Python's built-in `logging` module.

Logs are automatically stored inside:

```text
logs/
```

Example:

```text
logs/
└── log_2026-09-06.log
```

The application records events such as:

```text
Generating question
Successfully parsed question
Generated valid MCQ
Generation errors
```

---

# 🧪 Example Usage

Start the application:

```bash
streamlit run application.py
```

Then configure:

```text
Question Type → Multiple Choice

Topic → Machine Learning

Difficulty → Medium

Number of Questions → 5
```

Click:

```text
Generate Quiz
```

The AI generates questions such as:

```text
Question:
Which algorithm is commonly used for classification?

A. Linear Regression
B. Logistic Regression
C. K-Means
D. PCA
```

After answering all questions:

```text
Submit Quiz
```

The application calculates the score and displays:

```text
Score: 80%
```

The results can then be saved and downloaded as a CSV file.

---

# 🔑 Configuration

Application settings are centralized in:

```text
src/config/settings.py
```

Current configuration includes:

```python
MODEL_NAME = "llama-3.1-8b-instant"

TEMPERATURE = 0.9

MAX_RETRIES = 3
```

This makes model-related configuration easy to modify without changing the main application logic.

---

# 📌 Important Design Decisions

### Structured LLM Output

Instead of directly trusting the raw LLM response, the application uses Pydantic models to validate the generated data.

### Retry Handling

LLM generation and parsing can fail, so the application retries generation up to three times.

### Separation of Responsibilities

The project separates:

```text
UI
│
├── Quiz Management
│
├── Question Generation
│
├── LLM Client
│
├── Prompt Templates
│
├── Data Models
│
├── Configuration
│
└── Error / Logging
```

This makes the application easier to maintain and extend.

---

# 🚀 Future Improvements

Potential improvements include:

* 👤 User authentication
* 📚 Quiz history and persistent database storage
* 📈 Student performance analytics
* 🎯 Adaptive difficulty based on previous performance
* 🧠 Personalized learning recommendations
* 🗃️ Question bank storage
* 🔎 Duplicate question detection
* 📊 Interactive performance dashboards
* ⚡ Async question generation
* 🧪 Automated unit and integration tests
* 🔐 Kubernetes Secrets for API credentials
* 📦 Helm-based Kubernetes deployment
* 🔄 Improved GitOps workflow
* 📡 Application monitoring and metrics

---

# 🎯 Learning Outcomes

This project demonstrates practical experience with:

* Python application development
* LLM application development
* Prompt engineering
* LangChain
* Groq API integration
* Pydantic data validation
* Streamlit
* Exception handling
* Application logging
* Environment-based configuration
* Docker
* Jenkins CI/CD
* Docker Hub
* Kubernetes
* ArgoCD
* Git-based deployment workflows

---

# 👨‍💻 Author

**Your Name**

Built as an LLMOps project demonstrating the development and deployment of an AI-powered educational application.

---

# ⭐ If You Like This Project

If this project was useful or interesting, consider giving the repository a ⭐ on GitHub.

---

## 📄 License

This project is available for educational and demonstration purposes.
