# Phase 1: Linux Foundations
## Weeks 1-3 | The Foundation of Everything

> **Prerequisites:** A computer with Windows (for WSL2) or Mac/Linux  
> **Time:** 10-15 hours per week  
> **Outcome:** Command line mastery, automation skills, version control expertise

---

## 🎯 Learning Objectives

By the end of Phase 1, you will:
- [ ] Navigate the Linux file system confidently
- [ ] Write bash scripts to automate tasks
- [ ] Use Git for version control professionally
- [ ] Understand permissions, processes, and system basics

---

## 📅 Weekly Breakdown

### Week 1: Linux Basics

**What you'll learn:**
- File system structure (`/`, `/home`, `/etc`, `/var`)
- Navigation commands (`cd`, `ls`, `pwd`, `find`)
- File operations (`cp`, `mv`, `rm`, `mkdir`, `touch`)
- Viewing files (`cat`, `less`, `head`, `tail`, `grep`)
- Permissions (`chmod`, `chown`, `rwx`)

**Read:**
| Resource | Section | Time |
|----------|---------|------|
| [WSL2-DevOps-Beginner-Guide.md](../../WSL2-DevOps-Beginner-Guide.md) | Phases 1-4 | 2 hours |
| [devop-complete-guide.md](../../devop-complete-guide.md) | Chapter 2: Linux Fundamentals | 3 hours |

**Hands-on exercises:**
```bash
# Exercise 1: Navigate and explore
cd /
ls -la
cd /etc
cat passwd
cd ~

# Exercise 2: Create project structure
mkdir -p ~/projects/my-app/{src,tests,docs}
touch ~/projects/my-app/README.md
tree ~/projects/my-app

# Exercise 3: Permissions
chmod 755 script.sh   # rwxr-xr-x
chmod 644 config.txt  # rw-r--r--
```

**Checkpoint quiz:**
1. What does `ls -la` show that `ls` doesn't?
2. What's the difference between `/home` and `~`?
3. What permission does `755` give?

---

### Week 2: Bash Scripting

**What you'll learn:**
- Variables and data types
- Conditionals (`if`, `else`, `elif`)
- Loops (`for`, `while`)
- Functions
- Input/output handling
- Error handling

**Read:**
| Resource | Section | Time |
|----------|---------|------|
| [devop-complete-guide.md](../../devop-complete-guide.md) | Chapter 2: Shell Scripting section | 3 hours |
| [Practical.md](../../Practical.md) | Bash scripting exercises | 2 hours |

**Hands-on exercises:**

```bash
#!/bin/bash
# Exercise 1: Variables and user input
# Save as: greeting.sh

echo "What's your name?"
read NAME
echo "Hello, $NAME! Welcome to DevOps."
```

```bash
#!/bin/bash
# Exercise 2: Conditionals
# Save as: check_file.sh

FILE=$1

if [ -f "$FILE" ]; then
    echo "$FILE exists and is a regular file"
elif [ -d "$FILE" ]; then
    echo "$FILE exists and is a directory"
else
    echo "$FILE does not exist"
fi
```

```bash
#!/bin/bash
# Exercise 3: Loops and functions
# Save as: backup.sh

backup_file() {
    local file=$1
    local backup_dir="$HOME/backups"
    
    mkdir -p "$backup_dir"
    cp "$file" "$backup_dir/$(basename $file).$(date +%Y%m%d)"
    echo "Backed up: $file"
}

# Backup all .txt files in current directory
for file in *.txt; do
    if [ -f "$file" ]; then
        backup_file "$file"
    fi
done

echo "Backup complete!"
```

**Checkpoint quiz:**
1. How do you make a script executable?
2. What does `$1` represent in a script?
3. How do you check if a file exists in bash?

---

### Week 3: Git Mastery

**What you'll learn:**
- Git fundamentals (init, add, commit, status)
- Branching strategies (feature branches, main/develop)
- Merging and rebasing
- Remote repositories (push, pull, fetch)
- Collaboration (pull requests, code review)

