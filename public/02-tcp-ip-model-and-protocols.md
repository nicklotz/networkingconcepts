# Lab 02: TCP/IP Model and Protocols

> This lab explores the TCP/IP model in depth, examining TCP vs UDP, the three-way handshake, and common protocols and ports.

## A. TCP/IP Model Overview

> The **TCP/IP model** is a practical, 4-layer model that describes how the Internet actually works. Unlike the theoretical OSI model, TCP/IP was developed alongside the Internet protocols.

### TCP/IP vs OSI Comparison

| TCP/IP Layer | OSI Layers | Function | Protocols |
|--------------|------------|----------|-----------|
| Application | 5, 6, 7 | User services | HTTP, FTP, SSH, DNS, SMTP |
| Transport | 4 | End-to-end delivery | TCP, UDP |
| Internet | 3 | Routing, addressing | IP, ICMP, ARP |
| Link (Network Access) | 1, 2 | Physical transmission | Ethernet, Wi-Fi |

## B. Examining Network Interfaces (Link Layer)

1. View your network interfaces and their states.
```
ip link show
```

2. Show the ARP cache (maps IP addresses to MAC addresses).
```
ip neigh show
```

> **ARP (Address Resolution Protocol)** discovers the MAC address for a given IP address on the local network.

3. Clear and repopulate the ARP cache.
```
# View current cache
ip neigh show
```
```
# Ping a host to ensure it's in the cache
ping -c 1 $(ip route | grep default | awk '{print $3}')
```
```
# View the cache again
ip neigh show
```

4. View Ethernet frame statistics.
```
ip -s link show
```

> Statistics show packets transmitted/received, errors, and dropped frames at Layer 2.

## C. Understanding TCP vs UDP

### TCP (Transmission Control Protocol)
- **Connection-oriented**: Establishes connection before data transfer
- **Reliable**: Guarantees delivery with acknowledgments and retransmissions
- **Ordered**: Packets arrive in sequence
- **Flow control**: Prevents overwhelming the receiver
- **Use cases**: Web browsing, email, file transfer

### UDP (User Datagram Protocol)
- **Connectionless**: No connection establishment
- **Unreliable**: Best-effort delivery, no guarantees
- **Unordered**: Packets may arrive out of sequence
- **No flow control**: Sender can overwhelm receiver
- **Use cases**: Streaming, gaming, DNS queries

## D. Observing the TCP Three-Way Handshake

> TCP establishes connections using a three-step process: SYN → SYN-ACK → ACK

1. Start a packet capture filtering for TCP SYN packets.
```
sudo tcpdump -i any -c 10 'tcp[tcpflags] & (tcp-syn) != 0' &
```

2. Make a connection to trigger the handshake.
```
curl -s http://example.com > /dev/null
```

3. Bring the capture back to foreground to see results.
```
fg
```

4. For a more detailed view of the handshake, capture all TCP flags.
```
# Start capture
sudo tcpdump -i any -c 20 'host example.com and tcp' -nn &
```
```
# Make connection
curl -s http://example.com > /dev/null
```
```
# View results
fg
```

> Look for packets with flags:
> - `[S]` = SYN (connection request)
> - `[S.]` = SYN-ACK (connection accepted)
> - `[.]` = ACK (acknowledgment)
> - `[F.]` = FIN-ACK (connection closing)

5. View TCP connection states on your system.
```
ss -t state established
```

6. View all TCP states.
```
ss -ta
```

> Common TCP states: LISTEN, ESTABLISHED, TIME_WAIT, CLOSE_WAIT

## E. Examining Common Protocols and Ports

### Well-Known Ports Reference

| Port | Protocol | Service | Description |
|------|----------|---------|-------------|
| 20, 21 | TCP | FTP | File Transfer Protocol |
| 22 | TCP | SSH | Secure Shell |
| 23 | TCP | Telnet | Unencrypted remote access |
| 25 | TCP | SMTP | Simple Mail Transfer Protocol |
| 53 | TCP/UDP | DNS | Domain Name System |
| 67, 68 | UDP | DHCP | Dynamic Host Configuration |
| 80 | TCP | HTTP | Web traffic |
| 110 | TCP | POP3 | Post Office Protocol |
| 143 | TCP | IMAP | Internet Message Access Protocol |
| 443 | TCP | HTTPS | Secure web traffic |
| 3306 | TCP | MySQL | MySQL database |
| 5432 | TCP | PostgreSQL | PostgreSQL database |

1. Check which ports are listening on your system.
```
sudo ss -tlnp
```

2. Check for a specific port (e.g., SSH on port 22).
```
sudo ss -tlnp | grep ':22'
```

