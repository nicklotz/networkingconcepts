# Lab 03: IP Addressing, Subnets, and CIDR

> This lab explores IP addressing fundamentals, subnet calculations, and CIDR notation. All exercises can be performed on Ubuntu 24.04.

## A. IPv4 Address Fundamentals

> An **IPv4 address** is a 32-bit number that uniquely identifies a device on a network. It's typically written in dotted-decimal notation (e.g., 192.168.1.100).

### IPv4 Structure

```
Dotted Decimal:  192    .    168    .    1      .    100
Binary:          11000000    10101000    00000001    01100100
Bits:            8 bits      8 bits      8 bits      8 bits = 32 bits total
```

### Special Addresses

| Address | Purpose |
|---------|---------|
| 0.0.0.0 | Default route / "any" address |
| 127.0.0.1 | Loopback (localhost) |
| 255.255.255.255 | Broadcast to all hosts |
| 169.254.x.x | Link-local (APIPA - no DHCP) |

## B. Install Subnet Calculator Tools

1. Install `ipcalc` for subnet calculations.
```
sudo apt-get update && sudo apt-get install -y ipcalc sipcalc
```

> `ipcalc` and `sipcalc` are command-line tools that calculate subnet information.

## C. Examining Your IP Configuration

1. View all IP addresses on your system.
```
ip addr show
```

2. Display only IPv4 addresses.
```
ip -4 addr show
```

3. View IP configuration in a cleaner format.
```
ip -4 addr show | grep -E '(inet|^[0-9])'
```

4. Show the primary IP address.
```
hostname -I | awk '{print $1}'
```

5. View detailed interface information.
```
ip -d link show
```

## D. Understanding IP Address Classes

> **Classful addressing** (historical) divided IP addresses into classes based on the first octet.

| Class | First Octet Range | Default Subnet Mask | Network/Host Bits |
|-------|-------------------|---------------------|-------------------|
| A | 1-126 | 255.0.0.0 (/8) | 8 / 24 |
| B | 128-191 | 255.255.0.0 (/16) | 16 / 16 |
| C | 192-223 | 255.255.255.0 (/24) | 24 / 8 |
| D | 224-239 | N/A | Multicast |
| E | 240-255 | N/A | Experimental |

> **Note:** 127.x.x.x is reserved for loopback and not used for regular addressing.

1. Determine the class of various IP addresses.
```
# Class A example
ipcalc 10.0.0.1
```
```
# Class B example
ipcalc 172.16.0.1
```
```
# Class C example
ipcalc 192.168.1.1
```

## E. Private IP Address Ranges

> **Private IP addresses** are reserved for internal networks and are not routable on the public Internet.

| Class | Range | CIDR | Total Addresses |
|-------|-------|------|-----------------|
| A | 10.0.0.0 - 10.255.255.255 | 10.0.0.0/8 | 16,777,216 |
| B | 172.16.0.0 - 172.31.255.255 | 172.16.0.0/12 | 1,048,576 |
| C | 192.168.0.0 - 192.168.255.255 | 192.168.0.0/16 | 65,536 |

1. Check if your IP is in a private range.
```
MY_IP=$(hostname -I | awk '{print $1}')
echo "Your IP: $MY_IP"
ipcalc $MY_IP
```

2. View private range details.
```
echo "=== Class A Private ==="
ipcalc 10.0.0.0/8 | head -10
echo ""
echo "=== Class B Private ==="
ipcalc 172.16.0.0/12 | head -10
echo ""
echo "=== Class C Private ==="
ipcalc 192.168.0.0/16 | head -10
```

## F. Understanding CIDR Notation

> **CIDR (Classless Inter-Domain Routing)** replaced classful addressing and allows flexible subnet sizes.

### CIDR Format
```
192.168.1.0/24
     │        │
     │        └── Number of network bits (subnet mask)
     └── Network address
```

### Common CIDR Notations

