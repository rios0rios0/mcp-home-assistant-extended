# GitHub Copilot Instructions for MCP HA Extended

## Project Overview

MCP HA Extended is a Model Context Protocol (MCP) server that extends Home Assistant capabilities to support automation management. It can be deployed as:
1. A standalone Python application
2. A Home Assistant addon (recommended)

The server acts as a bridge between MCP-compatible clients (like Cursor IDE) and Home Assistant's REST API, enabling AI-powered automation management.

## Architecture

```
┌─────────────────┐
│   MCP Client    │  (e.g., Cursor IDE)
│                 │
└────────┬────────┘
         │ MCP Protocol (stdio)
         │
┌────────▼─────────────────────────┐
│  MCP HA Extended Server          │
│  (Python - this project)         │
│  - server.py: MCP tool handlers  │
│  - Uses aiohttp for async HTTP   │
└────────┬─────────────────────────┘
         │ HTTP REST API
         │
┌────────▼────────┐
│ Home Assistant   │
│ REST API        │
│ /api/automation │
└─────────────────┘
```

## Key Technologies & Dependencies

- **Python 3.10+**: Minimum required version
- **MCP SDK** (`mcp>=0.9.0`): Model Context Protocol implementation
- **aiohttp** (`>=3.9.0`): Async HTTP client for Home Assistant API
- **PyYAML** (`>=6.0`): YAML parsing for automation configurations
- **python-dotenv** (`>=1.0.0`): Environment variable management
- **PDM**: Package and dependency manager (NOT pip/poetry)

## Development Tools

### Package Management
- **ALWAYS use PDM** for dependency management
- Add dependencies: `pdm add <package>`
- Install dependencies: `pdm install`
- Update dependencies: `pdm update`
- **NEVER** use `pip install -r requirements.txt` (no requirements.txt exists)

### Testing
- Framework: **pytest** with **pytest-asyncio**
- Run tests: `pdm run pytest` or `pdm run test`
- All async functions must use `@pytest.mark.asyncio` decorator
- Mock external API calls (Home Assistant) using `unittest.mock`

### Code Quality
- **Black**: Code formatter (line length: 100)
- **Ruff**: Linter (targets Python 3.10+)
- Format code: `pdm run black .`
- Lint code: `pdm run ruff check .`

## Coding Standards

### Python Style
- Line length: **100 characters** (configured in pyproject.toml)
- Target: Python 3.10+ (use modern type hints: `dict | None` instead of `Optional[dict]`)
- Use type hints for all function signatures
- Use descriptive variable names
- Follow PEP 8 conventions (enforced by Black and Ruff)

### Async/Await Patterns
- Use `async`/`await` for all I/O operations
- Use `aiohttp.ClientSession()` for HTTP requests (not `requests`)
- Always use async context managers: `async with session.request(...) as response:`
- Handle exceptions appropriately in async functions

### MCP Tool Definitions
Each tool follows this pattern:
```python
Tool(
    name="tool_name",
    description="Clear description of what the tool does",
    inputSchema={
        "type": "object",
        "properties": {
            "param_name": {
                "type": "string",  # or "object", "array", etc.
                "description": "Parameter description"
            }
        },
        "required": ["param_name"]  # List required parameters
    }
)
```

### Home Assistant API Patterns
- Base URL: `{HA_URL}/api`
- All requests require: `Authorization: Bearer {HA_TOKEN}`
- Common endpoints:
  - `GET /api/automation` - List automations
  - `GET /api/automation/{id}` - Get automation
  - `POST /api/automation` - Create automation
  - `PUT /api/automation/{id}` - Update automation
  - `DELETE /api/automation/{id}` - Delete automation
  - `POST /api/automation/{id}/trigger` - Trigger automation

## File Structure

```
.
├── .github/
│   ├── workflows/
│   │   └── build.yaml          # Docker build CI/CD
│   └── copilot-instructions.md # This file
├── .docs/                      # Comprehensive documentation
│   ├── SUMMARY.md              # Documentation index
│   ├── QUICK_START.md          # 5-minute setup guide
│   ├── SETUP.md                # Detailed setup
│   ├── USAGE_EXAMPLES.md       # Code examples
│   └── IMPLEMENTATION_GUIDE.md # Technical details
├── src/
│   └── mcp_ha_extended/
│       ├── __init__.py         # Version info
│       └── server.py           # Main MCP server implementation
├── tests/
│   ├── __init__.py
│   ├── test_server.py          # Server tests (pytest)
│   └── manual_test_api.py      # Manual testing utilities
├── rootfs/                     # Home Assistant addon files
├── pyproject.toml              # PDM config, dependencies, tool settings
├── config.yaml                 # Addon configuration
└── Dockerfile                  # Multi-arch Docker build
```

