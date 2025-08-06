# Brocade ICX-7150 FastIron Configuration Manual

## Table of Contents
1. [Hardware Information](#hardware-information)
2. [Basic Operations](#basic-operations)
3. [VLAN Configuration](#vlan-configuration)
4. [Interface Configuration](#interface-configuration)
5. [PoE Configuration](#poe-configuration)
6. [Monitoring Commands](#monitoring-commands)
7. [Configuration Examples](#configuration-examples)
8. [Current Switch Reference](#current-switch-reference)

---

## Hardware Information

### ICX-7150-48ZP-HPOE Specifications
- **48 Ethernet Ports**: Ports 1-16 (2.5Gbe capable), Ports 17-48 (1Gbe)
- **8 SFP+ Ports**: 10Gbe capable (ports 1/2/1-8)
- **PoE Capacity**: 740W total
- **Management**: 1x management port, console port
- **Stacking**: ICX stacking capable

---

## Basic Operations

### SSH Connection
```bash
# Requires legacy crypto support for older firmware
ssh -oKexAlgorithms=+diffie-hellman-group14-sha1 -oHostKeyAlgorithms=+ssh-rsa -oMACs=+hmac-sha1 super@<switch-ip>
```

### Command Navigation
```bash
# Basic navigation
enable                    # Enter privileged mode
configure terminal        # Enter global configuration mode
exit                     # Exit current mode
end                      # Return to privileged mode from any config mode

# Configuration management
show running-config      # View current configuration
write memory            # Save configuration to startup-config
copy running-config tftp <ip> <filename>  # Backup config
```

---

## VLAN Configuration

### Creating VLANs
```bash
configure terminal
vlan <vlan-id>
  name "<vlan-name>"              # Optional: Set VLAN name
  tagged ethernet <port-list>     # Add trunk ports (carries multiple VLANs)
  untagged ethernet <port-list>   # Add access ports (single VLAN)
  router-interface ve <vlan-id>   # Create Layer 3 interface
  exit
```

### VLAN Management
```bash
# View VLANs
show vlan                        # Show all VLANs
show vlan <vlan-id>             # Show specific VLAN

# Modify VLAN membership
vlan <vlan-id>
  tagged ethernet <port>         # Add trunk port
  untagged ethernet <port>       # Add access port
  no tagged ethernet <port>      # Remove trunk port
  no untagged ethernet <port>    # Remove access port

# Delete VLAN
no vlan <vlan-id>
```

### Trunk vs Access Ports
```bash
# Trunk Port (carries multiple VLANs with 802.1Q tags)
vlan 10
  tagged ethernet 1/1/1          # Port carries VLAN 10 tagged
vlan 20
  tagged ethernet 1/1/1          # Same port also carries VLAN 20 tagged

# Access Port (single VLAN, untagged)
vlan 10
  untagged ethernet 1/1/5        # Port is in VLAN 10 only
```

---

## Interface Configuration

### Basic Interface Settings
```bash
configure terminal
interface ethernet <stack/slot/port>
  port-name "<description>"      # Set port description
  enable                         # Enable interface (default)
  disable                        # Disable interface
  exit
```

### Speed and Duplex Configuration
```bash
interface ethernet <port>
  speed-duplex auto              # Auto-negotiate (default)
  speed-duplex 10-full           # 10 Mbps full duplex
  speed-duplex 100-full          # 100 Mbps full duplex
  speed-duplex 1000-full         # 1 Gbps full duplex
  speed-duplex 2500-full         # 2.5 Gbps full duplex (ports 1-16 only)
  speed-duplex 10g-full          # 10 Gbps full duplex
  speed-duplex optic             # Auto-detect based on SFP (SFP+ ports)
```

---

## PoE Configuration

### Basic PoE Control
```bash
configure terminal
inline power ethernet <port>                    # Enable PoE
no inline power ethernet <port>                 # Disable PoE
```

### Power Management
```bash
# Set power limits
inline power ethernet <port> power-limit <milliwatts>
  # Standard PoE: 1000-30000 mW (1-30W)
  # PoE+: up to 95000 mW (95W)
  # PoE++: up to 120000 mW (120W)

# Priority settings
inline power ethernet <port> priority <1-3>     # 1=highest, 3=lowest

# Advanced options
inline power ethernet <port> power-by-class     # Auto-allocate by device class
inline power ethernet <port> couple-datalink    # Link PoE to data state
```

---

## Monitoring Commands

### System Information
```bash
show version                     # Hardware/software details
show system                      # System status
show flash                       # Flash memory usage
```

### Interface Monitoring
```bash
show interface brief             # Port status summary
show interface ethernet <port>   # Detailed interface information
show interface ethernet <port> | include <pattern>  # Filter output (limited support)
```

### VLAN and Network Monitoring
```bash
show vlan                        # All VLANs and membership
show vlan <vlan-id>             # Specific VLAN details
show mac-address                 # MAC address table
show spanning-tree              # STP status
```

### PoE Monitoring
```bash
show poe                         # All PoE port status
show poe ethernet <port>         # Specific port PoE details
show poe power-consumption       # Power usage summary
```

---

## Configuration Examples

### Example 1: Basic VLAN Setup
```bash
# Create management VLAN
configure terminal
vlan 100
  name "Management"
  untagged ethernet 1/1/48
  router-interface ve 100
  exit

# Configure management interface
interface ve 100
  ip address 192.168.100.1/24
  exit
```

### Example 2: Trunk Configuration
```bash
# Configure trunk port for AP
configure terminal
vlan 10
  name "Users"
  tagged ethernet 1/1/1
vlan 20
  name "Guests"
  tagged ethernet 1/1/1
vlan 30
  name "Management"
  tagged ethernet 1/1/1
  exit

interface ethernet 1/1/1
  port-name "Wireless_AP_Trunk"
  exit
```

### Example 3: PoE Camera Setup
```bash
# Configure PoE for IP cameras
configure terminal
vlan 70
  name "Cameras"
  untagged ethernet 1/1/47
  untagged ethernet 1/1/48
  exit

interface ethernet 1/1/47
  port-name "Camera_1"
  inline power ethernet 1/1/47
  inline power ethernet 1/1/47 power-limit 15000
  exit

interface ethernet 1/1/48
  port-name "Camera_2"
  inline power ethernet 1/1/48
  inline power ethernet 1/1/48 power-limit 15000
  exit
```

---

## Current Switch Reference

### Network Information
- **Model**: ICX7150-48ZP-HPOE
- **Software**: FastIron 08.0.95qT213
- **Management IP**: 192.168.2.10/24 (VLAN 1)
- **PoE Budget**: 740W total

### VLAN Configuration
- **VLAN 1**: Default/Management VLAN
- **VLAN 50**: Desktop VLAN (Port 1/1/4)
- **VLAN 70**: Camera VLAN (Ports 1/1/47-48)

### Port Assignments
```bash
# Infrastructure Ports (2.5Gbe Capable)
1/1/1: "Router_Uplink"           # Trunk (VLANs 50,70 tagged)
1/1/2: "Downstairs_AP"           # Trunk (VLANs 50,70 tagged)
1/1/3: "Upstairs_AP"             # Trunk (VLANs 50,70 tagged)
1/1/4: "Desktop_Upstairs_VLAN50" # VLAN 50 access

# Available High-Speed Ports (2.5Gbe Capable)
1/1/5-16: Available              # VLAN 1 (12 ports available)

# Available Standard Ports (1Gbe)
1/1/17-46: Available             # VLAN 1 (30 ports available)

# Camera Ports (1Gbe)
1/1/47: "Garage_Camera_VLAN70"         # VLAN 70 access
1/1/48: "Three_Season_Room_Camera_VLAN70" # VLAN 70 access

# Server/Storage Ports (10Gbe SFP+)
1/2/1: "PVE_Host_1"              # VLAN 1 access
1/2/2: "PVE_Host_2"              # VLAN 1 access
1/2/3: "Aux_Gaming_Desktops"     # VLAN 1 access
1/2/8: "Unifi_NAS_VLAN1"         # VLAN 1 access
```

### Port Capacity Summary
- **2.5Gbe Ports**: 13 total (1 used for desktop, 12 available)
- **1Gbe Standard Ports**: 32 total (2 used for cameras, 30 available)
- **10Gbe SFP+ Ports**: 4 configured for servers/storage
- **Trunk Ports**: 3 configured for network infrastructure

### Quick Verification Commands
```bash
show interface brief             # Port status and VLAN assignments
show vlan                        # VLAN membership verification
show poe                         # PoE status for camera ports
```

---

## Configuration Status: ✅ COMPLETE
Switch is optimally configured with efficient port allocation and proper VLAN segmentation.
