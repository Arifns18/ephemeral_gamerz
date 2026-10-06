# Quick Guide

How to set up the project on a new machine, and the commands you'll use every day.

All commands are for **Git Bash** on Windows and start from the repo root (`/e/Coding/ephemeral_gamerz`) unless noted.

---

## 1. First-time setup

Do this once per machine (or after a fresh clone).

```bash
# Go to the repo root
cd /e/Coding/ephemeral_gamerz

# Create the virtual environment in .venv/ (already ignored by git)
python -m venv .venv

# Activate it. Your prompt should now show (.venv)
source .venv/Scripts/activate

# Install the project's packages
python -m pip install --upgrade pip
pip install -r requirements.txt

# Create your local settings file from the template
cd ephemeral_gamerz
cp .env.example .env
```

Open `ephemeral_gamerz/.env` and replace `SECRET_KEY=change-me` with a real key. Generate one with:

```bash
python -c "from django.core.management.utils import get_random_secret_key; print(get_random_secret_key())"
```

Paste it into `.env` inside single quotes, e.g. `SECRET_KEY='abc...xyz'`. Keys can contain `$`, `#` and `&`, and the quotes stop those characters from being misread.

Then set up the database and start the server:

```bash
python manage.py migrate
python manage.py createsuperuser   # an admin account for /admin
python manage.py runserver
```

Open http://127.0.0.1:8000 in your browser.

### Point your editor at the venv

- **PyCharm:** Settings → Project → Python Interpreter → Add Interpreter → Existing → choose `.venv\Scripts\python.exe`.
  To make PyCharm's built-in terminal use Git Bash: Settings → Tools → Terminal → Shell path → `C:\Program Files\Git\bin\bash.exe`.
- **VS Code:** `Ctrl+Shift+P` → "Python: Select Interpreter" → choose `.venv`.

---

## 2. Every working session

```bash
# Start: go to the repo and activate the venv (every new terminal)
cd /e/Coding/ephemeral_gamerz
source .venv/Scripts/activate

# Run the site
cd ephemeral_gamerz
python manage.py runserver
# Stop the server with Ctrl+C

# Finish: leave the venv
deactivate
```

> **How do I know the venv is active?** Your prompt shows `(.venv)`, and `which python` prints a path ending in `.venv/Scripts/python`. If it doesn't, `pip install` will put packages in your global Python instead of this project.

---

## 3. After installing a new package

Every time you `pip install` something new, update `requirements.txt` so other machines get it too:

```bash
pip install <package-name>
pip freeze > requirements.txt
git add requirements.txt
```

> Run these from the **repo root** so `requirements.txt` lands next to `README.md`, and always with the venv **active**. Otherwise you'll record every package from your global Python.

---

## 4. After changing a model

Whenever you add or change fields in a `models.py`:

```bash
cd ephemeral_gamerz
python manage.py makemigrations
python manage.py migrate
```

Commit the new files in the app's `migrations/` folder along with the model change.

---

## 5. When you add a new setting to `.env`

1. Add the real value to `ephemeral_gamerz/.env` (never committed).
2. Add the same line with a placeholder value to `ephemeral_gamerz/.env.example` (committed).
3. Read it in `settings.py` with `env('NAME')`.

---

## 6. Before you commit

```bash
git status
```

Check that none of these appear in the list. `.gitignore` should already keep them out:

- `.env`
- `db.sqlite3`
- `.venv/`
- `__pycache__/`

---

## Troubleshooting (Git Bash)

**`source .venv/Scripts/activate` says "No such file or directory"**
You're not in the repo root. Run `cd /e/Coding/ephemeral_gamerz` first. Note that the venv uses `Scripts/`, not `bin/`, on Windows.

**`createsuperuser` or `manage.py shell` freezes, or your password is shown as you type it**
Older Git Bash versions have trouble with interactive Python prompts. Run the command through `winpty`:

```bash
winpty python manage.py createsuperuser
winpty python manage.py shell
```

Updating Git for Windows usually fixes this permanently.

**`python` opens the Microsoft Store or says "command not found"**
The venv isn't active, and Windows can't find Python. Activate the venv, or check that Python is on your `PATH`.

---

## Cheat sheet

| I want to… | Command |
|---|---|
| Go to the project | `cd /e/Coding/ephemeral_gamerz` |
| Activate the venv | `source .venv/Scripts/activate` |
| Leave the venv | `deactivate` |
| Check which Python is in use | `which python` |
| Install everything for the project | `pip install -r requirements.txt` |
| Save installed packages | `pip freeze > requirements.txt` |
| Run the site | `python manage.py runserver` |
| Create migrations after a model change | `python manage.py makemigrations` |
| Apply migrations | `python manage.py migrate` |
| Make an admin account | `python manage.py createsuperuser` |
| Check the project for problems | `python manage.py check` |
| Create a new app | `python manage.py startapp <name>` |
