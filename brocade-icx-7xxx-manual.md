# Brocade ICX-7150 FastIron 8 Configuration Manual

## Switch Information
- **Model**: ICX7150-48Z-HPOE (48-port PoE + 8x 10G SFP+)
- **Software**: FastIron 08.0.95qT213
- **PoE Capacity**: 740W total
- **Management IP**: 192.168.2.10/24 (VLAN 1)

## Basic Navigation
```bash
# SSH Connection (requires legacy crypto)
ssh -oKexAlgorithms=+diffie-hellman-group14-sha1 -oHostKeyAlgorithms=+ssh-rsa 192.168.2.10 -l super

# Command modes
enable                    # Enter privileged mode (no password set)
configure terminal        # Enter global configuration mode
exit                     # Exit current mode
show running-config      # View current configuration
write memory            # Save configuration
```

## VLAN Configuration ✅ TESTED
```bash
configure terminal

# Create VLAN and add ports
vlan <vlan-id>
  tagged ethernet <port-list>      # Add trunk ports (802.1Q tagged)
  untagged ethernet <port-list>    # Add access ports (untagged)
  router-interface ve <vlan-id>    # Create VLAN interface (SVI)
  exit

# Remove ports from VLAN
vlan <vlan-id>
  no tagged ethernet <port-list>   # Remove trunk ports
  no untagged ethernet <port-list> # Remove access ports

# Delete VLAN
no vlan <vlan-id>

# View VLANs
show vlan
show vlan <vlan-id>
```

## Trunk Port Configuration ✅ TESTED
```bash
# Trunk ports are created by adding tagged VLANs to interfaces
configure terminal
vlan <vlan-id>
  tagged ethernet <port-list>     # Ports 1/1/1-3 are trunks carrying VLANs 50,70

# Example from current config:
# - Ports 1/1/1-3 are trunks carrying VLANs 50 and 70 tagged
# - All other ports are access ports in VLAN 1 (untagged)
```

## Interface Configuration ✅ TESTED
```bash
configure terminal
interface ethernet <stack/slot/port>

# Port description
port-name "<description>"         # Set port name/description

# Speed/Duplex settings
speed-duplex auto                 # Auto-negotiate (default)
speed-duplex 10-full             # 10 Mbps full duplex
speed-duplex 100-full            # 100 Mbps full duplex  
speed-duplex 1000-full           # 1 Gbps full duplex
speed-duplex 2500-full           # 2.5 Gbps full duplex
speed-duplex 5g-full             # 5 Gbps full duplex
speed-duplex 10g-full            # 10 Gbps full duplex
speed-duplex 25g-full            # 25 Gbps full duplex
speed-duplex 40g-full            # 40 Gbps full duplex
speed-duplex 100g-full           # 100 Gbps full duplex
speed-duplex optic               # Auto-detect based on SFP type

# Interface control
enable                           # Enable interface
disable                          # Disable interface
exit
```

## PoE Configuration ✅ TESTED
```bash
configure terminal

# Basic PoE control
inline power ethernet <port>                     # Enable PoE on port
no inline power ethernet <port>                  # Disable PoE on port

# Power management
inline power ethernet <port> power-limit <milliwatts>  # Set power limit
  # Standard PoE: 1000-30000 mW (1-30W)
  # PoE+ (PoH): up to 95000 mW (95W)  
  # PoE++ (BT): up to 120000 mW (120W)

# Priority management  
inline power ethernet <port> priority <1-3>      # 1=highest, 3=lowest

# Advanced options
inline power ethernet <port> overdrive           # Allow >standard power (WARNING: cycles power)
inline power ethernet <port> power-by-class      # Auto-allocate by device class
inline power ethernet <port> couple-datalink     # Link PoE to data link state
inline power ethernet <port> poe-ha             # High availability (power during reload)
```

## Monitoring Commands ✅ TESTED
```bash
# System information
show version                     # Hardware/software details
show interface brief            # Port status summary
show running-config             # Current configuration

# VLAN information  
show vlan                       # All VLANs
show vlan <vlan-id>            # Specific VLAN

# PoE information
show poe                        # All PoE port status  
show poe ethernet <port>        # Specific port PoE status

# Interface information
show interface ethernet <port>   # Detailed interface status
show interface brief | include <pattern>  # Filtered interface list
```

## Current Switch Configuration Summary
- **VLAN 1**: Default VLAN, all ports untagged (except test changes)
- **VLAN 50**: Trunk VLAN on ports 1/1/1-3 and 1/2/1-3 (tagged)
- **VLAN 70**: Trunk VLAN on ports 1/1/1-3 (tagged)
- **Port 1/1/1**: "Router_Uplink" - trunk carrying VLANs 50,70
- **Ports 1/1/2-3**: AP ports - trunks carrying VLANs 50,70
- **PoE**: 740W capacity, all ports enabled but no devices powered
- **10G Module**: Ports 1/2/2,4-8 forced to 1G speed

## Layer 3 Routing Configuration ⚠️ TO TEST
```bash
# Enable IP routing globally
configure terminal
ip routing                       # Enable IP routing on the switch

# Create VLAN interfaces (SVIs) for inter-VLAN routing
interface ve <vlan-id>
  ip address <ip-address> <subnet-mask>  # Assign IP to VLAN interface
  ip helper-address <dhcp-server-ip>     # DHCP relay (optional)
  exit

# Static routing
ip route <destination-network> <subnet-mask> <next-hop-ip>
ip route 0.0.0.0 0.0.0.0 <default-gateway>  # Default route

# Example inter-VLAN routing setup:
interface ve 1
  ip address 192.168.1.1 255.255.255.0    # Gateway for VLAN 1
interface ve 50  
  ip address 192.168.50.1 255.255.255.0   # Gateway for VLAN 50
interface ve 70
  ip address 192.168.70.1 255.255.255.0   # Gateway for VLAN 70
```

## Routing Monitoring Commands ⚠️ TO TEST
```bash
# Routing table
show ip route                    # Display routing table
show ip route summary           # Routing table summary
show ip route <network>         # Specific route

# VLAN interfaces
show interface ve              # All VLAN interfaces
show interface ve <vlan-id>    # Specific VLAN interface

# ARP table
show arp                       # ARP table
show arp <ip-address>         # Specific ARP entry

# IP configuration
show ip interface             # IP interface status
show ip interface brief       # Brief IP interface status
```

## Advanced Routing Features ⚠️ TO TEST
```bash
# OSPF configuration
router ospf
  area <area-id>
  network <network> <wildcard-mask> area <area-id>
  exit

# Access Control Lists (ACLs)
access-list <number> <permit|deny> <source> <destination>
interface ethernet <port>
  ip access-group <acl-number> <in|out>

# DHCP relay
ip helper-address <dhcp-server-ip>  # Applied to VLAN interfaces
```