| CIDR | Subnet Mask | Usable Hosts | Common Use |
|------|-------------|--------------|------------|
| /8 | 255.0.0.0 | 16,777,214 | Large enterprise |
| /16 | 255.255.0.0 | 65,534 | Medium enterprise |
| /24 | 255.255.255.0 | 254 | Small office |
| /25 | 255.255.255.128 | 126 | Small network |
| /26 | 255.255.255.192 | 62 | Very small network |
| /27 | 255.255.255.224 | 30 | Tiny network |
| /28 | 255.255.255.240 | 14 | Point-to-multipoint |
| /30 | 255.255.255.252 | 2 | Point-to-point link |
| /32 | 255.255.255.255 | 1 | Single host |

1. Explore different CIDR blocks.
```
echo "=== /24 Network ==="
ipcalc 192.168.1.0/24
```
```
echo "=== /25 Network ==="
ipcalc 192.168.1.0/25
```
```
echo "=== /30 Network (Point-to-Point) ==="
ipcalc 192.168.1.0/30
```

## G. Subnet Calculations

> **Subnetting** divides a larger network into smaller, more manageable networks.

### Key Formulas
- **Usable hosts** = 2^(host bits) - 2 (subtract network and broadcast addresses)
- **Number of subnets** = 2^(borrowed bits)

1. Calculate subnet details for a /24 network.
```
ipcalc 192.168.1.0/24
```

> Note the output shows:
> - Network address (first address)
> - Broadcast address (last address)
> - HostMin and HostMax (usable range)
> - Total hosts

2. Split a /24 into four /26 subnets.
```
echo "Original /24 network:"
ipcalc 192.168.1.0/24 | grep -E '(Network|Broadcast|HostMin|HostMax|Hosts)'
echo ""
echo "Split into four /26 subnets:"
echo ""
echo "Subnet 1:"
ipcalc 192.168.1.0/26 | grep -E '(Network|Broadcast|HostMin|HostMax|Hosts)'
echo ""
echo "Subnet 2:"
ipcalc 192.168.1.64/26 | grep -E '(Network|Broadcast|HostMin|HostMax|Hosts)'
echo ""
echo "Subnet 3:"
ipcalc 192.168.1.128/26 | grep -E '(Network|Broadcast|HostMin|HostMax|Hosts)'
echo ""
echo "Subnet 4:"
ipcalc 192.168.1.192/26 | grep -E '(Network|Broadcast|HostMin|HostMax|Hosts)'
```

3. Use sipcalc for additional details.
```
sipcalc 192.168.1.0/24
```

4. Calculate how many hosts fit in different subnet sizes.
```
for cidr in 24 25 26 27 28 29 30; do
    hosts=$(ipcalc 192.168.1.0/$cidr | grep 'Hosts/Net' | awk '{print $2}')
    mask=$(ipcalc 192.168.1.0/$cidr | grep 'Netmask' | awk '{print $2}')
    echo "/$cidr ($mask): $hosts usable hosts"
done
```

## H. Binary Conversion Practice

> Understanding binary helps with subnet calculations and troubleshooting.

1. Convert an IP address to binary.
```
IP="192.168.1.100"
for octet in $(echo $IP | tr '.' ' '); do
    printf "%08d." $(echo "obase=2;$octet" | bc)
done | sed 's/.$/\n/'
```

2. View subnet mask in binary.
```
MASK="255.255.255.0"
echo "Subnet mask: $MASK"
for octet in $(echo $MASK | tr '.' ' '); do
    printf "%08d." $(echo "obase=2;$octet" | bc)
done | sed 's/.$/\n/'
```

3. Calculate network address using bitwise AND.
```
# The network address is IP AND Subnet Mask
IP="192.168.1.100"
CIDR=24
echo "IP: $IP"
echo "CIDR: /$CIDR"
ipcalc $IP/$CIDR | grep -E '(Network|Netmask)'
```

## I. IPv6 Fundamentals

> **IPv6** uses 128-bit addresses, providing a vastly larger address space than IPv4.

### IPv6 Format
```
Full:        2001:0db8:85a3:0000:0000:8a2e:0370:7334
Compressed:  2001:db8:85a3::8a2e:370:7334
```

### IPv6 Address Types

