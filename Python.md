---

# 🚀 PYTHON TOPIC 1 — Virtual Environment (venv)

A **virtual environment (venv)** is like:

```
Maven local repository + isolated Java runtime inside project
```

It creates a **separate package space** for each project so dependencies don’t clash.

---

## 💡 Why do we need venv?

Without venv → packages install globally → version conflicts.

With venv:

| Project   | Django Version | Flask Version |
| --------- | -------------- | ------------- |
| project-A | Django 4.0     | Flask 3.1     |
| project-B | Django 3.2     | Flask 2.0     |

Both can coexist without conflict.
This is essential for CI servers also.

---

## 🔥 Create a Virtual Environment

Command:

```bash
python3 -m venv venv
```

BREAKDOWN:

| Word                | Meaning                                   |
| ------------------- | ----------------------------------------- |
| `python3`           | Interpreter                               |
| `-m`                | Run Python module as script               |
| `venv`              | Module that creates virtual environments  |
| `venv` (second one) | Name of the environment directory created |

This creates a folder:

```
venv/
 ├─ bin/ (Linux/Mac) or Scripts/ (Windows)
 ├─ lib/
 ├─ site-packages/  ← where dependencies will be installed
 └─ pyvenv.cfg
```

Think of `venv` as:

```
~/.m2/repository  (but isolated per project, not system-wide)
```

---

## 🔥 Activate Virtual Environment

### On Linux / Mac:

```bash
source venv/bin/activate
```

### On Windows:

```bash
venv\Scripts\activate
```

After activation, shell changes:

```
(venv) username@
```

Meaning:
Now you are inside **that project’s environment**, not system Python.

---

## 🔥 Deactivate venv

```bash
deactivate
```

Simple. This restores global interpreter.

---

# 🟢 Summary Flashcard for Topic 1

| Command                    | Meaning          |
| -------------------------- | ---------------- |
| `python3 -m venv venv`     | Create venv      |
| `source venv/bin/activate` | Activate         |
| `deactivate`               | Exit environment |

---

We now go into **Python Topic 2: `requirements.txt`**, the equivalent of **dependencies list in Maven**.

This file is critical in CI/CD because it defines **ALL Python packages needed to run the application**.

---

# 🔥 What is `requirements.txt`?

It’s a plain text file where we list all Python dependencies & their versions.

Equivalent comparison:

| Maven (Java)                | Python                  |
| --------------------------- | ----------------------- |
| `pom.xml` `<dependencies>`  | `requirements.txt`      |
| versioned dependencies      | pinned library versions |
| resolved from Maven central | resolved from PyPI      |

---

## 📝 Sample `requirements.txt`

```
flask==3.0.0
requests==2.31.0
pandas==2.2.1
numpy==1.26.4
gunicorn==21.2.0
```

> Double equals `==` means **fixed version** — no auto update.

---

## 🔎 Install packages from requirements.txt

Command:

```bash
pip install -r requirements.txt
```

Breakdown:

| Keyword            | Meaning                                              |
| ------------------ | ---------------------------------------------------- |
| `pip`              | Python package manager (equivalent to `mvn install`) |
| `install`          | install packages                                     |
| `-r`               | read dependencies from file                          |
| `requirements.txt` | file name                                            |

---

## 🔥 Generate requirements.txt automatically

If you already installed packages into venv:

```bash
pip freeze > requirements.txt
```

Breakdown:

| Part               | Meaning                                     |
| ------------------ | ------------------------------------------- |
| `pip freeze`       | prints all installed packages with versions |
| `>`                | writes output into file                     |
| `requirements.txt` | file created/overwritten                    |

This is commonly done in deployments & Docker builds.

---

## 🔥 Upgrade packages & update file

### Upgrade manually:

```bash
pip install --upgrade flask
```

Update file:

```bash
pip freeze > requirements.txt
```

---

## 🟡 Version Syntax (Important)

| Syntax           | Meaning                                   |
| ---------------- | ----------------------------------------- |
| `package==1.2.3` | exact version lock (recommended for prod) |
| `package>=1.2.0` | install this or newer                     |
| `package<=1.5.0` | allow up to version 1.5                   |
| `package~=1.2.3` | compatible releases only                  |

