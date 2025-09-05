# Onkyo AV Remote

A Fluid-based implementation of the Onkyo eISCP (Integra Serial Control Protocol) for controlling Onkyo network-enabled audio receivers. This project replicates the functionality of the Python [onkyo-eiscp](https://github.com/miracle2k/onkyo-eiscp) project using the Parasol framework.

Can be used as a command-line tool, Fluid library, or via a web interface.

### Command Line Usage

```bash
# Discover receivers on the network
parasol.exe onkyo.fluid discover

# Test connectivity to specific receiver
parasol.exe onkyo.fluid host=192.168.1.40 ping

# Control power
parasol.exe onkyo.fluid power=on
parasol.exe onkyo.fluid power=off

# Get device information
parasol.exe onkyo.fluid info

# Use custom settings
parasol.exe onkyo.fluid host=IP port=PORT timeout=SECONDS command
```

### Command Reference

| Command | Arguments | Description |
|---------|-----------|-------------|
| `discover` | - | Scan network for Onkyo receivers |
| `ping` | - | Test connectivity to receiver |
| `power` | `on|off` | Control receiver power |
| `info` | - | Get device information |

### Options

| Option | Description |
|--------|-------------|
| `host=IP` | Target IP address (default: 192.168.1.40) |
| `port=PORT` | Target port (default: 60128) |
| `timeout=SECONDS` | Connection timeout (default: 5) |
| `help` | Show help message |

## Architecture

### Project Structure

```
projects/onkyo/
├── onkyo.fluid           # Main CLI interface
├── lib/
│   ├── eiscp.fluid       # Core eISCP protocol implementation
│   └── discovery.fluid   # Network discovery module
└── README.md             # This documentation
```

### Module Overview

#### `lib/eiscp.fluid` - Core Protocol
- **eISCP Packet Handling**: 16-byte headers, message encoding/decoding
- **Connection Management**: TCP socket management with timeouts
- **Command Interface**: High-level functions for common operations
- **Error Handling**: Network and protocol error management

Key functions:
- `connect(host, port, timeout)` - Establish connection
- `sendCommand(command, timeout)` - Send eISCP command
- `ping()`, `powerOn()`, `powerOff()`, `getDeviceInfo()` - Convenience functions

#### `lib/discovery.fluid` - Network Discovery  
- **UDP Broadcast**: Sends discovery packets to find receivers
- **Response Parsing**: Extracts device information from responses
- **Multi-receiver Support**: Discovers multiple devices on network

Key functions:
- `findReceivers(timeout)` - Discover all receivers
- `findReceiver(targetIP, timeout)` - Find specific receiver
- `listReceivers(timeout)` - Display discovered receivers

#### `onkyo.fluid` - CLI Interface
- **Argument Parsing**: Command-line option processing
- **Command Routing**: Routes commands to appropriate functions  
- **User Interface**: Clean output formatting and error messages

## Protocol Details

### eISCP Packet Structure

```
Offset | Size | Description
-------|------|------------
0      | 4    | Magic: 'ISCP'
4      | 4    | Header size (16, big-endian)
8      | 4    | Data size (big-endian)
12     | 1    | Version (0x01)
13     | 3    | Reserved (0x00, 0x00, 0x00)
16     | N    | ISCP message data
```

### ISCP Message Format

Commands are formatted as: `!1{command}\r`

Examples:
- `!1PWRQSTN\r` - Power status query
- `!1PWR01\r` - Power on
- `!1PWR00\r` - Power off
- `!1NRIIQSTN\r` - Device information query

### Discovery Protocol

Discovery uses UDP broadcast on port 60128 with commands:
- `!xECNQSTN` - Extended device information
- `!pECNQSTN` - Power status query

Response format: `!1ECNTX-NR609/60128/DX/001122334455`
- Model: TX-NR609
- Port: 60128  
- Area: DX
- MAC: 001122334455

## Future Phases

### Phase 2: Complete Command Library
- Full eISCP command set implementation
- Zone-specific controls (Main, Zone2, Zone3, etc.)
- Input source selection and management
- Volume and audio controls
- Command validation and help system

### Phase 3: HTTP Web Interface  
- Web-based control panel using `httpserver.fluid`
- RESTful API for external integration
- Real-time status updates
- Mobile-responsive interface

### Phase 4: Advanced Features
- Multiple receiver management
- Macro/scene support  
- State caching and synchronization
- Configuration file support
- Logging and usage analytics
