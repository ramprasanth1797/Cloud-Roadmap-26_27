# Day 06 - Routing Fundamentals, Virtual Router & Route Types

## 🎯 Learning Objectives

By the end of Day 06, I learned:

- Routing Fundamentals
- Routing Table
- Next Hop
- Virtual Router
- Connected Route
- Static Route
- Default Route
- Longest Prefix Match
- Basic Routing Troubleshooting

---

# 1. Packet Processing Order

Whenever a packet arrives at the Palo Alto firewall, it follows this sequence:

```text
Packet Arrives
      │
      ▼
Security Policy
      │
      ▼
NAT
      │
      ▼
Routing
      │
      ▼
Forward Packet
```

Each component has a different responsibility.

| Component | Responsibility |
|----------|----------------|
| Security Policy | Allow or Deny Traffic |
| NAT | Translate IP Address |
| Routing | Decide where to send the packet |

---

# 2. What is Routing?

Routing is the process of deciding **where a packet should be forwarded next**.

Routing does **not** decide whether traffic is allowed.

It only answers one question:

> **"Where should I send this packet?"**

Example:

```text
Source IP      : 192.168.10.25
Destination IP : 8.8.8.8
```

Packet Flow

```text
Security Policy
      ↓
Allow
      ↓
NAT
      ↓
Translate Source IP
      ↓
Routing
      ↓
Find Next Hop
      ↓
Forward Packet
```

### Remember

> **Routing decides the next hop towards the destination.**

---

# 3. Routing Table

A Routing Table is a database inside the firewall that stores all routes.

Example

| Destination Network | Next Hop |
|---------------------|----------|
| 192.168.10.0/24 | Connected |
| 10.200.0.0/16 | 192.168.50.2 |
| 0.0.0.0/0 | 49.204.100.1 |

Whenever a packet arrives, the firewall checks this table to determine the best route.

---

# 4. Next Hop

The firewall usually **does not send packets directly to the final destination**.

Instead, it forwards the packet to the **Next Hop**, and the next router continues forwarding it.

Example

```text
Destination
8.8.8.8

↓

Routing Table

↓

0.0.0.0/0

↓

Next Hop
49.204.100.1

↓

ISP

↓

Internet
```

### Remember

> **Routing forwards the packet to the Next Hop, not directly to the destination.**

---

# 5. What is a Virtual Router?

A **Virtual Router (VR)** is the routing engine inside the Palo Alto firewall.

Its job is to maintain the routing table and make routing decisions.

A Virtual Router stores:

- Connected Routes
- Static Routes
- Dynamic Routes
- Default Route
- OSPF
- BGP
- RIP (if configured)

Diagram

```text
Palo Alto Firewall
│
├── Security Policy
├── NAT
└── Virtual Router
      │
      ├── Routing Table
      ├── Connected Routes
      ├── Static Routes
      ├── Dynamic Routes
      └── Default Route
```

### Remember

> **Virtual Router = Routing Engine inside the firewall.**

---

# 6. Virtual Router vs Default Route

Many beginners confuse these.

| Virtual Router | Default Route |
|---------------|---------------|
| Routing Engine | One route inside the routing table |
| Stores all routing information | Used when no specific route exists |

Remember:

```text
Virtual Router
↓

Entire Routing Table

------------------------

Default Route
↓

Only one route inside that table
```

---

# 7. Connected Route

A Connected Route is created **automatically** when an IP address is assigned to a firewall interface.

Example

```text
Interface

ethernet1/1

IP Address

192.168.50.1/24
```

Automatically creates

```text
192.168.50.0/24

Connected Route
```

No manual configuration is required.

### Remember

> **Assign Interface IP → Connected Route is created automatically.**

---

# 8. Connected Route Example

```text
Interface

172.16.20.1/24
```

Automatically creates

```text
172.16.20.0/24

Connected Route
```

Notice

```text
172.16.20.1
↓

Interface IP

----------------------

172.16.20.0/24
↓

Connected Route
```

---

# 9. Static Route

A Static Route is manually configured by the administrator.

It is used to reach a **remote network**.

Example

```text
Destination Network

10.200.0.0/16

Next Hop

192.168.50.2
```

Meaning

> **To reach 10.200.0.0/16, first send the packet to 192.168.50.2.**

---

# 10. Connected Route vs Static Route