## Environment Variables

- `HA_URL`: Home Assistant URL (e.g., `http://homeassistant.local:8123`)
- `HA_TOKEN`: Long-lived access token from Home Assistant
- Both are **required** for the server to function

## Testing Guidelines

### Unit Tests
- Mock external dependencies (Home Assistant API calls)
- Use `AsyncMock` for async functions
- Test both success and error paths
- Example pattern:
  ```python
  @pytest.mark.asyncio
  async def test_function_name(self):
      with patch("mcp_ha_extended.server.HA_TOKEN", "test_token"):
          mock_response = AsyncMock()
          # ... setup mock
          result = await function_under_test()
          assert result == expected_value
  ```

### Manual Testing
- Use `tests/manual_test_api.py` for live testing
- Requires actual Home Assistant instance
- Test against real API endpoints

## Docker & Deployment

### Multi-Architecture Support
- Builds for: `aarch64`, `armv7`, `amd64`
- Uses QEMU for cross-compilation
- Base images defined in `build.yaml`

### Home Assistant Addon
- Configuration: `config.yaml`
- Startup scripts: `rootfs/etc/s6-overlay/`
- Published to Docker Hub: `rios0rios0/{arch}-addon-mcp-home-assistant-extended`

## Common Tasks

### Adding a New Tool
1. Add `Tool` definition in `list_tools()` function
2. Add handler in `call_tool()` function
3. Implement the API logic (use `ha_api_call()` helper)
4. Add unit tests in `tests/test_server.py`
5. Update documentation in `.docs/USAGE_EXAMPLES.md`

### Updating Dependencies
1. `pdm add <package>@<version>` or `pdm add <package>`
2. Verify with `pdm install`
3. Run tests: `pdm run pytest`
4. Update `pyproject.toml` if needed (usually automatic)

### Debugging
1. Check environment variables are set
2. Test Home Assistant API manually: `curl -H "Authorization: Bearer $HA_TOKEN" $HA_URL/api/`
3. Run server with debug logging
4. Check MCP client logs for protocol errors

## Documentation Standards

- All major features documented in `.docs/` directory
- Code comments for complex logic only (code should be self-explanatory)
- Docstrings for all public functions
- Update relevant `.docs/*.md` files when adding features
- Keep README.md high-level and beginner-friendly

## Best Practices

1. **Minimal changes**: Make surgical, focused changes
2. **Test-driven**: Write tests before or alongside code changes
3. **Type safety**: Use type hints consistently
4. **Error handling**: Provide clear error messages
5. **Async-first**: Use async/await for all I/O operations
6. **Documentation**: Update docs when adding features
7. **Security**: Never commit tokens or secrets
8. **PDM-only**: Do not introduce pip/poetry workflows

## Security Considerations

- Never hardcode `HA_TOKEN` or `HA_URL`
- Always validate user input before sending to Home Assistant
- Use environment variables for sensitive configuration
- Token should have minimal necessary permissions in Home Assistant
- Sanitize YAML input to prevent injection attacks

## CI/CD

- GitHub Actions builds Docker images on push to main/master
- Multi-architecture builds run in parallel
- Images tagged with branch name, version, and `latest`
- Automated testing should be added before merging breaking changes

## Questions to Ask Before Making Changes

1. Does this require new dependencies? (Use `pdm add`)
2. Does this change the API contract? (Update tools and docs)
3. Are there existing tests to update?
4. Should this be documented in `.docs/`?
5. Does this affect the Home Assistant addon configuration?
6. Is this change backward compatible?

## Preferred Solutions

- **HTTP Client**: aiohttp (not requests)
- **Package Manager**: PDM (not pip or poetry)
- **Testing**: pytest + pytest-asyncio (not unittest)
- **Formatting**: Black (configured in pyproject.toml)
- **Linting**: Ruff (not flake8 or pylint)
- **Type Checking**: Built-in type hints (consider adding mypy if needed)
