# Contributing Guidelines

Thank you for considering contributing to this project! Here are some guidelines to help you get started.

## How to Contribute

### Reporting Bugs

1. Check if the issue already exists in the Issues section
2. If not, create a new issue with:
   - Clear description of the bug
   - Steps to reproduce
   - Expected vs actual behavior
   - Environment details (OS, versions, etc.)

### Suggesting Features

1. Open an issue with the `enhancement` label
2. Describe the feature and its use case
3. Explain why it would be beneficial

### Submitting Changes

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/your-feature-name`
3. Make your changes
4. Test your changes thoroughly
5. Commit with clear messages: `git commit -m "Add feature: description"`
6. Push to your fork: `git push origin feature/your-feature-name`
7. Open a Pull Request

## Code Standards

### Ansible Playbooks

- Use meaningful task names
- Add comments for complex logic
- Follow YAML best practices (2-space indentation)
- Use variables for configurable values
- Include `changed_when` for command/shell tasks

### Docker

- Use multi-stage builds when applicable
- Minimize image layers
- Don't include secrets in images
- Add healthchecks where possible

### Documentation

- Update README.md if adding new features
- Document all environment variables
- Include examples for new functionality

## Commit Messages

Follow conventional commits format:

```
type(scope): description

[optional body]

[optional footer]
```

Types:
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `refactor`: Code refactoring
- `test`: Adding tests
- `chore`: Maintenance tasks

## Security

- Never commit secrets, passwords, or API keys
- Use environment variables or vault for sensitive data
- Report security vulnerabilities privately

## Questions?

Feel free to open an issue for any questions or clarifications.