3. View the system's services file that maps port numbers to names.
```
head -50 /etc/services
```

4. Look up a specific service.
```
grep -E '^(ssh|http|https|dns|smtp)' /etc/services
```

## F. Testing TCP Connections

1. Test if a TCP port is open using netcat.
```
# Test SSH port on localhost
nc -zv localhost 22
```

> `-z` = scan mode (don't send data), `-v` = verbose

2. Test a remote port.
```
nc -zv -w 3 google.com 443
```

> `-w 3` = 3 second timeout

3. Test multiple ports.
```
nc -zv google.com 80 443
```

4. Use the `ss` command to check if you can reach a port.
```
# Check if any connection to port 443 exists
ss -tn dst :443
```

## G. Working with UDP

1. Show UDP listening sockets.
```
sudo ss -ulnp
```

2. Test DNS (UDP port 53) using dig.
```
dig @8.8.8.8 google.com
```

> DNS typically uses UDP for queries (faster) but TCP for zone transfers or large responses.

3. Capture UDP DNS traffic.
```
# Start capture
sudo tcpdump -i any -c 5 udp port 53 &
```
```
# Generate DNS traffic
dig google.com
```
```
# View results
fg
```

4. Compare UDP and TCP DNS queries.
```
# UDP query (default)
dig google.com
```
```
# Force TCP query
dig +tcp google.com
```

## H. Examining HTTP/HTTPS (Application Layer)

1. Make a simple HTTP request and view the exchange.
```
curl -v http://example.com 2>&1 | head -40
```

2. View only response headers.
```
curl -I http://example.com
```

3. Make an HTTPS request and see TLS negotiation.
```
curl -v https://example.com 2>&1 | head -50
```

4. Check the SSL/TLS certificate of a site.
```
echo | openssl s_client -connect example.com:443 2>/dev/null | openssl x509 -noout -subject -dates
```

5. Test HTTP on different ports (if available).
```
# Standard HTTP
curl -I http://example.com:80

# Standard HTTPS
curl -I https://example.com:443
```

## I. Examining SSH (Secure Shell)

1. Check if SSH is running locally.
```
sudo ss -tlnp | grep ':22'
```

2. View SSH server configuration.
```
grep -v '^#' /etc/ssh/sshd_config | grep -v '^$' | head -20
```

3. Test SSH connectivity (will fail if no server, but shows connection attempt).
```
ssh -v -o ConnectTimeout=3 -o BatchMode=yes localhost exit 2>&1 | head -20
```

4. View your SSH client configuration.
```
cat ~/.ssh/config 2>/dev/null || echo "No SSH client config found"
```

## J. Practical Protocol Analysis

1. Capture and analyze an HTTP session.
```
# Start capture (captures full packet content)
sudo tcpdump -i any -c 50 -A 'tcp port 80 and host example.com' > /tmp/http_capture.txt 2>&1 &
```
```
# Make HTTP request
curl http://example.com
```
```
# Wait for capture to complete and view
sleep 2
cat /tmp/http_capture.txt
```

2. Identify the HTTP request method and response code.
```
grep -E '(GET|POST|HTTP/1)' /tmp/http_capture.txt | head -5
```

3. Capture ICMP traffic during a ping.
```
# Start capture
sudo tcpdump -i any -c 10 icmp > /tmp/icmp_capture.txt 2>&1 &
```
```
# Generate ICMP traffic
ping -c 3 8.8.8.8
```
```
# View capture
sleep 2
cat /tmp/icmp_capture.txt
```

> Notice the ICMP types: echo request (type 8) and echo reply (type 0).

## K. Connection Tracking

1. View established connections to remote hosts.
```
ss -tn state established
```

2. Count connections by state.
```
ss -tan | tail -n +2 | awk '{print $1}' | sort | uniq -c
```

3. View connections to a specific port.
```
ss -tn dst :443
```

4. Monitor connections in real-time.
```
# Watch for 10 seconds
watch -n 1 'ss -tn | head -20'
```

> Press Ctrl+C to stop watching.

## L. Clean Up

```
# Remove temporary capture files
rm -f /tmp/http_capture.txt /tmp/icmp_capture.txt
```

## M. Key Takeaways

- TCP/IP is a 4-layer practical model used by the Internet
- TCP provides reliability through the three-way handshake and acknowledgments
- UDP is faster but provides no delivery guarantees
- Well-known ports (0-1023) are reserved for standard services
- Tools like `ss`, `tcpdump`, and `nc` help analyze network traffic
- Understanding protocols helps troubleshoot connectivity issues
