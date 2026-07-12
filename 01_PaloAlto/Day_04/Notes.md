# Day 04 - NAT Fundamentals

## Topics Covered

- What is NAT?
- Why do we need NAT?
- Source NAT (SNAT)
- PAT / DIPP
- Destination NAT (DNAT)
- NAT vs Security Policy
- Basic NAT troubleshooting

---

## 1. What is NAT?

NAT stands for Network Address Translation.

NAT is a process where the firewall translates an IP address while traffic passes through it.

The main use of NAT is to allow devices with private IP addresses to communicate with external networks such as the Internet.

Example:

Before Source NAT:

Source IP: 192.168.1.10  
Destination IP: 8.8.8.8

After Source NAT:

Source IP: 49.204.100.25  
Destination IP: 8.8.8.8

The firewall translated the private source IP into a public IP.

---

## 2. Why Do We Need NAT?

There are three main reasons:

### Reason 1: Private IPs are not routable on the public Internet

Examples of private IP ranges include:

- 10.0.0.0/8
- 172.16.0.0/12
- 192.168.0.0/16

An internal device may have:

192.168.1.10

But this private IP cannot be routed directly across the public Internet.

Therefore, the firewall translates it into a public IP.

---

### Reason 2: Public IPv4 addresses are limited

Without NAT, every laptop, mobile phone, server, and other device would potentially require its own public IPv4 address.

NAT allows many internal devices to share one or more public IP addresses.

---

### Reason 3: NAT hides internal addressing

An external server does not need to see the original internal private IP.

For example:

Internal IP:

192.168.1.10

Translated Public IP:

49.204.100.25

The Internet sees the translated public IP instead of the original internal private IP.

Important:

NAT itself should not be considered a replacement for a firewall Security Policy.

---

## 3. Source NAT (SNAT)

Source NAT is generally used when an internal user accesses an external network such as the Internet.

Traffic flow:

Internal User → Firewall → Internet

Example:

Before NAT:

Source IP: 192.168.1.10  
Destination IP: 8.8.8.8

After NAT:

Source IP: 49.204.100.25  
Destination IP: 8.8.8.8

The Source IP is translated.

### Memory Rule

Source NAT → Changes the Source IP.

---

## 4. Does the Laptop Get a Public IP?

No.

The laptop keeps its original private IP:

192.168.1.10

The firewall performs the translation while processing the traffic.

Example:

Laptop:

192.168.1.10

Firewall translates:

192.168.1.10 → 49.204.100.25

The Internet sees:

49.204.100.25

When the return traffic arrives, the firewall uses its session/NAT information to send the traffic back to the correct internal client.

The laptop itself does not permanently become:

49.204.100.25

### Important Interview Point

After Source NAT, the PC still keeps its private IP address. The firewall performs the translation.

---

## 5. PAT - Port Address Translation

PAT stands for Port Address Translation.

In Palo Alto, this is commonly associated with:

Dynamic IP and Port (DIPP)

PAT allows multiple internal users to share the same public IP by using different source ports.

Example:

Ram:

192.168.1.10:55000

Translated to:

49.204.100.25:10001

John:

192.168.1.11:55001

Translated to:

49.204.100.25:10002

Sarah:

192.168.1.12:55002

Translated to:

49.204.100.25:10003

All three users share the same public IP:

49.204.100.25

But each translated session can be distinguished by its port information.

---

## 6. What Changes During PAT?

PAT can translate:

1. Source IP
2. Source Port

Example:

Before PAT:

192.168.1.10:55000

After PAT:

49.204.100.25:10001

### Memory Rule

PAT / DIPP → Source IP + Source Port.

---

## 7. PAT Translation Example

| User | Private IP | Private Port | Public IP | Translated Port |
|---|---|---|---|---|
| Ram | 192.168.1.10 | 55000 | 49.204.100.25 | 10001 |
| John | 192.168.1.11 | 55001 | 49.204.100.25 | 10002 |
| Sarah | 192.168.1.12 | 55002 | 49.204.100.25 | 10003 |

