# Part 1: Getting Started - From Zero to Running Code

> **Reading time:** ~15 minutes

A practical guide to setting up your development environment and understanding how real projects are organized. This guide assumes macOS.

**Next**: [Introduction & Table of Contents](00-introduction.md)

---

## Table of Contents

1. [Understanding Project Structure](#1-understanding-project-structure)
2. [Git and GitHub Basics](#2-git-and-github-basics)
3. [Python and uv Setup](#3-python-and-uv-setup)
4. [Getting the Codebase Running](#4-getting-the-codebase-running)
5. [Making Your First Changes](#5-making-your-first-changes)

---

## 1. Understanding Project Structure

### What is Project Structure?

**Project structure** is how code files are organized in a project. A well-organized codebase makes it easy to find and understand code.

#### Real-World Analogy

Think of it like organizing a library - fiction in one section, non-fiction in another, reference materials separate. Same concept with code.

#### Typical Django Project Structure

```
project-root/
├── README.md                    # Project documentation
├── pyproject.toml              # Dependencies and tool config
├── .gitignore                  # Files Git should ignore
├── manage.py                   # Django command-line utility
├── myapp/                      # Main Django app
│   ├── __init__.py
│   ├── models.py              # Database models
│   ├── views.py               # Request handlers/API endpoints
│   ├── urls.py                # URL routing
│   ├── tasks.py               # Background tasks (Celery)
│   └── tests/                 # Tests
├── frontend/                   # React/TypeScript code
│   ├── components/            # UI components
│   ├── hooks/                 # React hooks
│   └── pages/                 # Page components
├── scripts/                    # Utility scripts
└── data/                      # Data files
```

#### Key Files

- `pyproject.toml` - Python dependencies and tool configuration
- `manage.py` - Django management commands
- `.env` - Environment variables (secrets, config)
- `README.md` - Setup instructions

---

## 2. Git and GitHub Basics

### What is Git?

**Git** tracks changes to your code over time. Think of it like "Track Changes" in Word, but for code.

### What is GitHub?

**GitHub** hosts Git repositories (code projects) online. It's where teams collaborate.

### Why Use Git?

- **History**: See every change ever made
- **Collaboration**: Multiple people work on same code
- **Backup**: Code stored safely online
- **Branching**: Try changes without breaking working code

### Git Workflow

#### Clone a Repository

```bash
git clone https://github.com/organization/project.git
cd project
```

#### Check Status

```bash
git status  # See what files changed
```

#### Make Changes and Commit

```bash
# Create a branch
git checkout -b feature/my-changes

# Make changes to files...

# Stage changes
git add .

# Commit with descriptive message
git commit -m "Add user profile endpoint"
```

#### Push to GitHub

```bash
git push origin feature/my-changes
```

#### Pull Latest Changes

```bash
git checkout main
git pull origin main
```

### Branch Naming

Follow your team's convention:
- `initials/123-short-description` - Working on issue #123
- `feature/add-dashboard` - New feature
- `fix/login-bug` - Bug fix

### Common Commands

```bash
git status              # Check current state
git log --oneline       # View commit history
git diff                # See unstaged changes
git checkout main       # Switch to main branch
git pull                # Get latest changes
```

---

## 3. Python and uv Setup

### Installing Homebrew

**Homebrew** is a package manager for macOS - it makes installing developer tools and software easy.

Think of it like the App Store, but for command-line tools. Instead of manually downloading, installing, and updating software, Homebrew handles it all with simple commands.

```bash
# Install Homebrew (if you don't have it)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Verify installation
brew --version
```

**Common Homebrew commands:**
```bash
brew install <package>     # Install software
brew uninstall <package>   # Remove software
brew upgrade <package>     # Update specific package
brew upgrade               # Update all packages
brew list                  # Show installed packages
brew services start <pkg>  # Start a service (like PostgreSQL)
```

You'll use Homebrew throughout this guide to install PostgreSQL, Redis, and other tools.

### Installing uv

**uv** is a fast Python package manager that handles everything - dependencies, virtual environments, and Python versions.

```bash
# Install uv
curl -LsSf https://astral.sh/uv/install.sh | sh

# Verify
uv --version
```

**uv will automatically install Python** when you first run `uv sync` in a project. It reads the required Python version from `pyproject.toml` and installs it if needed.

### Understanding pyproject.toml

This file defines your project's dependencies and configuration:

```toml
[project]
name = "my-project"
requires-python = ">=3.13"
dependencies = [
    "django>=5.2",
    "psycopg[binary]>=3.2",
    "celery>=5.4",
]

[dependency-groups]
dev = [
    "pytest>=8.4",
    "ruff>=0.12",
]
```

**Main sections:**
- `dependencies` - Required to run the app
- `dependency-groups.dev` - Development tools (testing, linting)
- `dependency-groups.web` - Web server dependencies
- `tool.*` - Configuration for tools (ruff, pytest, pyright)

### Using uv

```bash
# Install all dependencies from pyproject.toml
uv sync

# Run commands in the project environment
uv run python manage.py runserver
uv run pytest

# Add a new dependency
uv add requests

# Add a dev dependency
uv add --dev pytest-cov

# Update dependencies
uv sync --upgrade
```

**uv automatically:**
- Creates a virtual environment in `.venv/`
- Installs exact versions from `uv.lock`
- Manages Python versions
- Handles dependency resolution

No need to manually activate virtual environments or use `pip`!

---

## 4. Getting the Codebase Running

### Step-by-Step Setup

#### 1. Clone and Enter Directory

```bash
git clone https://github.com/organization/project.git
cd project
```

#### 2. Read the README

Always start here - it has project-specific setup instructions.

#### 3. Install Dependencies

```bash
uv sync
```

This installs everything from `pyproject.toml` and creates `uv.lock`.

#### 4. Set Up Environment Variables

```bash
# Copy example file
cp .env.example .env

# Edit with your values
nano .env
```

**Common variables:**
```bash
DEBUG=True
DATABASE_URL=postgresql://localhost/project_db
REDIS_URL=redis://localhost:6379
SECRET_KEY=your-secret-key
```

#### 5. Set Up the Database

```bash
# Install PostgreSQL
brew install postgresql@16
brew services start postgresql@16

# Create database
createdb project_db

# Run migrations (creates tables)
uv run python manage.py migrate

# Create admin user (optional)
uv run python manage.py createsuperuser
```

#### 6. Set Up Redis (for caching/Celery)

```bash
# Install Redis
brew install redis
brew services start redis

# Verify it's running
redis-cli ping  # Should return "PONG"
```

#### 7. Run the Application

```bash
# Development server
uv run python manage.py runserver

# Or with specific port
uv run python manage.py runserver 8080
```

**Open in browser:** http://localhost:8000

#### 8. Run Background Worker (if using Celery)

In a separate terminal:

```bash
uv run celery -A projectname worker -l info
```

### Common Issues

**"Module not found"**
```bash
uv sync  # Install dependencies
```

**"Connection refused" (database)**
```bash
brew services start postgresql@16
```

**"Port already in use"**
```bash
# Kill process on port 8000
lsof -ti:8000 | xargs kill -9
```

---

## 5. Making Your First Changes

### Finding What to Work On

Look for issues labeled:
- `good first issue`
- `beginner friendly`
- Documentation improvements
- Small bug fixes

### Making a Change

#### 1. Create a Branch

```bash
git checkout main
git pull origin main
git checkout -b jg/123-fix-typo  # Use your initials and issue number
```

#### 2. Make Your Changes

Example: Add a simple utility function

```python
# In myapp/utils.py

def format_currency(amount: float) -> str:
    """Format amount as USD currency string."""
    return f"${amount:,.2f}"
```

#### 3. Add Tests

```python
# In myapp/tests/test_utils.py

from myapp.utils import format_currency

def test_format_currency():
    assert format_currency(1234.5) == "$1,234.50"
    assert format_currency(0) == "$0.00"
```

#### 4. Run Tests

```bash
uv run pytest
```

#### 5. Format and Lint

```bash
# Format code
uv run ruff format .

# Check for issues
uv run ruff check .

# Type check
uv run pyright
```

#### 6. Commit Your Changes

```bash
git add .
git commit -m "Add currency formatting utility function

- Created format_currency function
- Added tests
- Fixes #123"

git push origin jg/123-fix-typo
```

#### 7. Create Pull Request

On GitHub:
1. Go to repository
2. Click "Pull requests" → "New pull request"
3. Select your branch
4. Fill in description
5. Submit for review

### Good Commit Messages

**Good:**
```
Add user authentication endpoint

- Created /api/auth/login endpoint
- Added JWT token generation
- Included tests for success and error cases
```

**Bad:**
```
fix stuff
```

### Code Review

After submitting:
- Maintainers review your code
- They may request changes
- Make changes and push to same branch
- PR updates automatically

### Best Practices

**Before committing:**
- [ ] Code runs without errors
- [ ] Tests pass (`uv run pytest`)
- [ ] Code is formatted (`uv run ruff format .`)
- [ ] No linting errors (`uv run ruff check .`)
- [ ] Commit message is descriptive

**Keep changes focused:**
- One feature/fix per PR
- Don't mix unrelated changes
- Update tests when changing code

---

## Quick Reference

### Essential Commands

```bash
# Git
git clone <url>                  # Get code
git checkout -b <branch>         # Create branch
git add .                        # Stage changes
git commit -m "message"          # Commit
git push origin <branch>         # Push to GitHub
git pull origin main             # Get latest

# uv + Python
uv sync                          # Install dependencies
uv run python manage.py <cmd>   # Run Django command
uv run pytest                    # Run tests
uv run ruff format .             # Format code
uv run ruff check .              # Lint code
uv run pyright                   # Type check

# Django
uv run python manage.py runserver        # Start server
uv run python manage.py migrate          # Run migrations
uv run python manage.py makemigrations   # Create migrations
uv run python manage.py shell            # Python shell
uv run python manage.py test             # Run Django tests

# Database
createdb dbname                  # Create database
psql dbname                      # Connect to database
brew services start postgresql@16 # Start PostgreSQL

# Redis
brew services start redis         # Start Redis
redis-cli                         # Redis CLI
```

### Project-Specific Tools

Based on your `pyproject.toml`:
- **ruff** - Code formatting and linting
- **pytest** - Testing framework
- **pyright** - Type checking
- **djlint** - Template linting (for Django templates)

---

## Next Steps

Now that you understand the practical setup, learn the concepts:

**Continue to:** **[Part 2: Foundational Mental Models](02-foundational-mental-models.md)** - Understanding how code really works
