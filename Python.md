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
