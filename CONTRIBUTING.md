# Contributing to STF Docker Compose Template

Thank you for your interest in contributing! This template aims to provide a production-ready starting point for deploying STF with Docker Compose.

## How to Contribute

### Reporting Issues

If you encounter problems with this template:

1. Check existing issues to avoid duplicates
2. Provide clear description of the problem
3. Include relevant configuration (sanitized, no secrets!)
4. Share error logs if applicable
5. Specify your environment (OS, Docker version, etc.)

### Suggesting Enhancements

We welcome suggestions for improvements:

1. Open an issue describing the enhancement
2. Explain the use case and benefits
3. Provide examples if possible

### Pull Requests

1. **Fork** the repository
2. **Create a branch** from `master`:
   ```bash
   git checkout -b feature/your-feature-name
   ```
3. **Make your changes**:
   - Follow existing code style
   - Update documentation if needed
   - Test your changes thoroughly
4. **Commit** with clear messages:
   ```bash
   git commit -m "Add feature: description"
   ```
5. **Push** to your fork:
   ```bash
   git push origin feature/your-feature-name
   ```
6. **Open a Pull Request** with:
   - Clear title and description
   - Reference any related issues
   - Explain what changed and why

## Guidelines

### Configuration Files

- Keep .env.example files generic with placeholder values
- Never commit actual secrets or credentials
- Add comments explaining configuration options
- Test with example values to ensure they work

### Documentation

- Update README.md if adding new features
- Keep language clear and concise
- Include examples where helpful
- Link to official STF docs for detailed topics

### Docker Compose Files

- Follow Docker Compose best practices
- Use environment variables for configuration
- Comment complex or non-obvious settings
- Test changes with `docker-compose config` to validate syntax

### Testing

Before submitting:

1. Test central server deployment from scratch
2. Test remote provider deployment
3. Verify all documented features work
4. Check for any hardcoded values that should be configurable

## Code of Conduct

- Be respectful and constructive
- Focus on the technical merits
- Help others learn and improve
- Keep discussions on-topic

## Questions?

If you have questions about contributing:

- Open an issue with the "question" label
- Check existing discussions
- Reference official STF documentation for STF-specific questions

## License

By contributing, you agree that your contributions will be licensed under the same license as this project.

Thank you for helping improve this template!