**Read:**
| Resource | Section | Time |
|----------|---------|------|
| [WSL2-DevOps-Beginner-Guide.md](../../WSL2-DevOps-Beginner-Guide.md) | Phase 5: Git & SSH Setup | 1 hour |
| [devop-complete-guide.md](../../devop-complete-guide.md) | Git sections | 2 hours |

**Hands-on exercises:**

```bash
# Exercise 1: Initialize and first commits
mkdir my-project && cd my-project
git init
echo "# My Project" > README.md
git add README.md
git commit -m "Initial commit"

# Exercise 2: Branching
git checkout -b feature/add-docs
echo "Documentation here" > docs.md
git add docs.md
git commit -m "feat: add documentation"
git checkout main
git merge feature/add-docs

# Exercise 3: Remote repository
git remote add origin git@github.com:username/repo.git
git push -u origin main

# Exercise 4: Feature branch workflow
git checkout -b feature/new-feature
# ... make changes ...
git add .
git commit -m "feat: implement new feature"
git push origin feature/new-feature
# Create PR on GitHub
```

**Git commands cheatsheet:**
```
git status          # What's changed?
git add .           # Stage all changes
git commit -m "msg" # Commit with message
git push            # Push to remote
git pull            # Pull from remote
git branch          # List branches
git checkout -b x   # Create and switch to branch x
git merge x         # Merge branch x into current
git log --oneline   # Compact history
git diff            # See unstaged changes
git stash           # Temporarily save changes
git stash pop       # Restore stashed changes
```

**Checkpoint quiz:**
1. What's the difference between `git add .` and `git add -A`?
2. How do you undo the last commit (keeping changes)?
3. What's the purpose of a pull request?

---

## 📋 Phase 1 Project: Automation Script

**Create a system setup script that:**
1. Creates a project directory structure
2. Initializes a git repository
3. Creates basic files (README, .gitignore)
4. Sets up a simple development environment

```bash
#!/bin/bash
# project-setup.sh - Phase 1 Final Project

set -e  # Exit on error

PROJECT_NAME=$1

if [ -z "$PROJECT_NAME" ]; then
    echo "Usage: ./project-setup.sh <project-name>"
    exit 1
fi

echo "Creating project: $PROJECT_NAME"

# Create directory structure
mkdir -p "$PROJECT_NAME"/{src,tests,docs,scripts}

# Initialize git
cd "$PROJECT_NAME"
git init

# Create README
cat > README.md << EOF
# $PROJECT_NAME

## Description
Add your project description here.

## Installation
\`\`\`bash
# Add installation steps
\`\`\`

## Usage
\`\`\`bash
# Add usage examples
\`\`\`
EOF

# Create .gitignore
cat > .gitignore << EOF
# Dependencies
node_modules/
venv/

# Environment
.env
.env.local

# Build
dist/
build/

# IDE
.vscode/
.idea/

# OS
.DS_Store
Thumbs.db
EOF

# Create placeholder files
touch src/.gitkeep
touch tests/.gitkeep
echo "# Documentation" > docs/README.md

# Initial commit
git add .
git commit -m "Initial project setup"

echo "✅ Project '$PROJECT_NAME' created successfully!"
echo "   cd $PROJECT_NAME && code ."
```

---

## ✅ Phase 1 Completion Checklist

Before moving to Phase 2, ensure you can:

- [ ] Navigate Linux file system without looking up commands
- [ ] Explain Linux permissions (user/group/other, rwx)
- [ ] Write a bash script with variables, conditionals, and loops
- [ ] Use Git for daily development workflow
- [ ] Create and merge feature branches
- [ ] Push to and pull from GitHub
- [ ] Complete the Phase 1 project (automation script)

---

## 🔗 Quick Reference

**Essential Linux Commands:**
```bash
pwd, cd, ls, mkdir, rm, cp, mv, cat, grep, find, chmod, chown
```

**Essential Git Commands:**
```bash
git init, add, commit, push, pull, branch, checkout, merge, status, log
```

**Bash Script Template:**
```bash
#!/bin/bash
set -e  # Exit on error

# Variables
VAR="value"

# Functions
my_function() {
    echo "Hello, $1"
}

# Main logic
my_function "World"
```

---

## ➡️ Next Step

Ready for Phase 2? [Cloud & Infrastructure →](../phase2-infrastructure/README.md)