Example:

```
Django>=3.2,<4
```

Means:

✔ ≥3.2 allowed
❌ 4.x not allowed

---

## 🟢 requirements.txt in CI/CD Pipeline

Typical Jenkins pipeline step:

```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
python app.py
```

OR using Gunicorn in production:

```
gunicorn app:app --bind 0.0.0.0:8000
```

We will automate this when we reach deployment.

---

## 📌 Quick Flash Notes

```
requirements.txt = list of packages + versions
pip install -r requirements.txt → install everything
pip freeze > requirements.txt → generate file
Version pinning is IMPORTANT for stable deployments
```

---

# 🔥 What is `setup.py` Really?

It is just a **Python file used to package your project** — nothing more.

Think of it like:

```
setup.py = POM.xml only for packaging
```

But instead of XML, it is written in Python.

* If you **only run apps**, you don’t need setup.py
* If you **want to package/distribute your Python app**, you use setup.py

So unless you are **building tools/libraries to be installed**, setup.py is optional.

### 💡 One-line memory:

```
requirements.txt = Install dependencies to run a project
setup.py = Package the project itself
```

---

# Step-by-Step Understanding (Very Simple)

### If you create a Python project normally:

```
project/
 ├── app.py
 ├── requirements.txt
```

You can run it → NO setup.py needed.

---

### If you want this to install like a real program:

```
pip install myapp.whl
```

Then you need → setup.py
because setup.py **tells Python how to build/install the project**.

This is like converting a Python folder into a **product**.

---

# Now let's take the simplest setup.py possible

```python
from setuptools import setup

setup(name="myapp")
```

☑ That's a valid setup.py
☑ You don't need to understand everything

This just says:

> Package name = "myapp"

---

Now slowly expand it.

---

# Understand Full setup.py in Human Language

```python
from setuptools import setup, find_packages

setup(
    name="myapp",               # = project name (like artifactId)
    version="1.0.0",            # = release number
    packages=find_packages(),   # = auto detect python files/modules
    install_requires=[          # = dependencies needed to run project
        "flask==3.0.0",
        "requests==2.31.0"
    ],
)
```

### Full Meaning:

| Part               | Meaning                               |
| ------------------ | ------------------------------------- |
| `setup()`          | Like `<project>...</project>`         |
| `name`             | What the project will be installed as |
| `version`          | Version for release                   |
| `packages`         | What code to include                  |
| `install_requires` | Dependencies (like requirements.txt)  |

That's all setup.py does.

---

# A Visual Diagram (so brain locks it)

```
Your Python Project Folder
     ↓
 setup.py  ← tells how to package it
     ↓
 python setup.py bdist_wheel
     ↓
 .whl file generated  ← like JAR file
     ↓
 pip install myapp.whl  ← install like a real app
```

If you remember THIS flow, you understand setup.py.

---

# When you *don’t* need setup.py

| Situation                          | Need setup.py? |
| ---------------------------------- | -------------- |
| Running Flask/Django API           | ❌ Not needed   |
| Deploying to EC2/Lambda            | ❌ Not needed   |
| Packaging your project as CLI/tool | ✔ Yes          |
| Publishing to pip                  | ✔ Yes          |

So most people think setup.py is confusing because **they don’t need it yet**.

---

# Final Simple Summary for you

```
requirements.txt → packages you need
setup.py → package you become
```

You just didn't need setup.py — that's why it felt confusing.

---

### If you want — I can show you a **real mini project using setup.py**:

Example outcome → You type:

```
pip install .
```

and then run:

```
myapp
```

like a real CLI tool 🚀

If you want that practical demonstration reply:

```
Show practical example of setup.py
```

---

# 🚀 PYTHON TOPIC 4 — Deployment-Ready Build Artifacts

Unlike Java which produces a **JAR/WAR**,
Python apps are usually deployed as:

| Artifact Type             | Use Case                         |
| ------------------------- | -------------------------------- |
| `.zip` application bundle | EC2, Lambda deployments, servers |
| `.whl` (wheel) file       | Installable Python package       |
| Docker image (final prod) | Kubernetes / container deploys   |