When return traffic reaches the firewall, the firewall uses its session and NAT information to identify the correct internal client.

---

## 8. Why is PAT Important?

Without PAT, organizations could require many public IPv4 addresses.

PAT allows multiple users to share a public IP by distinguishing their sessions.

Example:

5000 employees

↓

One or more shared public IP addresses

↓

Different sessions and port mappings

This helps conserve public IPv4 addresses.

---

## 9. Destination NAT (DNAT)

Destination NAT is generally used when an external user needs to access an internal server through a public IP.

Traffic flow:

Internet User → Firewall → Internal Server

Example:

Public IP:

203.0.113.10

Actual internal web server:

192.168.10.100

The firewall translates:

203.0.113.10 → 192.168.10.100

The Destination IP is translated.

### Memory Rule

Destination NAT → Changes the Destination IP.

---

## 10. Source NAT vs Destination NAT

| Feature | Source NAT | Destination NAT |
|---|---|---|
| Typical direction | Internal → External | External → Internal |
| Main address translated | Source IP | Destination IP |
| Common use case | Employees accessing Internet | External users accessing internal server |

### Easy Memory Trick

Source NAT:

Inside → Outside

Destination NAT:

Outside → Inside

---

## 11. NAT Does Not Automatically Allow Traffic

This is one of the most important concepts.

NAT and Security Policy perform different jobs.

### NAT

Translates addresses.

### Security Policy

Allows or denies traffic.

### Golden Rule

NAT translates traffic. Security Policy allows or denies traffic.

Example:

Destination NAT exists:

203.0.113.10 → 192.168.10.100

But there is no matching Security Policy allow rule.

Result:

Traffic can still be denied.

A NAT rule alone does not automatically permit the traffic.

---

## 12. Missing Source NAT

Scenario:

Internal user:

192.168.10.25

Security Policy: Allow  
Routing: Correct  
Source NAT: Missing

Without Source NAT, the packet retains its private source IP.

The public Internet does not route RFC1918 private addresses as normal public Internet destinations.

Therefore, successful end-to-end Internet communication will fail.

### Memory Rule

No Source NAT → Private source IP remains untranslated → Internet communication fails.

---

## 13. Missing Destination NAT

Scenario:

Public IP:

49.204.100.50

Internal server:

192.168.50.10

Required translation:

49.204.100.50 → 192.168.50.10

If the Destination NAT rule is missing, the firewall does not perform the required translation from the public destination IP to the internal server IP.

Therefore, traffic does not reach the intended internal server through that NAT mapping.

---

## 14. Basic Troubleshooting Flow

When an internal user cannot access the Internet, check:

1. Source IP
2. Source and Destination Zones
3. Security Policy
4. NAT Rule
5. Translated IP
6. Routing
7. DNS, if IP connectivity works but domain names fail

Example:

If the user can ping:

8.8.8.8

But cannot resolve:

google.com

Check DNS.

---

## 15. Important Interview Questions

### Q1: What does Source NAT change?

The Source IP address.

### Q2: What does Destination NAT change?

The Destination IP address.

### Q3: What does PAT/DIPP commonly translate?

The Source IP and Source Port.

### Q4: Does the laptop itself become a public IP after Source NAT?

No. The laptop keeps its private IP. The firewall performs the translation.

### Q5: Can NAT alone allow traffic?

No. NAT translates addresses, while Security Policy allows or denies traffic.

### Q6: Why do we need NAT?

- Private IPs are not routable on the public Internet.
- Public IPv4 addresses are limited.
- Multiple internal devices can share public IP resources.

---

## 16. My Understanding

Today I learned that NAT translates IP addresses while traffic passes through the firewall.

Source NAT is generally used when internal users access external networks. It changes the source IP from a private IP to a public IP.

Destination NAT is generally used when external users access an internal server through a public IP. It changes the destination IP from the public IP to the internal server's private IP.

PAT, also known in Palo Alto as Dynamic IP and Port (DIPP), allows multiple internal users to share a public IP by using port translations to keep sessions separate.

Most importantly:

NAT translates traffic. Security Policy allows or denies traffic.
