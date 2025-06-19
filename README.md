# testRepo

Thanks,
Dheeraj Sharma

Begin forwarded message:

From: Dheeraj Sharma <dheerajkumar1994@gmail.com>
Date: 19 June 2025 at 15:34:22 IST
To: Sharma.Dheeraj@pincipal.com
Subject: Biome Research

﻿
name: reusable-lint-format

on:
 workflow_call:
   inputs:
     biome-config:
       default: './config/biome.json'
     flake8-config:
       default: './config/flake8.cfg'
     yamllint-config:
       default: './config/yamllint.yaml'

permissions:
 contents: write

jobs:
 lint-format:
   runs-on: ubuntu-latest

   steps:
     - uses: actions/checkout@v3

     - name: Set up Git
       run: |
         git config --global user.email "github-actions@github.com"
         git config --global user.name "GitHub Actions"

     - uses: actions/setup-node@v4
       with:
         node-version: 20

     - uses: actions/setup-python@v5
       with:
         python-version: '3.11'

     - uses: actions/setup-java@v4
       with:
         distribution: 'temurin'
         java-version: '17'

     - name: Detect Framework (Log Only)
       run: |
         echo "Detecting project type..."
         [[ -f next.config.js ]] && echo "🟨 Next.js detected"
         grep -q "@nestjs/" package.json 2>/dev/null && echo "🟪 NestJS detected"
         grep -q "react" package.json 2>/dev/null && echo "🟦 React detected"
         [[ -d cdk.out ]] && echo "🟧 CDK detected"
         [[ -f pom.xml ]] && echo "🟥 Spring Boot detected"

     - name: Install Biome
       run: npm install --save-dev biome

     - name: Format and Lint with Biome
       run: |
         npx biome format "**/*.{js,jsx,ts,tsx,json}" --write --config-path ${{ inputs.biome-config }}
         npx biome check "**/*.{js,jsx,ts,tsx,json}" --config-path ${{ inputs.biome-config }}

     - name: Install ESLint, markdownlint, htmlhint
       run: npm install --save-dev eslint markdownlint-cli htmlhint

     - name: Run ESLint
       run: |
         npx eslint "**/*.{js,jsx,ts,tsx}" --ext .js,.jsx,.ts,.tsx --fix || true

     - name: Lint Markdown
       run: npx markdownlint "**/*.md" || true

     - name: Lint HTML
       run: npx htmlhint "**/*.html" || true

     - name: Install yamllint
       run: sudo apt-get update && sudo apt-get install -y yamllint

     - name: Lint YAML
       run: yamllint . -c ${{ inputs.yamllint-config }} || true

     - name: Install Python tools
       run: pip install black flake8

     - name: Format Python
       run: black . --include '\.py$'

     - name: Lint Python
       run: flake8 . --config=${{ inputs.flake8-config }}

     - name: Format Java
       run: |
         curl -LJO https://github.com/google/google-java-format/releases/download/v1.17.0/google-java-format-1.17.0-all-deps.jar
         find . -name "*.java" > java_files.txt
         java -jar google-java-format-1.17.0-all-deps.jar --replace @java_files.txt

     - name: Lint Java
       run: |
         curl -LJO https://github.com/checkstyle/checkstyle/releases/download/checkstyle-10.12.4/checkstyle-10.12.4-all.jar
         wget https://raw.githubusercontent.com/checkstyle/checkstyle/master/src/main/resources/google_checks.xml
         java -jar checkstyle-10.12.4-all.jar -c google_checks.xml $(find . -name "*.java")

     - name: Install GitHub CLI
       run: sudo apt-get install -y gh

     - name: Create Pull Request with Fixes
       if: success()
       env:
         GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
       run: |
         git add .
         if git diff --cached --quiet; then
           echo "No changes to commit"
           exit 0
         fi

         BRANCH_NAME="lint-fixes/${GITHUB_SHA::7}"
         git checkout -b $BRANCH_NAME
         git commit -m "chore: lint & format fixes from reusable workflow"
         git push origin $BRANCH_NAME

         gh pr create \
           --base "${{ github.head_ref }}" \
           --head "$BRANCH_NAME" \
           --title "chore: auto-format fixes from CI" \
           --body "This PR contains formatting and linting fixes detected by the reusable lint workflow."



{
 "$schema": "https://biomejs.dev/schemas/1.0.0/schema.json",
 "formatter": { "enabled": true },
 "linter": {
   "enabled": true,
   "rules": {
     "recommended": true,
     "react": {
       "no-unused-props": "warn",
       "jsx-no-useless-fragment": "warn"
     }
   }
 },
 "files": {
   "ignore": ["node_modules/**", ".next/**", "dist/**", "cdk.out/**", "build/**"]
 }
}


[flake8]
max-line-length = 100
exclude = .git,__pycache__,cdk.out,.venv

extends: default

rules:
 line-length:
   max: 120
   level: warning
 document-start:
   present: false
 comments:
   require-starting-space: true


{
 "env": {
   "browser": true,
   "es2021": true
 },
 "extends": ["eslint:recommended", "plugin:react/recommended"],
 "parserOptions": {
   "ecmaVersion": "latest",
   "sourceType": "module"
 },
 "plugins": ["react"],
 "rules": {
   "no-unused-vars": "warn",
   "react/prop-types": "off"
 }
}

