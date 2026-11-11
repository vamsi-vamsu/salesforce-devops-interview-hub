# Salesforce DevOps Interview Hub

A searchable, responsive interview preparation site. The interview question bank and sample answers are maintained as Markdown files in `interview-answers/`. The website reads those Markdown files directly, so the Markdown documents are the single source of truth.

## Repository guide

The website includes a **Repository Guide** category with step-by-step instructions for editing existing answers, adding questions or categories, and publishing changes. Read `interview-answers/00-repository-guide.md` before making structural changes.

## Website structure

The navigation has two main sections:

- **Interview Q&A:** General Salesforce, technical, security, architecture, testing, data, leadership and behavioral topics.
- **Project Implementation:** Platform-specific implementation and release-management content grouped into subfolders.

### Project Implementation folders

- **Copado:** Release Management, Source Format Pipelines, AI & Release Optimization.
- **Gearset:** Release Management.
- **GitLab CI/CD:** Pipeline design and implementation.
- **GitHub Actions:** Workflow and deployment topics.
- **Azure DevOps:** Reserved for Azure Pipelines and Salesforce deployment Q&A as topics are added.
- The Project Implementation overview is in `interview-answers/project-implementation/README.md`.

The sidebar groups platform content under Project Implementation. Search and preparation tracking continue to work across all categories.

## Edit locally on your Mac

1. Install [Git](https://git-scm.com/downloads) and [Visual Studio Code](https://code.visualstudio.com/).
2. Clone this repository:

   ```bash
   git clone https://github.com/vamsi-vamsu/salesforce-devops-interview-hub.git
   cd salesforce-devops-interview-hub
   code .
   ```

3. Open a file under `interview-answers/` and edit the questions or answers using Markdown.
4. Save, preview Markdown in VS Code with **Cmd+Shift+V**, then commit and push:

   ```bash
   git add interview-answers/
   git commit -m "Update interview preparation answers"
   git push
   ```

GitHub Pages will publish the updated site after the commit is pushed.

## Markdown format

Each category has its own Markdown file. Platform-specific files are stored under `interview-answers/project-implementation/<platform>/`. Use `#` for the category heading and `## Q: ` for each question. Put the sample answer directly below the question. You can use normal Markdown headings, paragraphs, bullet lists, numbered steps and code fences.

Example:

```markdown
# GitLab CI/CD

## Q: Explain your GitLab CI/CD architecture.

We use two long-lived branches: Develop and main.

### Pipeline stages

1. Security
2. Deploy
3. Test
4. Review

## Q: How do you handle snapshot failures?

The pipeline falls back to regular scratch-org creation when the nightly snapshot is unavailable.
```

## Website features

- Search and category filters
- Expand/collapse question cards
- Prepared / Needs practice status tracking
- Personal notes stored in the current browser using localStorage
- Responsive layout

Personal notes and preparation status are browser-local. They are not synced to GitHub. The question bank and sample answers are public because this repository and its GitHub Pages site are public.

## Content structure

All interview questions and sample answers live in `interview-answers/*.md`. The website loads those files at runtime. Do not edit the old `answers.js` file; Markdown is now the source of truth.
