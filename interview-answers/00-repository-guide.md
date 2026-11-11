# Repository Guide

## Q: How do I use and maintain this interview preparation website?

This repository contains a public Salesforce DevOps interview preparation website. The interview question bank is maintained in Markdown files under `interview-answers/`. The website reads those files and displays the questions by category.

### 1. Set up the repository on your Mac

Install Git and Visual Studio Code, then clone the repository:

```bash
git clone https://github.com/vamsi-vamsu/salesforce-devops-interview-hub.git
cd salesforce-devops-interview-hub
code .
```

### 2. Update an existing question or answer

1. Open the relevant Markdown file inside `interview-answers/`.
2. Find the question using the `## Q: ` heading.
3. Edit the answer directly below the question.
4. Save the file.
5. Preview Markdown in VS Code with **Cmd+Shift+V**.

You do not need to edit `index.html` when changing the content of an existing category.

### 3. Add a new question to an existing category

Use this format:

```markdown
## Q: How do you manage Salesforce deployments?

Describe the implementation, the decisions made, the tools used and the outcome.

### Key steps

1. Validate the change.
2. Deploy to the target environment.
3. Run tests and post-deployment checks.
```

Keep the question heading format as `## Q: ` so the website can identify it.

### 4. Add a new category

1. Create a new numbered Markdown file in `interview-answers/`, for example `20-new-topic.md`.
2. Start the file with a category heading using `# Category Name`.
3. Add each question using `## Q: Question text`.
4. Open the root `index.html` file.
5. Add the new file's raw GitHub URL to the `MARKDOWN_FILES` array in the script. Follow the existing URL format and place the entry where the category should appear.
6. Save both files.

The category order in the website follows the order of the files in the `MARKDOWN_FILES` array.

### 5. Publish your changes

From the repository folder, run:

```bash
git add interview-answers/ index.html README.md
git commit -m "Update interview preparation content"
git push
```

GitHub Pages publishes the updated website after the changes reach the configured branch. Allow a short time for the deployment to finish, then refresh the website.

### 6. Important repository files

- `index.html`: website layout, styling, search, filters and Markdown loading.
- `interview-answers/*.md`: public interview questions and answers.
- `README.md`: project overview and quick-start instructions.
- `.github/workflows/`: GitHub Actions workflows, if configured.

### 7. Privacy and public content

This is a public repository. Anyone can view or copy the Markdown files and website content. Do not add personal contact details, passwords, tokens, customer names, confidential project information or private company data. Preparation status and personal notes entered in the website are stored in the current browser and are not uploaded to GitHub.

### 8. Writing standards

- Use first-person language only when describing a verified implementation.
- Separate hands-on experience from conceptual understanding.
- Do not invent org counts, pipeline counts, release metrics or project outcomes.
- Explain the purpose of each pipeline, the trigger, target environment, validation steps and deployment outcome.
- Keep answers direct and suitable for speaking during an interview.
