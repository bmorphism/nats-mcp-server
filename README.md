[![MseeP.ai Security Assessment Badge](https://mseep.net/pr/bmorphism-nats-mcp-server-badge.png)](https://mseep.ai/app/bmorphism-nats-mcp-server)

# NATS MCP Server

An MCP (Model Context Protocol) server that provides access to [NATS](https://nats.io/), a cloud native messaging system, through the NATS CLI.

## Features

- Publish messages with advanced options (headers, templates, reply subjects)
- Subscribe to subjects with configurable timeouts and message counts
- Request-reply pattern support with headers
- Full NATS CLI integration
- Error handling and cleanup

## Requirements

- Node.js >= 14.0.0
- NATS CLI (nats)

### Installing NATS CLI

#### macOS

Using Homebrew:
```bash
brew tap nats-io/nats-tools
brew install nats-io/nats-tools/nats
```

#### Linux

Using snap:
```bash
snap install nats
```

Using apt (Debian/Ubuntu):
```bash
# First, add the NATS repository
echo "deb https://dl.nats.io/nats-io/repo/deb nats release" | sudo tee /etc/apt/sources.list.d/nats.list
curl -fsSL https://dl.nats.io/nats-io/repo/deb/gpg.key | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/nats.gpg

# Then install
sudo apt-get update
sudo apt-get install nats-server
```

Using yum (RHEL/CentOS):
```bash
# Add the NATS repository
echo "[nats]
name=NATS Repository
baseurl=https://dl.nats.io/nats-io/repo/rpm/centos/\$releasever/\$basearch
gpgcheck=1
gpgkey=https://dl.nats.io/nats-io/repo/rpm/gpg.key
enabled=1" | sudo tee /etc/yum.repos.d/nats.repo

# Install
sudo yum install nats-server
```

#### Windows

Using Chocolatey:
```powershell
choco install nats-io-client
```

Using Scoop:
```powershell
scoop bucket add nats https://github.com/nats-io/scoop-bucket.git
scoop install nats
```

Manual installation:
1. Download the latest release from [NATS CLI Releases](https://github.com/nats-io/natscli/releases)
2. Extract the archive
3. Add the binary to your system PATH

#### Building from Source

If packages are not available for your system:
```bash
# Requires Go 1.16+
go install github.com/nats-io/natscli/nats@latest
```

### Verifying Installation

After installation, verify the CLI works:
```bash
# Check version
nats --version

# Test connection to default server
nats ping

# List available commands
nats help
```

## Installation

```bash
# Install from npm
npm install @modelcontextprotocol/nats-mcp-server

# Or clone and build from source
git clone https://github.com/bmorphism/nats-mcp-server.git
cd nats-mcp-server
npm install
npm run build
```

## Configuration

The server can be configured using environment variables:

- `NATS_URL`: NATS server URL (default: 'nats://localhost:4222')

## Usage

The server provides the following MCP tools:

### publish

Publish a message to a NATS subject with advanced options.

Parameters:
- `subject` (required): NATS subject to publish to
- `message` (required): Message to publish
- `reply` (optional): Reply subject for request-reply patterns
- `headers` (optional): Array of message headers (key-value pairs)
- `count` (optional): Number of messages to publish
- `sleep` (optional): Sleep duration between messages (e.g., "100ms")
- `template` (optional): Enable Go template processing in message

Example with templates:
```typescript
const result = await mcp.useTool("nats", "publish", {
  subject: "greetings",
  message: "Message {{Count}} @ {{Time}}",
  count: 5,
  sleep: "1s",
  template: true
});
```

Example with headers:
```typescript
const result = await mcp.useTool("nats", "publish", {
  subject: "orders",
  message: "New order received",
  headers: [
    { key: "OrderId", value: "12345" },
    { key: "Priority", value: "high" }
  ]
});
```

### subscribe

Subscribe to a NATS subject and receive messages.

Parameters:
- `subject` (required): NATS subject to subscribe to
- `timeout` (optional): Subscription timeout in milliseconds (default: 5000)
- `count` (optional): Number of messages to receive before exiting
- `raw` (optional): Show only message payload

Example:
```typescript
const result = await mcp.useTool("nats", "subscribe", {
  subject: "greetings",
  timeout: 10000,
  count: 5,
  raw: true
});
```

### request

Send a request message and wait for a reply.

Parameters:
- `subject` (required): NATS subject to send request to
- `message` (required): Request message
- `timeout` (optional): Request timeout in milliseconds (default: 5000)
- `headers` (optional): Array of request headers (key-value pairs)

Example:
```typescript
const result = await mcp.useTool("nats", "request", {
  subject: "service.time",
  message: "What time is it?",
  timeout: 3000,
  headers: [
    { key: "Locale", value: "en-US" }
  ]
});
```

## Template Functions

When using templates (template: true), the following functions are available:

- `Count`: Message number
- `TimeStamp`: RFC3339 format current time
- `Unix`: Seconds since 1970 in UTC
- `UnixNano`: Nano seconds since 1970 in UTC
- `Time`: Current time
- `ID`: Unique ID
- `Random(min, max)`: Random string between min and max length

Example with random strings:
```typescript
const result = await mcp.useTool("nats", "publish", {
  subject: "test",
  message: "Random data: {{ Random 10 100 }}",
  template: true
});
```

## Adding to MCP Configuration

### For Cline (VSCode Extension)

Add to `~/Library/Application Support/Code/User/globalStorage/saoudrizwan.claude-dev/settings/cline_mcp_settings.json`:

```json
{
  "mcpServers": {
    "nats": {
      "command": "node",
      "args": ["/path/to/nats-mcp-server/build/index.js"],
      "env": {
        "NATS_URL": "nats://localhost:4222"
      }
    }
  }
}
```

### For Claude Desktop

Add to `~/Library/Application Support/Claude/claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "nats": {
      "command": "node",
      "args": ["/path/to/nats-mcp-server/build/index.js"],
      "env": {
        "NATS_URL": "nats://localhost:4222"
      }
    }
  }
}
```

## Error Handling

The server includes robust error handling for:
- Connection failures
- Invalid parameters
- Timeouts
- Network errors
- NATS-specific errors
- CLI execution errors

## Development

To build the project:

```bash
npm run build
```

The build output will be in the `build` directory.

## Contributing

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## License

MIT

## Acknowledgments

- [NATS.io](https://nats.io/) - For the amazing messaging system
- [NATS CLI](https://github.com/nats-io/natscli) - For the CLI tool
- [Model Context Protocol](https://github.com/modelcontextprotocol) - For the MCP framework