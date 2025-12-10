# Lab 01: OSI Model and Networking Basics

> This lab introduces fundamental networking concepts and tools. All exercises are designed for Ubuntu 24.04 LTS.

## A. Understanding the OSI Model

> The **OSI (Open Systems Interconnection)** model is a conceptual framework that describes how network communication works in seven layers.

### OSI Model Layers

| Layer | Name | Function | Example Protocols | PDU |
|-------|------|----------|-------------------|-----|
| 7 | Application | User interface, network services | HTTP, FTP, SSH, DNS | Data |
| 6 | Presentation | Data formatting, encryption | SSL/TLS, JPEG, ASCII | Data |
| 5 | Session | Session management | NetBIOS, RPC | Data |
| 4 | Transport | End-to-end delivery, reliability | TCP, UDP | Segment |
| 3 | Network | Logical addressing, routing | IP, ICMP | Packet |
| 2 | Data Link | Physical addressing, framing | Ethernet, Wi-Fi | Frame |
| 1 | Physical | Bit transmission | Cables, signals | Bits |

> **PDU (Protocol Data Unit)** is the term for data at each layer. As data moves down the stack, each layer adds its header (encapsulation).

### Mnemonic Devices

**Top-down:** "All People Seem To Need Data Processing"

**Bottom-up:** "Please Do Not Throw Sausage Pizza Away"

## B. Setting Up the Lab Environment

1. Update your system and install networking tools.
```
sudo apt-get update && sudo apt-get install -y net-tools iproute2 tcpdump traceroute dnsutils iputils-ping curl wget nmap
```

> These are the essential networking tools we'll use throughout all labs.

## C. Exploring Network Interfaces (Layer 1-2)

> Network interfaces are the physical or virtual connection points between your system and the network.

1. List all network interfaces using the modern `ip` command.
```
ip link show
```

> Each interface shows: index number, name (e.g., eth0, enp0s3, lo), state (UP/DOWN), and MAC address.

2. View the same information using the legacy `ifconfig` command.
```
ifconfig -a
```

> **Note:** `ifconfig` is deprecated but still common in older documentation. Prefer `ip` commands.

3. Show detailed information about a specific interface.
```
# Replace eth0 with your actual interface name
ip link show eth0
```

4. View the MAC (hardware) address of your interfaces.
```
ip link show | grep link/ether
```

> The **MAC address** is a 48-bit hardware address (e.g., `00:1a:2b:3c:4d:5e`) that operates at Layer 2 (Data Link).

5. Display interface statistics (packets sent/received, errors).
```
ip -s link show
```

## D. Exploring IP Addresses (Layer 3)

> IP addresses provide logical addressing at the Network layer, enabling routing across networks.

1. View all IP addresses assigned to your interfaces.
```
ip addr show
```

> Look for `inet` (IPv4) and `inet6` (IPv6) addresses.

2. Show only IPv4 addresses.
```
ip -4 addr show
```

3. Show only IPv6 addresses.
```
ip -6 addr show
```

4. View your primary IP address in a concise format.
```
hostname -I
```

5. Examine the loopback interface.
```
ip addr show lo
```

> The **loopback interface** (127.0.0.1) allows a machine to communicate with itself. It's used for local testing and inter-process communication.

## E. Testing Connectivity with Ping (Layer 3)

> The `ping` command uses ICMP (Internet Control Message Protocol) to test reachability of hosts.

1. Ping the loopback address to verify your TCP/IP stack works.
```
ping -c 4 127.0.0.1
```

> `-c 4` limits to 4 packets. Without it, ping runs continuously until interrupted with Ctrl+C.

2. Ping your default gateway (typically your router).
```
# First, find your gateway
ip route | grep default
```
```
# Then ping it (replace with your actual gateway IP)
ping -c 4 $(ip route | grep default | awk '{print $3}')
```

3. Ping an external host.
```
ping -c 4 8.8.8.8
```

> 8.8.8.8 is Google's public DNS server, commonly used for testing Internet connectivity.

4. Ping a domain name to test DNS resolution.
```
ping -c 4 google.com
```