| Type | Prefix | Description |
|------|--------|-------------|
| Global Unicast | 2000::/3 | Publicly routable |
| Link-Local | fe80::/10 | Single link only |
| Unique Local | fc00::/7 | Private (like RFC1918) |
| Loopback | ::1 | Localhost |
| Multicast | ff00::/8 | One-to-many |

1. View your IPv6 addresses.
```
ip -6 addr show
```

2. View only link-local addresses.
```
ip -6 addr show scope link
```

3. View global IPv6 addresses (if any).
```
ip -6 addr show scope global
```

4. Ping the IPv6 loopback.
```
ping -6 -c 3 ::1
```

5. Test IPv6 connectivity to external hosts.
```
ping -6 -c 3 ipv6.google.com 2>/dev/null || echo "IPv6 not available or Google IPv6 unreachable"
```

6. Use ipcalc with IPv6.
```
sipcalc 2001:db8::1/64
```

## J. Practical IP Address Management

1. View your current network configuration.
```
echo "=== Interface Information ==="
ip -4 addr show | grep -E '(^[0-9]|inet)'
echo ""
echo "=== Default Gateway ==="
ip route | grep default
echo ""
echo "=== DNS Servers ==="
cat /etc/resolv.conf | grep nameserver
```

2. Determine which subnet your IP belongs to.
```
MY_IP=$(hostname -I | awk '{print $1}')
# Get subnet info from interface
ip -4 addr show | grep "$MY_IP" | awk '{print $2}'
```

3. Check if two IPs are on the same subnet.
```
check_same_subnet() {
    IP1=$1
    IP2=$2
    CIDR=$3

    NET1=$(ipcalc $IP1/$CIDR | grep Network | awk '{print $2}')
    NET2=$(ipcalc $IP2/$CIDR | grep Network | awk '{print $2}')

    echo "IP1 $IP1 is in network: $NET1"
    echo "IP2 $IP2 is in network: $NET2"

    if [ "$NET1" = "$NET2" ]; then
        echo "Result: Same subnet!"
    else
        echo "Result: Different subnets"
    fi
}

# Test with two IPs
check_same_subnet 192.168.1.10 192.168.1.200 24
echo ""
check_same_subnet 192.168.1.10 192.168.2.10 24
```

4. List all possible host addresses in a small subnet.
```
echo "All hosts in 192.168.1.0/29:"
sipcalc 192.168.1.0/29 | grep -A 10 "Usable range"
```

## K. Subnet Design Exercise

> Design subnets for a small office with these requirements:
> - Server network: 10 hosts
> - Employee network: 50 hosts
> - Guest network: 20 hosts
> - Management network: 5 hosts

1. Calculate the required subnet sizes.
```
echo "=== Subnet Design ==="
echo ""
echo "Server network (10 hosts) - need /28 (14 usable):"
ipcalc 10.0.0.0/28 | grep -E '(Network|HostMin|HostMax|Hosts)'
echo ""
echo "Employee network (50 hosts) - need /26 (62 usable):"
ipcalc 10.0.0.64/26 | grep -E '(Network|HostMin|HostMax|Hosts)'
echo ""
echo "Guest network (20 hosts) - need /27 (30 usable):"
ipcalc 10.0.0.128/27 | grep -E '(Network|HostMin|HostMax|Hosts)'
echo ""
echo "Management network (5 hosts) - need /29 (6 usable):"
ipcalc 10.0.0.160/29 | grep -E '(Network|HostMin|HostMax|Hosts)'
```

## L. Clean Up

No cleanup needed for this lab - we only installed packages and ran read-only commands.

## M. Key Takeaways

- IPv4 addresses are 32-bit numbers written in dotted-decimal notation
- Private IP ranges (10.x, 172.16-31.x, 192.168.x) are for internal networks
- CIDR notation (/24, /16, etc.) replaced classful addressing for flexibility
- Subnetting divides networks for security, organization, and efficiency
- Tools like `ipcalc` and `sipcalc` simplify subnet calculations
- IPv6 provides a much larger address space with 128-bit addresses
- Understanding binary math helps with manual subnet calculations
