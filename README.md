# SQLatte Documentation

Official documentation for [SQLatte](https://github.com/osmanuygar/sqlatte) - A Natural Language to SQL Analytics Platform.

## 🚀 View Documentation

Visit: **https://osmanuygar.github.io/sqlatte-docs/**

## 📖 What's Documented

- **Getting Started**: Installation, quick start, Docker deployment
- **Configuration**: Database providers, LLM providers, analytics setup
- **Widgets**: Standard widget, auth widget, embedding guide
- **Features**: Scheduling, analytics, runtime prompts, barista personality
- **API Reference**: REST endpoints, WebSocket API
- **Examples**: Query examples, use cases
- **Architecture**: Design principles, system overview

## 🛠️ Local Development

### Prerequisites
- Python 3.8+
- pip

### Setup

```bash
# Clone this repository
git clone https://github.com/osmanuygar/sqlatte-docs.git
cd sqlatte-docs

# Install dependencies
pip install -r requirements.txt

# Serve locally
mkdocs serve

# Visit http://127.0.0.1:8000
```

### Build

```bash
# Build static site
mkdocs build

# Output in site/ directory
```

## 📝 Contributing

We welcome contributions! Here's how:

1. Fork this repository
2. Create a branch: `git checkout -b docs/new-feature`
3. Make your changes
4. Test locally: `mkdocs serve`
5. Commit: `git commit -m "Add documentation for X"`
6. Push: `git push origin docs/new-feature`
7. Create a Pull Request

### Documentation Style Guide

- Use clear, concise language
- Include code examples
- Add warnings/tips where helpful
- Test all code snippets
- Keep the barista personality ☕

### Manual Deploy

```bash
mkdocs gh-deploy
```

## 📄 License

Same as the main [SQLatte project](https://github.com/osmanuygar/sqlatte).

## 🔗 Links

- **Main Repository**: [github.com/osmanuygar/sqlatte](https://github.com/osmanuygar/sqlatte)
- **Documentation Site**: [osmanuygar.github.io/sqlatte-docs](https://osmanuygar.github.io/sqlatte-docs)
- **Issues**: [Report documentation issues](https://github.com/osmanuygar/sqlatte-docs/issues)

---

Built with [MkDocs](https://www.mkdocs.org/) and [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/)
