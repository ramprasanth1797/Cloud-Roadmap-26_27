# Mission 005 - Day 02

## Topic: VLAN and Security Zones

### VLAN

A VLAN (Virtual Local Area Network) is used to logically separate network traffic into different broadcast domains.

Benefits:
- Separates user traffic
- Improves security
- Reduces broadcast traffic
- Easier network management

Example:

HR VLAN

Finance VLAN

Guest VLAN

IT VLAN

---

## Security Zone

A Security Zone is a logical group of interfaces used by the Palo Alto Firewall to control traffic.

Unlike VLANs, Security Zones decide whether traffic should be allowed or denied.

Examples:

Trust

Untrust

Guest

Server

DMZ

---

## VLAN vs Security Zone

VLAN

- Separates network traffic.
- Works at Layer 2.
- Created on Switches.

Security Zone

- Controls traffic between networks.
- Used by the Firewall.
- Used in Security Policies.

---

## Intrazone Traffic

Traffic moving within the same Security Zone.

Example:

Trust → Trust

By default:

Allow

Firewall still inspects:

- App-ID
- User-ID
- Threat Prevention
- Security Profiles

---

## Interzone Traffic

Traffic moving between different Security Zones.

Example:

Guest → Trust

Trust → Untrust

Server → Untrust

By default:

Deny

Traffic is allowed only if a Security Policy exists.

---

## Why Enterprises Use Zones Instead of IP Addresses

IP addresses can change because of:

- DHCP
- VPN
- Remote Users
- Device replacement

Security Zones remain the same.

This makes policy management easier.

---

## Can One Security Zone Have Multiple Interfaces?

Yes.

One Security Zone can contain multiple Layer 3 interfaces.

Example:

Trust Zone

Ethernet1/1

Ethernet1/2

Ethernet1/3

All belong to the Trust Zone.

---

## Same Zone, Different Interfaces

Even if traffic moves between different interfaces inside the same Security Zone, Palo Alto still inspects the traffic.

The firewall checks:

- App-ID
- User-ID
- Security Policies
- Threat Prevention

---

## Enterprise Example

Guest Zone

↓

Trust Zone

↓

Denied

Guest users cannot access internal resources.

Trust Zone

↓

Untrust

↓

Allowed

Employees can access the Internet.

---

## Key Interview Points

- VLAN separates traffic.
- Security Zone controls traffic.
- Intrazone = Same Zone.
- Interzone = Different Zones.
- Interzone traffic is denied by default.
- One Zone can contain multiple interfaces.
- Palo Alto inspects traffic even within the same Zone.

---

## Day 02 Summary

Today I learned:

- VLAN
- Security Zones
- Difference between VLAN and Security Zone
- Intrazone Traffic
- Interzone Traffic
- Zone-based Security Policies
- Enterprise Firewall Design
