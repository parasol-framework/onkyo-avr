# Onkyo AV Remote

A Fluid-based implementation of the Onkyo eISCP (Integra Serial Control Protocol) for controlling Onkyo network-enabled audio receivers. This project replicates the functionality of the Python [onkyo-eiscp](https://github.com/miracle2k/onkyo-eiscp) project using the Parasol framework.

Can be used as a command-line tool or Fluid library.

## Command Line Usage

### Basic Examples

```bash
# Discover receivers on the network
parasol onkyo.fluid discover

# Get device information and available commands
parasol onkyo.fluid info

# Show verbose device information with all command details
parasol onkyo.fluid info=verbose

# Control power using friendly commands
parasol onkyo.fluid power=on
parasol onkyo.fluid power=off

# Send raw ISCP commands
parasol onkyo.fluid command=PWR01        # Power on
parasol onkyo.fluid command=MVL30        # Set volume to 0x30 (48)

# Send multiple commands (comma-separated)
parasol onkyo.fluid command=PWR01,MVL50  # Power on and set volume

# Use custom settings
parasol onkyo.fluid host=192.168.178.50 port=60128 timeout=10 command=PWRQSTN

# Get help for specific command
parasol onkyo.fluid help=PWR
```

## Command Reference

### Actions

| Action | Description |
|--------|-------------|
| `discover` | Scan network for Onkyo receivers via UDP broadcast |
| `info=true\|verbose` | Show device information and available commands |
| `command=VALUE` | Send custom command(s) (raw ISCP or comma-separated) |
| `help=true|CMD` | Show general help or help for specific 3-letter command |

### Configuration Options

| Option | Description | Default |
|--------|-------------|---------|
| `host=IP` | Target IP address | Auto-discover |
| `port=NUMBER` | Target port | 60128 |
| `timeout=SECONDS` | Connection timeout in seconds | 5 |
| `response-output=FILE` | Log all responses to file in hex format | None |
| `limit=NUMBER` | Limit number of discovered devices (with discover) | Unlimited |

### Friendly Commands

The program supports friendly command syntax for common operations. Instead of using raw ISCP commands, you can use intuitive parameter names:

```bash
# Power control
parasol onkyo.fluid power=on
parasol onkyo.fluid power=off
parasol onkyo.fluid power=standby

# Volume control (if supported by your model)
parasol onkyo.fluid volume=30
parasol onkyo.fluid mute=on
parasol onkyo.fluid mute=off

# Input selection (model-specific)
parasol onkyo.fluid input=tv
parasol onkyo.fluid input=bluray
```

To see all available friendly commands for your specific receiver model, run:
```bash
parasol onkyo.fluid info=verbose
```

## Notes

- All numeric values in commands are hexadecimal
- The program automatically discovers receivers if no host is specified
- For verbose logging, add `--log-api` to the command line
- When sending multiple commands, a 250ms delay is inserted between each command

## Library Usage

The Onkyo modules can also be used as Fluid libraries in your own scripts:

```lua
local eiscp = require('./lib/eiscp')
local discovery = require('./lib/discovery')

-- Discover receivers
local receivers = discovery.findReceivers(5.0)  -- 5 second timeout

-- Connect to a receiver
local connection = eiscp.connect('192.168.1.40', 60128, 5.0)

-- Send commands
connection:sendCommand('PWR01')  -- Power on
connection:sendCommand('PWRQSTN')  -- Query power status

-- Disconnect
connection:disconnect()
```
