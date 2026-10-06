# ESP32 TinyML Weather Prediction

A **portable, handheld weather forecaster** that runs a tiny machine-learning model directly on an ESP32 microcontroller. It uses local measurements (temperature, humidity, barometric pressure) to predict short-term weather **without an internet connection**, which is useful for hikers in remote areas.

**Course:** EE 300W, Section 003L, Penn State
**Team:** Sadid Hassan, Dhaval Patel, Tirth Patel, Jaden Carroll, Tyler Dows

> **Project status:** early development. The README only describes what is actually done. See [Project Status](#project-status) below.

---

## Table of Contents

1. [What does this project do?](#what-does-this-project-do)
2. [Project status](#project-status)
3. [Getting started (Windows)](#getting-started-windows)
4. [Running the notebooks](#running-the-notebooks)
5. [Working together with Git](#working-together-with-git)
6. [Repository structure](#repository-structure)
7. [Troubleshooting](#troubleshooting)
8. [Plain-English glossary](#plain-english-glossary)

---

## What does this project do?

```
Sensors  ->  ESP32  ->  Tiny ML model  ->  Forecast  ->  E-paper display
(temp, humidity,        (runs on the        (Clear /
 pressure)               chip itself)        Precipitation /
                                             Changing)
```

**The model's job:** look at the recent *trend* of temperature, humidity, and pressure (for example, "pressure has been dropping for 3 hours") and predict what the weather will do over the next few hours.

**Planned prediction categories:**

| Class | Meaning |
|---|---|
| Clear | No precipitation expected |
| Precipitation | Rain, drizzle, snow, or thunderstorms expected |
| Changing | No precipitation yet, but conditions are shifting (for example, pressure falling quickly) |

The exact definition of "Changing" will be set from the data, not guessed.

**Why train on airport data?** Our physical sensors are not available yet, so we train on years of public weather records from **University Park Airport (KUNV)**, then later test and adjust the model with our own device's measurements.

**Why only temperature, humidity, and pressure?** Those are what our handheld device can measure. Wind and rain are in the airport data, but our device has no wind or rain sensor, so they are never used as model *inputs*. Rain information is only used to create the "answer key" that the model learns from.

---

## Project status

| Stage | Status |
|---|---|
| Python / TensorFlow environment | Done |
| Download KUNV historical data (2015 to 2025) | Done |
| Explore and understand the data | Done (`01_weather_data_exploration.ipynb`) |
| Clean data, resample to hourly, build labels | In progress (`02_data_preprocessing.ipynb`) |
| Build input features and split data for training | Not started |
| Build and train TensorFlow model | Not started |
| Evaluate model | Not started |
| Convert to TensorFlow Lite and shrink for ESP32 | Not started |
| Run on the ESP32 with real sensors | Not started |

---

## Getting started (Windows)

You only need to do this **once**. It takes about 15 to 20 minutes, mostly waiting for downloads.

### Step 1: Install the tools

Install these three programs if you don't already have them:

1. **Python 3.11**: https://www.python.org/downloads/
   - During installation, **check the box "Add python.exe to PATH"** on the first screen.
   - Use **3.11**. TensorFlow does not always work with the newest Python version.
2. **Git**: https://git-scm.com/download/win (the default options are fine)
3. **Visual Studio Code**: https://code.visualstudio.com/

Then open VS Code, click the **Extensions** icon in the left bar (four squares), and install:
- **Python** (by Microsoft)
- **Jupyter** (by Microsoft)

### Step 2: Get access to the repository

The repository is **private**. Ask me (Sadid) to invite you as a collaborator, then accept the invitation email from GitHub. You will not be able to download the code until you accept. This might change if I make the repo public, in which case just ignore this step.

### Step 3: Download the project ("clone")

Open **PowerShell** (press the Windows key, type `PowerShell`, press Enter) and run:

```powershell
cd $HOME\Documents
git clone https://github.com/sadid-hassan/ESP32-TinyML-Weather-Prediction.git
cd ESP32-TinyML-Weather-Prediction
```

This creates a folder called `ESP32-TinyML-Weather-Prediction` in your Documents.

### Step 4: Open the folder in VS Code

In PowerShell, still inside the project folder, run:

```powershell
code .
```

(Or in VS Code: **File > Open Folder** and choose the project folder.)

Open the terminal inside VS Code with **Ctrl + `** (the key above Tab). **All the commands below go in this VS Code terminal**, and it should say PowerShell.

### Step 5: Create your own Python environment

A **virtual environment** (`.venv`) is a private box of Python packages for this project, so it doesn't interfere with anything else on your computer. It is **not** stored on GitHub, so every teammate has to create their own.

```powershell
py -3.11 -m venv .venv
.\.venv\Scripts\Activate.ps1
```

After the second command your prompt should start with **`(.venv)`**. That means the environment is active.

> **If you see a red error about "running scripts is disabled":** run this once, then try the activate command again:
> ```powershell
> Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned
> ```

### Step 6: Install the packages

```powershell
python -m pip install --upgrade pip
pip install -r requirements.txt
```

This installs TensorFlow, Pandas, and the other libraries listed in `requirements.txt`. **It can take several minutes** because TensorFlow is large. A wall of text scrolling by is normal.

### Step 7: Create the data folders

Git does not store empty folders, and the data is deliberately not on GitHub (it is large and can be re-downloaded), so create them yourself:

```powershell
mkdir data\raw
mkdir data\processed
```

(If PowerShell says the folder already exists, that's fine.)

### Step 8: Check that everything works

```powershell
python -c "import tensorflow as tf; import pandas as pd; print('TensorFlow', tf.__version__, '| Pandas', pd.__version__)"
```

If it prints version numbers, you're ready. A warning about GPU support is **normal and harmless**. Our model is tiny and trains fine on a regular CPU.

---

## Running the notebooks

A **Jupyter notebook** (`.ipynb` file) is a document made of small blocks called **cells**. Some cells contain explanation text, and some contain Python code you can run one at a time and see the results (tables, graphs) right below.

### Opening and running

1. In VS Code, open the `notebooks` folder and click a notebook.
2. **Pick the right kernel** (the Python environment the notebook uses). Click **Select Kernel** in the top right, then **Python Environments**, then choose the one that says **`.venv`**. *This is the step people most often forget.*
3. Run a cell by clicking it and pressing **Shift + Enter**. Run cells **in order from the top**, because later cells depend on earlier ones.

### Notebook order

| Notebook | What it does |
|---|---|
| `01_weather_data_exploration.ipynb` | Downloads the airport data and examines it. **Run this one first**, since it downloads the data file the others need. |
| `02_data_preprocessing.ipynb` | Cleans the data and prepares it for the model. |

The download in notebook 01 saves a ~27 MB file to `data/raw/`. It only downloads once, and if the file already exists the notebook skips the download. If you get an error that the folder doesn't exist, you skipped Step 7.

---

## Working together with Git

**Git** tracks changes to the project. **GitHub** is the website that stores the shared copy. Think of it as shared Google Docs with a very detailed history.

### The daily routine

**Before you start working**, always get the newest version:

```powershell
git pull
```

**After finishing a meaningful chunk of work**, save and share it:

```powershell
git add .
git commit -m "Short description of what you did"
git push
```

- `git add .` picks which changes to include.
- `git commit -m "..."` saves a snapshot on your computer with a note.
- `git push` uploads your commits to GitHub so everyone else can get them.

### Rules that save headaches

1. **Always `git pull` before you start.** Otherwise you may be working on an old version.
2. **Tell the team before editing a notebook someone else is working on.** Notebooks are stored in a format that Git can't merge well, and two people editing the same notebook creates painful conflicts.
3. **Never commit passwords, API keys, or personal information.**
4. **Don't commit datasets or the `.venv` folder.** They are already listed in `.gitignore`, so Git ignores them automatically. Run `git status` before `git add .` if you are unsure. It should not list any `.csv` files or `.venv`.
5. **Small, clear commit messages** like `Add pressure trend features`, not `stuff` or `final final v2`.

---

## Repository structure

```
ESP32-TinyML-Weather-Prediction/
|
|-- notebooks/        Step-by-step Jupyter notebooks (exploration, cleaning, training...)
|-- data/
|   |-- raw/          Original downloaded weather data (NOT on GitHub)
|   `-- processed/    Cleaned data ready for the model (NOT on GitHub)
|-- models/           Trained model files (added later)
|-- src/              Reusable Python code (added as the project matures)
|-- requirements.txt  List of Python packages the project needs
|-- .gitignore        Tells Git which files NOT to upload
`-- README.md         This file
```

---

## Troubleshooting

| Problem | Fix |
|---|---|
| `py` or `python` is "not recognized" | Python is not installed or not on PATH. Reinstall Python 3.11 and check **Add python.exe to PATH**. Then close and reopen VS Code. |
| Red error when running `Activate.ps1` | Run `Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned`, then retry. |
| Prompt doesn't show `(.venv)` | Run `.\.venv\Scripts\Activate.ps1` again. Do this each time you open a new terminal. |
| `ModuleNotFoundError: No module named 'pandas'` (or tensorflow, etc.) | Either the notebook is using the wrong kernel (select the `.venv` kernel) or packages are not installed (activate `.venv`, run `pip install -r requirements.txt`). |
| Notebook can't find the data file | Run notebook 01 first, and make sure `data\raw` exists (Step 7). |
| `git clone` says repository not found | You haven't accepted the GitHub invitation, or you aren't signed in. Ask Sadid. |
| `git push` is rejected | Someone pushed first. Run `git pull`, then `git push` again. |
| TensorFlow warns about GPU | Ignore it. It's expected on Windows. |

Still stuck? Copy the **full** error message and send it to the team chat, with what you were doing when it happened.

---

## Plain-English glossary

| Term | Meaning |
|---|---|
| **ESP32** | A small, low-power microcontroller (a tiny computer) that will run our device. |
| **Machine learning (ML)** | Instead of writing rules by hand, we show a program many examples and it finds the patterns itself. |
| **TinyML** | Machine learning small enough to run on a microcontroller. |
| **TensorFlow** | The Python library we use to build and train the model. |
| **TensorFlow Lite** | A shrunken version of a trained model that fits on the ESP32. |
| **Model** | The trained "pattern finder." It takes in measurements and outputs a prediction. |
| **Training** | Showing the model past examples so it learns which patterns come before rain. |
| **Features (inputs)** | What the model sees: temperature, humidity, pressure, and how they changed recently. |
| **Label (target)** | The correct answer the model is trying to learn, such as "precipitation happened within 3 hours." |
| **Forecast horizon** | How far ahead we predict (currently 3 hours). |
| **ASOS / METAR** | The automated weather stations at airports and the standard report format they publish. We use KUNV's records. |
| **Pandas** | A Python library for working with tables of data. |
| **Data leakage** | Accidentally letting the model peek at the future during training. It looks great in testing and fails in real life, and we are actively avoiding it. |
| **Virtual environment (`.venv`)** | A private set of Python packages for one project. |

---

*This README will grow as the project does. Results, model details, and ESP32 deployment instructions will be added once they actually exist.*
