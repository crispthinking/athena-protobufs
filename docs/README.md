# Athena Protobufs Documentation

This directory contains comprehensive Sphinx documentation for the Athena API protobuf schema. The documentation provides detailed API references, usage examples, and deployment guidance for the Athena CSAM detection service.

## Documentation Structure

```
docs/
├── index.rst                 # Main documentation index
├── overview.rst              # Architecture and concepts overview
├── api_reference.rst         # Detailed API documentation
├── examples.rst              # Code examples in multiple languages
├── deployment_guide.rst      # Production deployment guide
├── conf.py                   # Sphinx configuration
├── requirements.txt          # Python dependencies
├── Makefile                  # Build commands
├── dev-server.py            # Development server script
├── _static/
│   └── custom.css           # Custom styling
└── _build/                  # Generated documentation (created during build)
```

## Quick Start

### Prerequisites

- Python 3.8 or higher
- uv (modern Python package manager)

### Installation

1. Install uv if not already installed:
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
# or
pip install uv
```

2. Sync dependencies:
```bash
uv sync
```

### Building Documentation

Build HTML documentation:
```bash
make html
```

The generated documentation will be available in `_build/html/index.html`.

### Development Server

For development with live reloading:
```bash
# Using the development script
python dev-server.py

# Or using sphinx-autobuild directly
make livehtml
```

This will start a development server at `http://localhost:8000` that automatically rebuilds when files change.

## Available Make Commands

- `make html` - Build HTML documentation
- `make clean` - Clean build directory
- `make linkcheck` - Check for broken links
- `make spelling` - Run spell checking (requires sphinxcontrib-spelling)
- `make latexpdf` - Generate PDF documentation (requires LaTeX)
- `make epub` - Generate EPUB documentation
- `make install` - Install documentation dependencies
- `make dev-setup` - Set up development environment

## Additional Commands

For advanced usage, you can run Sphinx commands directly:

```bash
# Live reload development server
make livehtml

# Build with specific options
uv run sphinx-build -b html . _build/html

# Check spelling (if enabled)
make spelling

# Build PDF (requires LaTeX)
make latexpdf

# Build EPUB
make epub
```

## Documentation Sections

### 1. Overview (`overview.rst`)
- Architecture and design principles
- Core concepts (deployments, affiliates, correlation)
- Data flow and processing pipeline
- Performance considerations
- Security guidelines

### 2. API Reference (`api_reference.rst`)
- Complete protobuf schema documentation
- Service definitions and RPC methods
- Message types and field descriptions
- Enumeration values and error codes
- Language-specific options

### 3. Examples (`examples.rst`)
- Usage patterns and workflow guidance
- Best practices for implementation
- Error handling strategies
- Performance optimization guidance
- Development and testing considerations

### 4. Deployment Guide (`deployment_guide.rst`)
- Client code generation
- Production configuration
- Security best practices
- Monitoring and observability
- Load balancing strategies
- Troubleshooting guide

## Customization

### Styling

Custom CSS is located in `_static/custom.css` and includes:
- Protocol buffer specific styling
- Code block enhancements
- Responsive design improvements
- API reference formatting
- Error code highlighting

### Configuration

Sphinx configuration is in `conf.py` and includes:
- Extensions for MyST parser, autodoc, intersphinx
- Read the Docs theme configuration
- Cross-reference settings
- HTML output options

## CI/CD Integration

The documentation includes GitHub Actions workflow (`.github/workflows/docs.yml`) that:
- Builds documentation on push/PR
- Checks for broken links
- Validates protobuf syntax
- Deploys to GitHub Pages (on main branch)
- Runs documentation tests

## Contributing

When updating documentation:

1. **Content Changes**: Edit the `.rst` files directly
2. **API Changes**: Update `api_reference.rst` when protobuf schema changes
3. **Examples**: Add new examples to `examples.rst` with multiple language support
4. **Styling**: Modify `_static/custom.css` for visual changes

### Writing Guidelines

- Use clear, concise language
- Include code examples for complex concepts
- Cross-reference related sections using Sphinx directives
- Follow reStructuredText formatting conventions
- Provide clear guidance and best practices

### Building Process

1. Make your changes
2. Build locally: `make html`
3. Check for warnings in build output
4. Test links: `make linkcheck`
5. Review generated HTML
6. Commit changes

## Dependencies

Core dependencies include:
- **Sphinx**: Documentation generation framework
- **sphinx-rtd-theme**: Read the Docs theme
- **myst-parser**: Markdown support in Sphinx
- **sphinx-autobuild**: Live reloading development server
- **grpcio/grpcio-tools**: Protocol buffer support

See `pyproject.toml` for complete dependency list with version constraints. Dependencies are managed with uv for faster and more reliable installation.

## Troubleshooting

### Common Issues

1. **Build Failures**: Check that all dependencies are installed with `uv sync`
2. **Missing References**: Ensure all cross-references use correct targets
3. **Broken Links**: Run `make linkcheck` to identify issues
4. **Styling Issues**: Check `_static/custom.css` and browser developer tools
5. **Dependency Issues**: Try `uv sync --reinstall` to refresh all packages

### Getting Help

- Check Sphinx documentation: https://www.sphinx-doc.org/
- Review existing documentation structure
- Use the development server for immediate feedback
- Test builds locally before committing

## License

This documentation is part of the Athena Protobufs project and follows the same licensing terms as the main project.
