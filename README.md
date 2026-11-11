# Salesforce DevOps Interview Hub

A searchable, responsive interview preparation site. The interview question bank and sample answers are maintained as Markdown files in `interview-answers/`. The website reads those Markdown files directly, so the Markdown documents are the single source of truth.

## Repository guide

The website includes a **Repository Guide** category with step-by-step instructions for editing existing answers, adding questions or categories, and publishing changes. Read `interview-answers/00-repository-guide.md` before making structural changes.

## Project implementation

The final category, **Project Implementation**, documents end-to-end implementation stories for Copado, GitLab CI/CD, Gearset, GitHub Actions and Azure DevOps. It distinguishes verified hands-on experience from topics that still need project-specific details.

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

Each category has its own file. Use `#` for the category heading and `## Q: ` for each question. Put the sample answer directly below the question. You can use normal Markdown headings, paragraphs, bullet lists, numbered steps and code fences.

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
