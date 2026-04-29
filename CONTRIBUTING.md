# CONTRIBUTING.md

> Welcome! Thank you for your interest in improving `TechInterviewMastery`. Your contributions help ensure that developers have access to high-quality, accurate interview preparation material.

## 🤝 Why Contribute?

This repository is a community-driven resource. Whether you are an expert in **Generative AI**, **.NET internals**, or **System Architecture**, your insights can help thousands of candidates prepare for their next role. We aim to maintain accuracy and clarity across all 520+ questions.

## 📝 Prerequisites

Before contributing, please ensure:

- You have a GitHub account.
- You are familiar with Markdown (`README.md` style).
- Your code snippets are syntactically correct (e.g., `C#`, `Java`, `Python`).
- You understand the MIT License terms.

## 🚀 How to Contribute

1. **Fork** this repository.
2. **Clone** your forked repo:

    ```bash
    git clone https://github.com/EduardoNicacio/TechInterviewMastery.git
    cd TechInterviewMastery
    ```

3. **Create a Branch**: Use descriptive naming (e.g., `feature/add-python-gil-qa` or `docs/fix-java-concurrency`).
4. **Make Changes**: Add, edit, or fix questions following the guidelines below.
5. **Commit** with clear messages.
6. **Push** to your branch:

    ```bash
    git push origin feature/add-python-gil-qa
    ```

7. **Open a Pull Request (PR)** linking to an issue or describing the change.

## 📋 Content Guidelines

### Markdown Formatting

- Use standard Markdown headers (`#`, `##`, `###`).
- Questions should be numbered sequentially within each file (e.g., `1. Question Title`).
- Answers must be clear, concise, and technically accurate. Avoid overly verbose explanations unless necessary for clarity.
- **Code Snippets**: Always use fenced code blocks with the appropriate language identifier:

  ```csharp
  public class Example { }
  ```

### Quality Standards

- **Accuracy**: Verify all technical facts (e.g., .NET memory management, SQL query performance). Do not hallucinate answers.
- **Relevance**: Ensure questions align with senior-level expectations. Avoid basic syntax questions unless they are foundational for advanced topics.
- **No Duplicates**: Check existing files before adding similar content. If a topic is missing but related to an existing file, consider appending it there instead of creating a new file.

### File Structure

Add your Q&A pairs to the appropriate technology folder (e.g., `qna/.NET_CSharp/`).

- **File Naming**: Use descriptive names like `01_Architecture_QA.md` or `02_Java_Concurrency.md`.
- **Ordering**: Maintain alphabetical or numerical order within files.

## 📝 Commit Message Convention

We use [Conventional Commits](https://www.conventionalcommits.org/) to keep our history clean:

```text
<type>(<scope>): <description>

[optional body]

[optional footer(s)]
```

**Examples:**

- `feat(qna/.NET_CSharp): add async/await memory management questions`
- `fix(qna/Architecture): correct Vertical Slice definition in Q42`
- `docs: update README with new contribution badges`

## 📝 Pull Request Template

When submitting your PR, please include the following information:

### Summary

Briefly describe what you are adding or fixing.

### Changes Made

List specific files modified and the nature of changes (e.g., "Added 5 C# Memory Management questions").

### Testing/Verification

- Did you verify the code snippets compile/run?
- Are the answers verified against official documentation?

### Screenshots (Optional)

If applicable, include diagrams or screenshots.

## 🐛 Reporting Issues

If you find a bug in an existing question or have suggestions for new topics:

1. Go to the **Issues** tab.
2. Create a new issue with a clear title and description.
3. Include relevant code snippets if possible.

## ⚖️ Code of Conduct

- Be respectful and inclusive.
- Do not share personal contact information or private data in PRs/Comments.
- Respect the MIT License terms (you may retain copyright on your specific contributions).

## 📬 Contact

For questions about contributing, reach out to:

- **Email**: [enicacio@gmail.com](mailto:enicacio@gmail.com)
