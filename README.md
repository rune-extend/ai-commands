# 🤖 AI Commands for Development

This repository contains a collection of **AI commands and prompts** designed to improve productivity and code quality during software development.

## 🎯 Purpose

This repository centralizes **structured prompts** intended for AI assistants (like GitHub Copilot, ChatGPT, Claude, etc.) to automate and standardize common development tasks. Each `.md` file contains detailed instructions enabling AI to generate high-quality code that follows best practices.

## 📚 Available Commands

### 🔨 [Git Commit Message Generator](commit.md)
Automatically generates commit messages following [Conventional Commits](https://www.conventionalcommits.org/), creates changelogs (changesets), and updates README.md documentation when needed.

**Usage**: Analyze staged files and generate a professional commit message with automatic version management.

### 📖 [JSDoc Generator](jsdoc.md)
Generates complete and detailed JSDoc documentation for your TypeScript/JavaScript files, including interfaces, functions, React components, hooks, and constants.

**Usage**: Automatically document your code with examples, descriptions, and comprehensive annotations for better IDE experience.

### 📝 [Script Logging Best Practices](script-logging.md)
Creates a structured and hierarchical logging system for your Node.js scripts, with debug mode support, visual grouping, progress indicators, and conditional coloring.

**Usage**: Implement professional logging with visual groups, debug/normal separation, and output examples for different scenarios.

### ✅ [Unit Test Generator](unit-test.md)
Generates comprehensive unit tests with Jest for React (hooks) and NestJS (services), following best practices and codebase conventions.

**Usage**: Create exhaustive test suites covering all use cases, with proper timer, event, async/await handling, and cleanup.

## 🚀 How to Use

1. **Copy the content** of the `.md` file corresponding to your need
2. **Paste it into your AI assistant** (ChatGPT, Claude, Copilot, etc.)
3. **Add the context** of your code (source file, feature to test, etc.)
4. **The AI generates** the code following the prompt instructions

## 📋 Repository Structure

```
.
├── README.md              # This file
├── commit.md              # Commit message generation
├── jsdoc.md               # JSDoc documentation generation
├── script-logging.md      # Logging best practices
└── unit-test.md           # Unit test generation
```

## 🎨 Features

- ✅ **Detailed prompts**: Complete and structured instructions
- ✅ **Best practices**: Follows industry conventions and standards
- ✅ **Examples included**: Patterns and code examples provided
- ✅ **Versatile**: Compatible with different AI assistants
- ✅ **Extensible**: Easily extendable with new commands

## 🤝 Contributing

Contributions are welcome! Feel free to:
- Propose new AI commands
- Improve existing prompts
- Add examples or use cases
- Fix errors or clarify instructions

## 📝 License

This repository is provided as-is for personal or professional use. Feel free to adapt it to your specific needs.

---

💡 **Tip**: These prompts work best with powerful AI assistants like GPT-4, Claude 3, or GitHub Copilot Chat. The more detailed context you provide, the better the results will be.
