# Contributing to ESPHome E-Ink Weather Display

Thank you for your interest in contributing! This document provides guidelines for contributing to this project.

## How to Contribute

### Reporting Bugs

1. Check existing [issues](../../issues) to avoid duplicates
2. Create a new issue with:
   - Clear title and description
   - Steps to reproduce
   - Hardware details (ESP32 board, display model)
   - ESPHome version
   - Configuration (sanitized - remove personal data!)
   - Logs (if applicable)
   - Screenshots (if relevant)

### Suggesting Features

1. Check if feature already exists or is planned
2. Open an issue describing:
   - The feature you want
   - Why it would be useful
   - How it should work
   - Example use cases

### Submitting Code Changes

1. **Fork the repository**
2. **Create a branch** for your feature:
   ```bash
   git checkout -b feature/your-feature-name
   ```
3. **Make your changes**:
   - Follow existing code style
   - Add comments for complex logic
   - Update documentation if needed
4. **Test thoroughly**:
   - Test on actual hardware if possible
   - Verify no regressions
5. **Commit your changes**:
   ```bash
   git add .
   git commit -m "Add: your feature description"
   ```
6. **Push to your fork**:
   ```bash
   git push origin feature/your-feature-name
   ```
7. **Create a Pull Request**:
   - Describe your changes
   - Reference related issues
   - Request review

## Code Style

### YAML Configuration

- Use 2 spaces for indentation (not tabs)
- Add comments for complex sections
- Keep lines under 120 characters
- Group related configurations together

### C++ Lambda Code

- Use meaningful variable names
- Add comments for non-obvious logic
- Follow ESPHome coding conventions
- Keep functions focused and simple

### Documentation

- Update README.md for user-facing changes
- Update INSTALLATION.md for setup changes
- Add comments in YAML for configuration options

## Testing

### Before Submitting

- [ ] Code follows project style
- [ ] Comments explain complex logic
- [ ] Documentation is updated
- [ ] No hardcoded credentials or personal data
- [ ] Tested on hardware (if possible)

### Testing Checklist

For display changes:
- [ ] Display renders correctly
- [ ] No garbage or artifacts
- [ ] All text is readable
- [ ] Icons appear correctly

For deep sleep changes:
- [ ] Device enters deep sleep
- [ ] Device wakes up correctly
- [ ] Battery life is reasonable

For sensor changes:
- [ ] All required sensors are documented
- [ ] Fallback values work correctly
- [ ] No crashes on missing data

## Documentation

### Adding New Features

When adding a feature:
1. Update README.md with description
2. Add to INSTALLATION.md if setup changes needed
3. Add example configuration if applicable
4. Update entity/sensor requirements list

### Adding New Room Support

To add a new room:
1. Add sensor entities in documentation
2. Update display lambda code
3. Adjust layout/positions if needed
4. Document the changes

## Personal Data

**Never commit personal information!**

Remove or sanitize:
- WiFi SSID and passwords
- API keys and tokens
- Personal IP addresses
- Home Assistant entity IDs specific to your setup
- Any other sensitive information

Use `!secret` for credentials or provide generic examples.

## Questions?

If you have questions:
1. Check existing issues and discussions
2. Read the documentation
3. Open a new issue with your question

Thank you for contributing! 🎉