{
 "tagname-lowercase": true,
 "attr-lowercase": true,
 "doctype-first": true,
 "id-unique": true,
 "style-disabled": true,
 "attr-value-double-quotes": true
}

default: true
MD013: false   # Disable line-length rule



# 🧠 Engineering Initiative: Unified Linting & Formatting Pipeline Using Biome and Friends

## 🧭 Objective

To enhance **developer productivity**, ensure **consistent code quality**, and reduce **manual review overhead**, we’ve implemented a **centralized, reusable GitHub Actions pipeline** that automatically formats and lints code across our repositories.

This pipeline leverages [Biome](https://biomejs.dev/) for JavaScript/TypeScript/JSON, and integrates best-in-class tools for Python, Java, YAML, HTML, and Markdown.

---

## 🎯 Why Biome?

### ⚡ What is Biome?

Biome is a blazing-fast, modern toolchain for formatting and linting frontend code (JS, TS, JSON, JSX, TSX). It’s built in Rust, making it **significantly faster than ESLint + Prettier**, with **zero dependencies**, **zero config by default**, and a unified interface.

### ✅ Benefits of Biome Adoption

| Area | Impact |
|------|--------|
| 🏃 Performance | Biome is ~20x faster than ESLint/Prettier, reducing CI time. |
| 📦 Simplicity | No extra dependencies or toolchain to maintain in each project. |
| 🔁 Unified tooling | Combines formatting and linting in one tool. |
| 🧠 Smarter rules | Modern ruleset with excellent JSX/TS/JSON support. |
| 🧪 Scalability | Works for React, Next.js, CDK, NestJS, and other frontend stacks. |

---

## 🔧 Tooling Strategy by Language

| Language / Stack | Formatter | Linter | Tool |
|------------------|-----------|--------|------|
| JS/TS/JSON       | ✅        | ✅     | Biome + ESLint fallback |
| React/Next.js    | ✅        | ✅     | Biome + ESLint |
| Python           | ✅        | ✅     | Black + Flake8 |
| Java             | ✅        | ✅     | Google Java Format + Checkstyle |
| YAML             | ❌        | ✅     | yamllint |
| Markdown         | ❌        | ✅     | markdownlint |
| HTML             | ❌        | ✅     | htmlhint |

---

## 🚀 Pipeline Design

### 🔁 Reusable GitHub Action Workflow

- Centralized workflow stored in `emu/github-pipelines`
- Called from all repositories via:

```yaml
jobs:
 lint:
   uses: emu/github-pipelines/.github/workflows/reusable-lint-format.yml@main
```

### 🧩 Features

- 🔎 Auto-detects frameworks like React, Next.js, CDK, Spring Boot
- 🧼 Auto-formats code with best-practice defaults
- 🛑 Fails the CI if critical linting rules are violated
- 🛠️ Automatically creates a **Pull Request** with fixes (instead of direct commits)
- 🎛️ Supports **override configs per repo** (biome.json, .flake8, .yamllint.yaml)

---

## 🗃️ Config Structure

All default rules are centrally stored:

```
github-pipelines/
├── .github/workflows/reusable-lint-format.yml
└── config/
   ├── biome.json
   ├── flake8.cfg
   ├── yamllint.yaml
   ├── .eslintrc.json
   ├── .htmlhintrc
   └── .markdownlint.yaml
```

Teams can override any config by passing custom paths as inputs.

---

## 🛡️ Engineering Outcomes

| Goal | Result |
|------|--------|
| 🕒 Reduce manual PR feedback | Auto-formatting removes style nitpicks from review |
| 🔁 Improve CI/CD repeatability | One consistent pipeline across all repos |
| 🌱 Better onboarding | New projects adopt company standards by default |
| 🧰 Lower toolchain maintenance | No per-project lint/setup duplication |
| 🔍 Better visibility | All formatting changes are PR’d back, no silent overwrites |

---

## 🔄 Maintenance & Upgrade Plan

- Reusable workflow will be **version tagged** (`v1`, `v2`, etc.) for safe rollout
- All default tools (Biome, Flake8, etc.) are **version-pinned**
- Future enhancement plans include:
 - 🧪 Adding CDK synth/validation steps
 - 📖 Linting documentation/code comments
 - ⚙️ GitHub App/CLI automation for onboarding new repos

---

## 📦 Rollout Plan

1. ✅ Centralize and publish `github-pipelines` reusable workflow
2. ✅ Enable this in high-priority apps (React, CDK, Spring Boot)
3. 🛠️ Provide example `.github/workflows/lint.yml` for teams
4. 📣 Train team leads on override mechanisms (if needed)
5. 🔁 Set up GitHub CLI script to apply this across all supported repos

---

## 📢 How to Use in Your Repo

```yaml
jobs:
 lint:
   uses: emu/github-pipelines/.github/workflows/reusable-lint-format.yml@main
   with:
     biome-config: .github/biome.json            # Optional override
     flake8-config: .github/.flake8              # Optional override
     yamllint-config: .github/.yamllint.yaml     # Optional override
```

---

## ✅ Conclusion

This initiative helps standardize and streamline our CI hygiene, making our codebase more maintainable, reviewable, and scalable. With tools like Biome, we’re adopting the **next-gen JS toolchain** — while also supporting our polyglot (Python, Java, YAML, Markdown) ecosystem.

🧠 Less friction.  
🤖 More automation.  
🚀 Better code, faster.




Thanks,
