# Mission 009 - Day 03

## Topic: Security Policies

### What is a Security Policy?

A Security Policy is a set of rules that tells the Palo Alto Firewall whether to Allow or Deny traffic based on different criteria such as:

- Source Zone
- Destination Zone
- Application (App-ID)
- User (User-ID)
- Service
- Action (Allow/Deny)

---

# Rule Matching

The firewall checks every rule one by one from the top.

A rule is matched only if ALL the conditions match.

Example:

Source Zone = Trust ✅

Destination Zone = Untrust ✅

Application = Teams ✅

User = Ram ✅

Service = Application-Default ✅

Action = Allow

If even one condition doesn't match, the firewall checks the next rule.

---

# First Match Wins

Palo Alto always processes rules from Top to Bottom.

Once a rule matches:

- Apply the Action
- Stop checking remaining rules

The firewall never searches for the "best" rule.

It only searches for the "first matching" rule.

---

# Top-Down Processing

Firewall processing order:

Rule 1

↓

Rule 2

↓

Rule 3

↓

...

↓

Implicit Deny

---

# Shadow Rule

A Shadow Rule is a rule that will never be matched because a previous rule already matches the traffic.

Example:

Rule 1
Application = Any
Action = Allow

Rule 2
Application = Teams
Action = Deny

Rule 2 is never reached because Rule 1 already matches every application.

---

# Implicit Deny

Every Palo Alto Firewall has a hidden rule at the end of the Security Policy.

ANY

↓

ANY

↓

ANY

↓

DENY

If no configured rule matches the traffic, the firewall automatically denies the traffic.

This is called the Implicit Deny Rule.

---

# Service

The Service field specifies the TCP/UDP port used by the application.

Examples:

SSH → TCP 22

HTTPS → TCP 443

RDP → TCP 3389

---

# Application-Default vs Service = Any

## Application-Default

Allows the application only on its default ports.

Example:

SSH → TCP 22 ✅

SSH → TCP 2222 ❌

More Secure

Recommended in Enterprise

---

## Service = Any

Allows the application on any TCP/UDP port.

Example:

SSH → TCP 22 ✅

SSH → TCP 2222 ✅

Less Secure

Used only when required

---

# Enterprise Security Policy Best Practices

1. Place Specific Rules first.
2. Place General Rules later.
3. Use Application-Default whenever possible.
4. Keep Allow Internet near the bottom.
5. Deny Guest access to Internal Networks.
6. Rely on Implicit Deny for unmatched traffic.

---

# Enterprise Example

Rule 1
Trust → Untrust
Outlook
Allow

Rule 2
Trust → Untrust
Teams
Allow

Rule 3
Trust → Untrust
Office365
Allow

Rule 4
Trust → Server
Payroll
Allow

Rule 5
Trust → Server
SSH
Allow

Rule 6
Trust → Server
RDP
Allow

Rule 7
Guest → Trust
Any
Deny

Rule 8
Guest → Server
Any
Deny

Rule 9
Guest → Untrust
Any
Allow

Rule 10
Trust → Untrust
Any
Allow

↓

Implicit Deny

---

# Production Troubleshooting Checklist

When a user cannot access an application:

1. Check whether a Session is created.
2. Verify App-ID.
3. Verify User-ID.
4. Verify Source and Destination Zones.
5. Check the Security Policy.
6. Verify Service (Application-Default or Any).
7. Check Rule Order (First Match Wins).
8. Verify whether the traffic is hitting the Implicit Deny Rule.

---

# Key Interview Points

- Security Policy controls Allow/Deny decisions.
- Palo Alto uses Top-Down Processing.
- First Matching Rule Wins.
- Every rule must fully match.
- Shadow Rules are never evaluated.
- Every firewall has an Implicit Deny Rule.
- Application-Default is more secure than Service = Any.
- Specific rules should always be placed above General rules.

---

# Day 03 Summary

Today I learned:

- How Security Policies work.
- Rule Matching process.
- First Match Wins.
- Top-Down Rule Processing.
- Shadow Rules.
- Implicit Deny Rule.
- Difference between Application-Default and Service = Any.
- Enterprise Firewall Policy Design.
- Production Troubleshooting methodology.
