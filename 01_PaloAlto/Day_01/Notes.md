# Mission 001 - Day 1

This file contains my Day 1 Palo Alto notes .

---

# What is a Firewall?

A firewall is a network security device that monitors, filters, allows, or blocks incoming and outgoing traffic based on predefined security rules.

---

# Why is a Firewall Important?

A firewall protects an organization's network from unauthorized access, cyber attacks, malware, and unwanted traffic. It acts as the first line of defense between trusted and untrusted networks.

---

# Traditional Firewall vs Palo Alto Next-Generation Firewall (NGFW)

| Traditional Firewall          | Palo Alto NGFW                 |
| ----------------------------- | -------------------------------|
| Checks Source IP              | Checks Source IP               |
| Checks Destination IP         | Checks Destination IP          |
| Checks Port Number            | Checks Port Number             |
| Checks Protocol               | Checks Protocol                |
| Basic Packet Filtering        | Application Awareness (App-ID) |
| No User Awareness             | User-IDn                       |
| Limited Threat Protection     | Content-ID                     |
| Cannot Detect Unknown Malware | WildFire                       |
| Limited HTTPS Inspection      | SSL Decryption                 |
| Basic Logging                 | Advanced Logging & Visibilitym |

---

# Why do Organizations Choose Palo Alto?

Organizations choose Palo Alto because it provides advanced security beyond traditional firewalls.

## 1. App-ID
- Identifies applications regardless of the port being used.
- Example:
  - Allow Microsoft Teams
  - Block Dropbox
  - Allow Office365

---

## 2. User-ID
- Creates security policies based on users instead of IP addresses.
- Example:
  - Allow HR Department
  - Allow IT Team
  - Block Guest Users

---

## 3. Content-ID
- Scans traffic for:
  - Malware
  - Viruses
  - Spyware
  - Exploits

---

## 4. WildFire
- Detects unknown (zero-day) malware using cloud sandbox analysis.
- Protects organizations from new attacks before antivirus signatures exist.

---

## 5. SSL Decryption
- Decrypts encrypted HTTPS traffic.
- Inspects the traffic.
- Re-encrypts it before forwarding.
- Helps detect malware hidden inside HTTPS.

---

## 6. Zone-Based Security
Example zones:
- Trust
- Untrust
- DMZ
- VPN
- Guest

Policies are created between zones instead of individual IP addresses.

---

## 7. Threat Prevention
Blocks:
- SQL Injection
- Buffer Overflow
- Remote Code Execution
- Known Vulnerabilities
- Command & Control Traffic

---

# Why Palo Alto is Better than a Traditional Firewall

- Application Awareness
- User-Based Policies
- Malware Protection
- Zero-Day Protection
- SSL Inspection
- Advanced Threat Prevention
- Better Visibility
- Detailed Logging
- Zone-Based Security

---

# My Key Takeaways

- A traditional firewall mainly checks IP, Port, and Protocol.
- Palo Alto identifies the application, user, and content inside the traffic.
- App-ID is one of the biggest advantages of Palo Alto.
- WildFire protects against unknown malware.
- SSL Decryption allows inspection of encrypted HTTPS traffic.
- Palo Alto provides much better visibility and security than a traditional firewall.

---

# Revision Schedule (1-4-7 Method)

- ✅ Review Tomorrow (Day 2)
- ✅ Review after 4 Days
- ✅ Review after 7 Days



# Palo Alto Architecture

## Management Plane

## Control Plane

## Data Plane

## How all three work together


# Mission 004 – Packet Flow

## Step 1 – Session Creation

## Step 2 – Zone Identification

## Step 3 – User-ID

## Step 4 – App-ID

## Step 5 – Security Policy

## Step 6 – Threat Prevention

## Step 7 – NAT

## Step 8 – Routing

## Step 9 – Forward Packet

## Production Example

## Key Takeaways

## Production Example

## Interview Questions

## My Understanding