In interviews, CI/CD pipelines usually expect you to know:

```
Python → install deps → bundle → deploy
```

---

## 1️⃣ **ZIP Based Deployment (Most common & easiest)**

Directory before zipping:

```
project/
 ├─ app.py
 ├─ utils.py
 ├─ requirements.txt
 └─ venv/ (not included in artifact)
```

Pipeline steps:

```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt -t package/
cp *.py package/
cd package && zip -r ../app.zip .
```

Final artifact = `app.zip`
Upload to server / Lambda / VM.

**AWS Deployment Example (Lambda):**

```
aws lambda update-function-code --function-name myapp --zip-file fileb://app.zip
```

This alone is enough to deploy Python microservices.

---

## 2️⃣ **Wheel Build (.whl)** — Python Equivalent of .jar

Wheel format is for packaging **installable Python distributions**.

Build command:

```bash
python setup.py bdist_wheel
```

Output:

```
dist/myapp-1.0.0-py3-none-any.whl
```

Install like a library:

```bash
pip install dist/myapp*.whl
```

Use case:
→ internal libraries shared across microservices
→ publish to private PyPI or Nexus

---

## 3️⃣ **Publishing Python Packages to Nexus / Artifactory**

```bash
pip install twine
twine upload --repository-url <nexus_repo_url> dist/*
```

Credentials stored in:

```
~/.pypirc
```

---

## 🔥 CI/CD Jenkins Pipeline for Python (Real Example)

```groovy
pipeline {
  agent any
  stages {
    stage('Install Dependencies') {
      steps {
        sh 'python3 -m venv venv'
        sh '. venv/bin/activate && pip install -r requirements.txt'
      }
    }
    stage('Build Artifact') {
      steps {
        sh 'zip -r app.zip .'
        archiveArtifacts artifacts: 'app.zip', onlyIfSuccessful: true
      }
    }
    stage('Deploy') {
      steps {
        sh 'scp app.zip user@server:/apps/'
      }
    }
  }
}
```

🔥 Equivalent to Maven pipeline but lighter.

---

# 🔥 PYTHON TOPIC 5 — Directory Structure (Standard Layout)

```
project/
├── src/
│   ├── main.py
│   ├── helpers/
│   │   └── formatter.py
├── tests/
│   └── test_main.py
├── requirements.txt
├── setup.py (optional)
└── README.md
```

CI/CD expects this format.

Interview tip:

> "Python projects use venv for isolation, requirements.txt for dependencies, and produce either a ZIP or Wheel artifact for deployment."

---

# 🔥 PYTHON TOPIC 6 — Environment Variables in Deployment

Used for DB credentials, secrets, config.

```python
import os

DB_HOST = os.getenv("DB_HOST")
API_KEY = os.getenv("API_KEY")
```

Set values in CI/CD:

```bash
export DB_HOST=db.prod.internal
export API_KEY=abc123
python app.py
```

or in Jenkins:

```
withCredentials([string(credentialsId: 'api-key', variable: 'API_KEY')]) {
    sh 'python app.py'
}
```

---

# 🔥 PYTHON TOPIC 7 — Logging & App Execution in Production

Use Gunicorn for web apps (Flask/FastAPI):

```bash
pip install gunicorn
gunicorn app:app --bind 0.0.0.0:8000 --workers 4
```

This is Python’s equivalent of **java -jar app.jar**.

Running Flask directly = dev mode
Running via Gunicorn/uvicorn = **production mode**

---

# 🚀 At this point, You Know Python Build Pipeline End-to-End

| You Can                                 | Status |
| --------------------------------------- | ------ |
| Create & activate venv                  | ✔      |
| Manage dependencies w/ requirements.txt | ✔      |
| Build deployable artifacts (ZIP/Wheel)  | ✔      |
| Use Jenkins pipeline to deploy Python   | ✔      |
| Run apps with Gunicorn in production    | ✔      |

You now understand Python build systems enough for interviews + real DevOps work.

---

