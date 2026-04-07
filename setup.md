# One-time machine setup (never again after this)
brew install pipx
pipx install poetry
poetry config virtualenvs.in-project true
pipx inject poetry poetry-plugin-export 

# Per-project setup
cd your-project-folder
poetry init   # follow the prompts

# Add this to the TOP of pyproject.toml:
[tool.poetry]
package-mode = false

[tool.poetry.requires-plugins]
poetry-plugin-export = ">=1.8"

# Create the venv and lockfile
poetry install

# Add packages
poetry add numpy pandas scikit-learn 
poetry add --group dev pytest ruff pre-commit 

# Add pre-commit.yaml
touch .pre-commit-config.yaml

paste into pre-commit:
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.3.4
    hooks:
      - id: ruff
        args: [ --fix ]
      - id: ruff-format 

# Add ci.yml file
mkdir -p .github/workflows
touch .github/workflows/ci.yml


name: CI Pipeline

on:
  push:
    branches: [ main, master ]
  pull_request:
    branches: [ main, master ]

jobs:
  quality-check:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'

      - name: Install Poetry
        uses: snok/install-poetry@v1
        with:
          version: 2.3.3 
          virtualenvs-create: true
          virtualenvs-in-project: true

      - name: Cache dependencies
        id: cached-poetry-dependencies
        uses: actions/cache@v4
        with:
          path: .venv
          key: venv-${{ runner.os }}-${{ hashFiles('**/poetry.lock') }}

      - name: Install dependencies
        if: steps.cached-poetry-dependencies.outputs.cache-hit != 'true'
        run: poetry install --no-interaction --no-root

      - name: Run Ruff (Linting & Formatting)
        run: poetry run ruff check .

      - name: Run Tests (Logic Check)
        run: poetry run pytest


# Install pre-commit
poetry run pre-commit install

# Export requirements.txt (run this before committing)
poetry export -f requirements.txt --output requirements.txt --without-hashes

# Add to .gitignore
.venv/
__pycache__/
*.pyc