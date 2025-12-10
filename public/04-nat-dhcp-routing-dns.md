# Lab 04: NAT, DHCP, Routing, and DNS

> This lab explores Network Address Translation (NAT), DHCP, IP routing, and the Domain Name System (DNS). All exercises are designed for Ubuntu 24.04.

## A. Network Address Translation (NAT) Overview

> **NAT (Network Address Translation)** allows multiple devices with private IP addresses to share a single public IP address for Internet access.

### Why NAT?
- **IPv4 Conservation:** Not enough public IPv4 addresses for all devices
- **Security:** Hides internal network structure from the Internet
- **Flexibility:** Internal addressing can be changed without affecting external connectivity

### NAT Types

| Type | Description | Use Case |
|------|-------------|----------|
| SNAT (Source NAT) | Changes source IP (outbound) | LAN to Internet |
| DNAT (Destination NAT) | Changes destination IP (inbound) | Port forwarding |
| PAT/NAPT | Many-to-one using ports | Home routers |
| 1:1 NAT | One-to-one IP mapping | DMZ servers |

## B. Examining NAT on Your System

1. View your private IP address.
```
ip -4 addr show | grep inet | grep -v 127.0.0.1
```

2. Discover your public IP address (as seen by the Internet).
```
curl -s ifconfig.me && echo ""
```

> If your private and public IPs differ, NAT is in use between you and the Internet.

3. View NAT/connection tracking on Linux (if available).
```
# Check if conntrack is installed
which conntrack && sudo conntrack -L 2>/dev/null | head -10 || echo "conntrack not installed (normal for basic systems)"
```

4. View iptables NAT rules (requires root).
```
sudo iptables -t nat -L -n -v 2>/dev/null | head -20 || echo "No NAT rules configured"
```

## C. Understanding IP Routing

> **Routing** determines how packets travel from source to destination across multiple networks.

### Routing Concepts

| Term | Description |
|------|-------------|
| Route | Path to a destination network |
| Gateway | Next-hop router for reaching other networks |
| Default Route | Where to send packets with no specific route |
| Metric | Route preference (lower = preferred) |

1. View your routing table.
```
ip route show
```

> The `default` route shows your gateway for Internet access.

2. View routing table in traditional format.
```
route -n
```

3. Display the default gateway.
```
ip route | grep default
```

4. View detailed route information.
```
ip route show table all | head -30
```

5. Check which route would be used for a specific destination.
```
ip route get 8.8.8.8
```

6. Check route to a private network address.
```
ip route get 10.0.0.1
```

## D. Tracing Packet Paths

> `traceroute` shows each router (hop) between you and a destination.

1. Trace the path to a public server.
```
traceroute -n google.com
```

> `-n` prevents DNS lookups for faster results. Each line shows:
> - Hop number
> - Router IP
> - Three round-trip times (RTT)

2. Trace with ICMP instead of UDP (may work better through firewalls).
```
sudo traceroute -I google.com
```

3. Use `mtr` for a combined ping/traceroute (if available).
```
which mtr && sudo mtr -r -c 5 google.com || echo "mtr not installed - install with: sudo apt install mtr"
```

4. Compare paths to different destinations.
```
echo "=== Path to Google DNS ==="
traceroute -n -m 15 8.8.8.8
echo ""
echo "=== Path to Cloudflare DNS ==="
traceroute -n -m 15 1.1.1.1
```

## E. DHCP (Dynamic Host Configuration Protocol)

> **DHCP** automatically assigns IP addresses and network configuration to devices.

### DHCP Process (DORA)

1. **Discover** - Client broadcasts "I need an IP"
2. **Offer** - Server offers an IP address
3. **Request** - Client requests the offered IP
4. **Acknowledge** - Server confirms the assignment

### DHCP Provides

- IP address and subnet mask
- Default gateway
- DNS server addresses
- Lease duration

1. Check if your interface uses DHCP.
```
cat /etc/netplan/*.yaml 2>/dev/null || cat /etc/network/interfaces 2>/dev/null | head -20
```

2. View current DHCP lease information (if using NetworkManager).
```
ls /var/lib/NetworkManager/ 2>/dev/null
```

3. View DHCP lease details (if using dhclient).
```
cat /var/lib/dhcp/dhclient.*.leases 2>/dev/null | tail -30 || echo "No dhclient leases found"
```

4. View network configuration from systemd-networkd.
```
networkctl status 2>/dev/null | head -20 || echo "systemd-networkd not managing interfaces"
```

5. Manually request DHCP (be careful - may change your IP!).
```
# View current DHCP client ID
echo "Current network configuration:"
ip addr show $(ip route | grep default | awk '{print $5}') | head -5
```

> **Note:** Running `sudo dhclient -v` would request a new DHCP lease. This is disruptive in a lab environment.

## F. Domain Name System (DNS) Fundamentals

> **DNS** translates human-readable domain names (like google.com) into IP addresses (like 142.250.80.46).

### DNS Hierarchy

```
Root (.)
  └── TLD (.com, .org, .net)
        └── Domain (example.com)
              └── Subdomain (www.example.com)
```

### DNS Record Types

| Type | Purpose | Example |
|------|---------|---------|
| A | IPv4 address | example.com → 93.184.216.34 |
| AAAA | IPv6 address | example.com → 2606:2800:220:1:... |
| CNAME | Alias | www → example.com |
| MX | Mail server | mail.example.com (priority 10) |
| NS | Name server | ns1.example.com |
| TXT | Text data | SPF, DKIM records |
| PTR | Reverse lookup | IP → hostname |
| SOA | Zone authority | Primary NS, admin email |

## G. DNS Configuration

1. View your DNS resolver configuration.
```
cat /etc/resolv.conf
```

