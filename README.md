# Home Assistant Automation MCP Server

This is a custom MCP (Model Context Protocol) server that extends Home Assistant capabilities to support creating, editing, and listing automations.

## Overview

The existing Home Assistant MCP server provides device control but doesn't support automation management. This server wraps Home Assistant's REST API to provide automation CRUD operations.

## Architecture

```
┌─────────────────┐
│   Cursor/IDE    │
│   (MCP Client)  │
└────────┬────────┘
         │ MCP Protocol
         │
┌────────▼─────────────────────────┐
│  Custom HA Automation MCP Server  │
│  (Python - this project)          │
└────────┬─────────────────────────┘
         │ HTTP REST API
         │
┌────────▼────────┐
│ Home Assistant   │
│ REST API        │
│ /api/automation │
└─────────────────┘
```

## Home Assistant REST API Endpoints

Home Assistant provides these automation endpoints:

- `GET /api/automation` - List all automations
- `GET /api/automation/{automation_id}` - Get specific automation
- `POST /api/automation` - Create new automation
- `PUT /api/automation/{automation_id}` - Update automation
- `DELETE /api/automation/{automation_id}` - Delete automation
- `POST /api/automation/{automation_id}/trigger` - Trigger automation manually

## Implementation Options

### Option 1: Extend Existing MCP Server (Recommended)
If you have access to the Home Assistant MCP server source code, add new tools directly.

### Option 2: Create Standalone MCP Server (This Implementation)
Create a separate MCP server that only handles automations, running alongside the existing HA MCP server.

### Option 3: Fork and Extend
Fork the existing Home Assistant MCP server and add automation tools.

## Prerequisites

- Python 3.10+
- Home Assistant instance with REST API enabled
- Long-lived access token from Home Assistant
- MCP SDK (for Python)

## Setup Instructions

1. Install dependencies:
```bash
pip install mcp aiohttp python-dotenv
```

2. Configure environment variables:
```bash
export HA_URL="http://homeassistant.local:8123"
export HA_TOKEN="your_long_lived_access_token"
```

3. Run the MCP server:
```bash
python mcp_ha_automations/server.py
```

4. Configure Cursor to use this MCP server (add to Cursor settings)

## MCP Tools Provided

1. **list_automations** - List all automations
2. **get_automation** - Get details of a specific automation
3. **create_automation** - Create a new automation from YAML
4. **update_automation** - Update an existing automation
5. **delete_automation** - Delete an automation
6. **trigger_automation** - Manually trigger an automation
7. **enable_automation** - Enable an automation
8. **disable_automation** - Disable an automation