| Connected Route | Static Route |
|----------------|--------------|
| Automatic | Manual |
| Directly Connected Network | Remote Network |
| No Next Hop Required | Next Hop Required |

Remember

```text
Connected Route

↓

Automatic

↓

Directly Connected

------------------------

Static Route

↓

Manual

↓

Remote Network

↓

Next Hop Required
```

---

# 11. Default Route (0.0.0.0/0)

A Default Route is used when **no more specific route exists**.

Example

```text
Destination Network

0.0.0.0/0

Next Hop

49.204.100.1
```

Meaning

> **If no specific route matches, send the packet to the ISP.**

Example

```text
Destination

8.8.8.8

↓

Connected Route?

No

↓

Static Route?

No

↓

Default Route

↓

49.204.100.1
```

### Remember

> **No specific route → Use Default Route.**

---

# 12. What Happens if No Default Route Exists?

Scenario

```text
Connected Route

Not Found

↓

Static Route

Not Found

↓

Default Route

Not Configured
```

Result

```text
Firewall

↓

No Route

↓

Packet Dropped
```

**Important**

This is **not** a Security Policy deny.

This is a **Routing failure**.

---

# 13. Longest Prefix Match

When multiple routes match a destination, the firewall chooses the **most specific route**.

Example

Routing Table

```text
10.0.0.0/8

10.10.0.0/16

10.10.20.0/24

0.0.0.0/0
```

Destination

```text
10.10.20.100
```

Matches

```text
10.0.0.0/8

✔

10.10.0.0/16

✔

10.10.20.0/24

✔
```

Selected Route

```text
10.10.20.0/24
```

Because **/24** is more specific than **/16** and **/8**.

### Remember

> **If multiple routes match, the route with the longest prefix wins.**

---

# 14. Prefix Priority

```text
/32

↓

/24

↓

/16

↓

/8

↓

/0
```

The larger the prefix length, the more specific the route.

---

# 15. Routing Troubleshooting Checklist

When troubleshooting routing problems:

```text
1. Is Security Policy allowing the traffic?

↓

2. Is NAT working correctly?

↓

3. Is there a Connected Route?

↓

4. Is there a Static Route?

↓

5. Is there a Default Route?

↓

6. Which routes match?

↓

7. Which route has the Longest Prefix?

↓

8. Is the Next Hop reachable?
```

---

# 16. Production Example

User

```text
192.168.10.25
```

Destination

```text
8.8.8.8
```

Packet Flow

```text
Security Policy

↓

Allow

↓

Source NAT

↓

49.204.100.10

↓

Routing Table

↓

No Connected Route

↓

No Static Route

↓

Default Route

0.0.0.0/0

↓

Next Hop

49.204.100.1

↓

ISP

↓

Internet
```

---

# Interview Questions

## What is the difference between Connected Route and Static Route?

**Connected Route**

- Automatically created
- Directly connected network
- No Next Hop required

**Static Route**

- Manually configured
- Used for remote networks
- Requires a Next Hop

---

## What is the purpose of the Default Route?

The Default Route forwards traffic to the configured Next Hop when no more specific route matches the destination.

---

## What is Longest Prefix Match?

When multiple routes match a destination, the firewall chooses the route with the **largest prefix length (most specific route).**

---

# Day 06 Revision Sheet

```text
Routing
↓
Where should I send the packet?

--------------------------------

Virtual Router
↓
Routing Engine

--------------------------------

Connected Route
↓
Automatic
↓
Directly Connected Network

--------------------------------

Static Route
↓
Manual
↓
Remote Network
↓
Next Hop Required

--------------------------------

Default Route
↓
0.0.0.0/0
↓
Used when no specific route exists

--------------------------------

Longest Prefix Match
↓
Most Specific Route Wins

--------------------------------

Packet Flow

Security Policy
↓

NAT
↓

Routing
↓

Forward to Next Hop
```

---

# Key Takeaways

- Routing decides **where to send the packet next**.
- A Virtual Router is the routing engine inside the Palo Alto firewall.
- Connected Routes are created automatically for directly connected networks.
- Static Routes are manually configured and require a Next Hop.
- A Default Route (`0.0.0.0/0`) is used when no specific route matches.
- If multiple routes match, the firewall selects the **Longest Prefix Match**.
- If no valid route exists, the firewall **drops the packet** because of a routing failure, not because of a Security Policy deny.