2. Check systemd-resolved configuration (Ubuntu 24.04 default).
```
resolvectl status 2>/dev/null | head -30
```

3. View the actual DNS servers being used.
```
resolvectl dns 2>/dev/null || cat /etc/resolv.conf | grep nameserver
```

4. Check DNS cache statistics.
```
resolvectl statistics 2>/dev/null
```

## H. DNS Lookup Tools

### Using `dig`

1. Basic DNS lookup.
```
dig example.com
```

> The ANSWER SECTION shows the resolved IP address.

2. Query for a specific record type.
```
# A record (IPv4)
dig example.com A

# AAAA record (IPv6)
dig example.com AAAA

# MX record (mail servers)
dig example.com MX

# NS record (name servers)
dig example.com NS

# TXT record
dig example.com TXT
```

3. Get a short answer only.
```
dig +short example.com
```

4. Query a specific DNS server.
```
# Query Google DNS
dig @8.8.8.8 example.com

# Query Cloudflare DNS
dig @1.1.1.1 example.com
```

5. Trace the full DNS resolution path.
```
dig +trace example.com
```

> This shows how the query flows from root servers down to the authoritative server.

6. Perform a reverse DNS lookup (IP to hostname).
```
dig -x 8.8.8.8
```

### Using `host`

7. Simple lookup with `host`.
```
host example.com
```

8. Query specific record types.
```
host -t MX example.com
host -t NS example.com
```

### Using `nslookup`

9. Interactive and non-interactive lookups.
```
nslookup example.com
nslookup -type=MX example.com
```

## I. DNS Resolution Process

1. Check local hostname resolution.
```
cat /etc/hosts
```

2. Check the Name Service Switch configuration.
```
grep hosts /etc/nsswitch.conf
```

> This shows the order of lookup: typically files (hosts file) first, then DNS.

3. Test resolution through different methods.
```
echo "=== /etc/hosts lookup ==="
getent hosts localhost

echo ""
echo "=== DNS lookup ==="
getent hosts example.com
```

4. Compare resolution times between cached and uncached queries.
```
echo "First query (may hit cache):"
time dig +short example.com > /dev/null

echo ""
echo "Query to specific server (bypasses local cache):"
time dig @8.8.8.8 +short example.com > /dev/null
```

## J. DNS Troubleshooting

1. Check if DNS is working.
```
dig +short google.com && echo "DNS working" || echo "DNS not working"
```

2. Test DNS server availability.
```
# Test Google DNS
nc -zv -w 3 8.8.8.8 53 2>&1 | grep -E '(succeeded|failed)'

# Test Cloudflare DNS
nc -zv -w 3 1.1.1.1 53 2>&1 | grep -E '(succeeded|failed)'
```

3. Check DNS response time.
```
dig example.com | grep "Query time"
```

4. Verify authoritative name servers.
```
dig NS example.com +short
```

5. Query the authoritative server directly.
```
# Get NS record
NS=$(dig NS example.com +short | head -1)
echo "Authoritative NS: $NS"

# Query it directly
dig @$NS example.com
```

## K. Practical DNS Scenarios

1. Find all IP addresses for a load-balanced site.
```
dig +short www.google.com
```

> Multiple IPs indicate load balancing or CDN.

2. Check mail server configuration.
```
echo "=== MX Records ==="
dig MX gmail.com +short
```

3. Verify SPF record (email authentication).
```
dig TXT gmail.com +short | grep spf
```

4. Check DNSSEC status.
```
dig example.com +dnssec | grep -E '(RRSIG|flags)'
```

5. Find the SOA (Start of Authority) record.
```
dig SOA example.com
```

> SOA contains: primary NS, admin email, serial number, refresh/retry/expire timers.

## L. Combining NAT, Routing, and DNS Knowledge

1. Complete network path analysis.
```
TARGET="google.com"

echo "=== Step 1: DNS Resolution ==="
IP=$(dig +short $TARGET | head -1)
echo "$TARGET resolves to: $IP"

echo ""
echo "=== Step 2: Routing Decision ==="
ip route get $IP

echo ""
echo "=== Step 3: Path to Destination ==="
traceroute -n -m 10 $IP

echo ""
echo "=== Step 4: Verify Connectivity ==="
ping -c 3 $IP
```

2. Troubleshoot connectivity step by step.
```
echo "=== Network Troubleshooting Checklist ==="
echo ""
echo "1. Interface status:"
ip link show | grep -E '^[0-9]|state'
echo ""
echo "2. IP address assigned:"
ip -4 addr show | grep inet | grep -v 127.0.0.1
echo ""
echo "3. Default gateway:"
ip route | grep default
echo ""
echo "4. Gateway reachable:"
ping -c 2 $(ip route | grep default | awk '{print $3}') 2>/dev/null && echo "Gateway OK" || echo "Gateway unreachable"
echo ""
echo "5. Internet reachable (by IP):"
ping -c 2 8.8.8.8 2>/dev/null && echo "Internet OK" || echo "Internet unreachable"
echo ""
echo "6. DNS working:"
dig +short google.com 2>/dev/null && echo "DNS OK" || echo "DNS not working"
```

## M. Clean Up

No cleanup needed - all commands were read-only observations.

## N. Key Takeaways

- **NAT** allows private IPs to access the Internet through shared public IPs
- **Routing** determines packet paths; the default gateway handles unknown destinations
- **DHCP** automatically configures IP addresses and network settings
- **DNS** translates domain names to IP addresses using a hierarchical system
- Troubleshooting order: Link → IP → Gateway → Internet → DNS
- Tools: `ip route`, `traceroute` for routing; `dig`, `host`, `nslookup` for DNS