> If this works but pinging 8.8.8.8 doesn't, you have DNS working but no Internet route. If 8.8.8.8 works but google.com doesn't, DNS is the problem.

5. Show the path packets take to a destination.
```
traceroute google.com
```

> Each line represents a "hop" (router) along the path. The three times shown are round-trip times for probe packets.

## F. Examining Active Connections (Layer 4)

> The Transport layer manages end-to-end connections using TCP and UDP protocols.

1. Show all active TCP connections.
```
ss -t
```

2. Show all listening TCP ports.
```
ss -tl
```

> Listening ports are services waiting for incoming connections.

3. Show TCP connections with process information (requires sudo).
```
sudo ss -tlp
```

> The `-p` flag shows which process owns each socket.

4. Show all UDP sockets.
```
ss -u
```

5. Show all connections with numeric addresses (don't resolve names).
```
ss -tuln
```

> `-n` prevents DNS lookups, making the output faster.

6. View the same information using the legacy `netstat` command.
```
netstat -tuln
```

7. Show connection statistics.
```
ss -s
```

## G. Capturing Network Traffic (All Layers)

> `tcpdump` captures packets at the network interface level, showing data from multiple OSI layers.

1. List available interfaces for capture.
```
sudo tcpdump -D
```

2. Capture packets on the default interface (press Ctrl+C to stop).
```
# Capture 10 packets
sudo tcpdump -c 10
```

3. Capture only ICMP (ping) traffic.
```
# In one terminal, start the capture
sudo tcpdump -c 5 icmp
```
```
# In another terminal, generate ping traffic
ping -c 3 8.8.8.8
```

4. Capture traffic on a specific port (e.g., HTTP port 80).
```
# Start capture in one terminal
sudo tcpdump -c 5 port 80
```
```
# In another terminal, make an HTTP request
curl http://example.com
```

5. Show packet contents in ASCII.
```
sudo tcpdump -c 3 -A port 80
```

6. Show packet contents in hex and ASCII.
```
sudo tcpdump -c 3 -XX port 80
```

> This shows the actual bytes being transmitted, including Ethernet headers (Layer 2), IP headers (Layer 3), TCP headers (Layer 4), and payload (Layer 7).

7. Save captured packets to a file for later analysis.
```
sudo tcpdump -c 20 -w /tmp/capture.pcap
```

8. Read packets from a saved file.
```
sudo tcpdump -r /tmp/capture.pcap
```

> Saved `.pcap` files can be opened in Wireshark for detailed graphical analysis.

## H. Examining Network Services (Layer 7)

> Application layer protocols provide services directly to users and applications.

1. Make an HTTP request and see the headers.
```
curl -v http://example.com 2>&1 | head -30
```

> The `-v` flag shows the HTTP request and response headers.

2. View just the HTTP response headers.
```
curl -I http://example.com
```

3. Test HTTPS connectivity.
```
curl -I https://example.com
```

4. Check what services are running on well-known ports.
```
sudo ss -tlnp | grep -E ':(22|80|443|53)\s'
```

> This checks for SSH (22), HTTP (80), HTTPS (443), and DNS (53).

## I. Mapping OSI Layers to Commands

| Layer | Commands | What They Show |
|-------|----------|----------------|
| 1-2 Physical/Data Link | `ip link`, `ifconfig` | Interfaces, MAC addresses |
| 3 Network | `ip addr`, `ping`, `traceroute` | IP addresses, reachability, routing |
| 4 Transport | `ss`, `netstat` | TCP/UDP connections, ports |
| 7 Application | `curl`, `dig`, `nslookup` | HTTP requests, DNS queries |
| All | `tcpdump` | Raw packet capture |

## J. Clean Up

```
# Remove the capture file if created
rm -f /tmp/capture.pcap
```

## K. Key Takeaways

- The OSI model provides a framework for understanding network communication
- Each layer has specific responsibilities and protocols
- Modern Linux uses `ip` commands; `ifconfig`/`netstat` are legacy but still useful
- `tcpdump` lets you see actual network traffic at all layers
- Understanding layers helps isolate problems: can you ping the IP but not the hostname? That's a Layer 7 (DNS) issue!
